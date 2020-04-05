---
title: HashMap正反双映射--LRU时间复杂度O(1)实现
date: 2020-02-27T10:25:17+08:00
coverImage: https://s2.ax1x.com/2020/02/27/3dmzWD.jpg
categories: 
    - 数据结构与算法
tags: 
    - 数据结构与算法
    - 操作系统
    - 缓存技术
    - HashMap
---
<!-- toc -->
该文章受LeetCode上面的[一道题目](https://leetcode-cn.com/problems/lfu-cache/)启发。题目要求在时间复杂度O(1)的情况下实现LRU缓存算法。

<!-- more -->

## 1. 背景

首先说一下LRU：最不经常使用淘汰算法（Least Frequently Used）。LFU是淘汰一段时间内，使用**次数最少**的页面。

例如，容量3，访问顺序依次是 `2,1,2,1,2,3,4`。当请求`4` 时，缓存容量已满，这时需要淘汰一个页面，按照LRU规则，淘汰`3`号页面。

另外题目要求，如果淘汰时有多个次数最少的页面，则淘汰这些次数最少的页面中最久没使用的页面。

题目要求O(1) 时间复杂度内实现get和put方法：
- `get(key)` - 如果键存在于缓存中，则获取键的值（总是正数），否则返回 -1。
- `put(key, value)` - 如果键不存在，请设置或插入值。当缓存达到其容量时，它应该在插入新项目之前，使最不经常使用的项目无效。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，最近最少使用的键将被去除。

## 2. 分析

题目要求时间复杂度O(1)，纵览所有数据结构也只有HashMap有可能实现O(1)时间复杂度存取了。

朴素的想法是HashMap中`key`为页号，`value`为 HashNode（值和访问次数）。key-->hashNode

``` Java
class HashNode{
    Integer value; // 值
    Integer freq;  // 访问次数
}
```
![](/img/数据结构与算法/HashMap%E6%AD%A3%E5%8F%8D%E5%8F%8C%E6%98%A0%E5%B0%84--LRU%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6O(1)%E5%AE%9E%E7%8E%B0/HashNode.png)

实现LFU需要在缓存满的时候删除访问次数(`freq`)最低的页面，而HashMap可以很方便根据key获取我们这里的HashNode(包含freq)，却不能根据freq找到key,以便在hashMap中删除这个访问频率最低的key。

因此我们希望增加一个反向映射(freq-->key)，以实现根据freq来找到key。由于一个freq可能对应多个key，所以这里应该是一个key集合。另外为了满足拿到集合中最久没使用的key,采用`LinkedHashSet`。

``` Java
class HashNode2{
    LinkedHashSet<Integer> keySet;
}
```

![](/img/数据结构与算法/HashMap%E6%AD%A3%E5%8F%8D%E5%8F%8C%E6%98%A0%E5%B0%84--LRU%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6O(1)%E5%AE%9E%E7%8E%B0/HashNode2.png)

有了反向映射(freq-->key)，如何在这个反向映射中找到最低的freq呢？如果遍历freq，时间复杂度就不是O(1)了，所以这里还是不能采用HashMap。因此我们还需要一个数据结构维护freq的顺序，可以采用链表维护，由于每次只要获取最小的freq。按照升序，链表的第一个节点就是最小的freq了。满足时间复杂度要求。

这时我们需要将`HashNode2`更改为`LinkNode`:

``` Java
class LinkNode{
    Integer freq;
    LinkedHashSet<Integer> keySet;
    LinkNode next;
}
```

![](/img/数据结构与算法/HashMap%E6%AD%A3%E5%8F%8D%E5%8F%8C%E6%98%A0%E5%B0%84--LRU%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6O(1)%E5%AE%9E%E7%8E%B0/LinkNode.png)

我们每次get时，需要更新freq，我们不仅要修改`key-->hashNode`的映射，我们还需要将`LinkNode.keySet`中的`key`取出来放到下一个`LinkNode`中去。这就需要能够很快根据freq找到这个`LinkNode`。观察发现，`HashNode`中包含freq，`LinkNode`中也包含freq。故而我们可以将`HashNode`更改为下面这样：

```Java
class HashNode{
    Integer value;      // 值
    LinkNode linkNode;  // 包含访问次数和相同访问次数的其他key
}
```

![](/img/数据结构与算法/HashMap%E6%AD%A3%E5%8F%8D%E5%8F%8C%E6%98%A0%E5%B0%84--LRU%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6O(1)%E5%AE%9E%E7%8E%B0/ALL.png)

至此这个结构应该就可以完成LFU的设计了。

## 3. 代码实现

注意：代码中为了方便描述，链表部分采用了双向链表

```Java
class LFUCache {
    // 定义HashMap的节点
    class HashNode {
        int value;
        LinkNode linkNode;

        public HashNode(int value, LinkNode linkNode) {
            this.value = value;
            this.linkNode = linkNode;
        }
    }
    // 链表节点
    class LinkNode {
        int freq;
        LinkedHashSet<Integer> keySet;     // 记录相同freq的所有key
        LinkNode pre;
        LinkNode next;

        public LinkNode(int freq, Integer key) {
            this.freq = freq;
            keySet = new LinkedHashSet<>();
            keySet.add(key);
        }
    }

    private HashMap<Integer, HashNode> totalMap;    // 总map,根据key找value
    private LinkNode head;                          // 链表维护频率顺序
    private LinkNode tail;
    private int capacity;                           // 缓存容量

    public LFUCache(int capacity) {
        this.capacity = capacity;
        totalMap = new HashMap<>();
        head = new LinkNode(-1, null);
        tail = new LinkNode(-1, null);
        head.next = tail;
        tail.pre = head;
    }

    // 将node插入after之后
    private void insertLinkNode(LinkNode node, LinkNode after) {
        node.next = after.next;
        after.next.pre = node;
        after.next = node;
        node.pre = after;
    }

    // 删除一个节点
    private void removeLinkNode(LinkNode node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
    }

    public int get(int key) {
        if (totalMap.get(key) == null) {
            return -1;
        }
        // get 一次，访问频率需要更新
        LinkNode linkNode = totalMap.get(key).linkNode;
        linkNode.keySet.remove(key);
        int newFreq = linkNode.freq + 1;
        // 如果linkNode.keySet为空，也就没有存在意义
        if (linkNode.keySet.size() == 0) {
            removeLinkNode(linkNode);
            linkNode = linkNode.pre;
        }
        // 处理新freq对应的linkNode
        if (linkNode.next.freq == newFreq) {
            linkNode.next.keySet.add(key);
        } else {
            LinkNode tmp = new LinkNode(newFreq, key);
            insertLinkNode(tmp, linkNode);
        }
        // 更新totalMap
        int retValue = totalMap.get(key).value;
        totalMap.put(key, new HashNode(retValue, linkNode.next));

        return retValue;
    }

    public void put(int key, int value) {
        if(capacity == 0){
            return;
        }
        if (totalMap.get(key) == null) {
            // 插入hashNode之前检测有没有满，满的话需要先删除一个
            if (totalMap.size() == capacity) {
                // 把访问次数最少的（LFU）且最近未使用的（LRU）删除
                LinkNode linkNode = head.next;
                Iterator<Integer> it = linkNode.keySet.iterator();
                totalMap.remove(it.next());
                it.remove();
                if (linkNode.keySet.size() == 0) {
                    removeLinkNode(linkNode);
                }
            }
            // 需要插入一个hashNode，这时检查有没有freq == 1的linkNode
            LinkNode linkNode = head.next;
            if (linkNode.freq != 1) {
                // 没有的话要插入一个linkNode
                LinkNode tmp = new LinkNode(1, key);
                insertLinkNode(tmp, head);
            }
            linkNode = head.next;
            linkNode.keySet.add(key);
            // 插入totalMap
            totalMap.put(key, new HashNode(value, linkNode));
        } else {
            // 如果key已经存在，相当于get一次（增加频率），另外修改value
            get(key);
            // 更新value
            HashNode hashNode = totalMap.get(key);
            hashNode.value = value;
            totalMap.put(key, hashNode);
        }
    }

    public static void main(String[] args) {
        LFUCache cache = new LFUCache(2);
        cache.put(1, 1);
        cache.put(2, 2);
        System.out.println(cache.get(1));       // 返回 1
        cache.put(3, 3);    // 去除 key 2
        System.out.println(cache.get(2));       // 返回 -1 (未找到key 2)
        System.out.println(cache.get(3));       // 返回 3
        cache.put(4, 4);    // 去除 key 1
        System.out.println(cache.get(1));       // 返回 -1 (未找到 key 1)
        System.out.println(cache.get(3));       // 返回 3
        System.out.println(cache.get(4));       // 返回 4
    }
}
```


---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
