---
title: LRU和LFU算法
date: 2020-02-26T15:36:12+08:00
coverImage: https://s2.ax1x.com/2020/02/26/3UQuAx.jpg
categories: 
    - 数据结构和算法
tags: 
    - 数据结构和算法
    - 操作系统
    - 缓存技术
---
<!-- toc -->
LRU和LFU都是操作系统中内存管理的页面置换算法，本质上是一个缓存算法。

**LRU：** 最近最少使用淘汰算法（Least Recently Used）。LRU是淘汰**最长时间没有被使用**的页面。  
**LFU：** 最不经常使用淘汰算法（Least Frequently Used）。LFU是淘汰一段时间内，使用**次数最少**的页面。

<!-- more -->

## 1. 例子详解

假设LFU方法的时期T为10分钟，访问如下页面所花的时间正好为10分钟，内存块大小为3。

若所需页面顺序依次如下：

2  1  2  1  2  3  4

---------------------------------------->

当需要使用页面4时，内存块中存储着1、2、3，内存块中没有页面4，就会发生缺页中断，而且此时内存块已满，需要进行页面置换。

若按LRU算法，应替换掉页面1。因为页面1是最长时间没有被使用的了，页面2和3都在它后面被使用过。

若按LFU算法，应换页面3。因为在这段时间内，页面1被访问了2次，页面2被访问了3次，而页面3只被访问了1次，一段时间内被访问的次数最少。

可见LRU关键是看页面最后一次被使用到发生替换的时间长短，时间越长，页面就会被置换； 而LFU关键是看一定时间段内页面被使用的频率（次数），使用频率越低，页面就会被置换。

也就是说： LRU算法适合：较大的文件比如游戏客户端（最近加载的地图文件） 

LFU算法适合：较小的文件和教零碎的文件比如系统文件、应用程序文件 其中：LRU消耗CPU资源较少，LFU消耗CPU资源较多。

## 2. 算法实现

### 2.1. LRU

维护一个链表，每有一个新的页面被请求时：
- 如果链表中有该页面，则取出该节点放到链表头部
- 否则
    - 如果缓存容量未满，直接将新页面插到链表头部
    - 否则删除最后一个节点再插到头部

当然上述通过链表维护效率有点低，每次请求新页面都要遍历链表。一种优化手段是使用带链表的HashMap,可以将时间复杂度降到O(1)。

带链表的HashMap维护节点被访问的顺序。每次需要删除节点时，直接删除最后的节点即可。

``` Java
class LRUCache {
    private LinkedHashMap<Integer, Integer> cache;
    private int capacity;

    public LRUCache(int capacity) {
        this.cache = new LinkedHashMap<>(capacity, 0.75f, true){
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return this.size() > capacity;
            }
        };
        this.capacity = capacity;
    }

    public int get(int key) {
        return this.cache.getOrDefault(key,-1);
    }

    public void put(int key, int value) {
        this.cache.put(key,value);
    }
}
```

### 2.2. LFU

LFU在意的是访问频率，只需要每次有页面被访问时，对应频率增加，需要删除时，删除访问频率最低的即可。

如果要求在访问频率相同时，删除最近最少使用的那个页面。则可以同LRU一样维护一个链表，但是需要增加记录频率。

维护一个链表，每有一个新的页面被请求时：
- 如果链表中有该页面，则取出该节点放到链表头部
- 否则
    - 如果缓存容量未满，直接将新页面插到链表头部
    - 否则遍历链表，取最小频率的节点删除，然后将新页面插到链表头部。遍历时从头往尾遍历，用小于等于比较，这样会取到最小频率值且靠后的节点。

当然上述做法同样要效率低一些，也可以采用HashMap优化，将时间复杂度降到O(1)。

优化方法参考：[LeetCode](https://leetcode-cn.com/problems/lfu-cache/solution/shuo-de-ming-bai-by-jason-2/)

---

系列：
上一篇：{% post_link  %}
下一篇：{% post_link  %}
