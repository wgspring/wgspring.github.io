---
title: socket实现不同程序间的通信
date: 2020-01-05T07:14:49.096Z
coverImage: https://i.loli.net/2020/02/14/OQq4v6JNdMsDHza.png
categories: 
    - socket
tags: 
    - c++
    - c#
    - socket
---
<!-- toc -->

每一个应用程序是一个独立的进程，拥有自己独立的资源，如果我们期望他们之间发生通信，则可以通过socket通信来实现。socket能够实现网络通信，也就是跨设备的应用间通信。例如计算机A上的QQ和计算机B上的QQ就是采用socket通信的。
<!-- more -->

## 1. 进程
当我们运行一个程序时，操作系统会为该程序启动一个进程来执行。比较直观的感受，可以通过打开Windows的任务管理器，或则Linux系统终端执行`top`查看进程。

![](/img/other/socket实现不同程序间的通信/进程管理.png)

每一个应用程序是一个独立的进程，拥有自己独立的资源，如果我们期望他们之间发生通信，则可以通过socket通信来实现。socket能够实现网络通信，也就是跨设备的应用间通信。例如计算机A上的QQ和计算机B上的QQ就是采用socket通信的。

## 2. TCP通信交互流程

说到网络通信，不得不提到计算机网络中的通信协议。关于TCP和UDP的区别不在本篇讨论范围之内。本文主要讲解通过socket使用TCP协议实现进程通信。下图展示了socket通信过程。

![](/img/other/socket实现不同程序间的通信/socket通信过程.png)

## 3. C++实现socket通信

### 3.1. Server实现

``` cpp Server.cpp
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

using namespace std;

#define MAXLINE 4096

int main()
{
    int server_socket, client_connect;
    char buff[MAXLINE];
    int status;

    // 1. 创建socket
    server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket == -1)
    {
        cout << "create socket failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    // 2. 设置端口号为6666
    struct sockaddr_in address;
    memset(&address, 0, sizeof(address)); // 初始化servaddr所在内存为全0
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = htonl(INADDR_ANY); // 本机预使用的IP任意
    address.sin_port = htons(6666);

    // 3. 绑定端口号到socket上面
    status = bind(server_socket, (struct sockaddr *)&address, sizeof(address));
    if (status == -1)
    {
        cout << "bind socket failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    // 4. 监听端口号
    status = listen(server_socket, 10);
    if (status == -1)
    {
        cout << "listen socket failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    cout << "======waiting for client's request======" << endl;

    // 5. 接受客户端的连接请求
    client_connect = accept(server_socket, (struct sockaddr *)NULL, NULL);
    if (client_connect == -1)
    {
        cout << "accept socket failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    cout << "some one connected!!" << endl;

    // 6. 发送消息
    char sendline[] = "welcome!";
    status = send(client_connect, sendline, strlen(sendline), 0);
    if (status < 0)
    {
        cout << "send msg failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    // 7. 一直接收消息
    while (true)
    {
        int n = recv(client_connect, buff, MAXLINE, 0);
        if (n == 0) //当信息长度为0，说明客户端连接断开
            break;
        buff[n] = '\0'; // 设置文本终止符
        cout << "recv msg from client: " << buff << endl;
    }

    // 8. 关闭连接和服务器
    close(client_connect);
    close(server_socket);
    return 0;
}
```

### 3.2. Client实现

``` cpp Client.cpp
#include <iostream>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define MAXLINE 4096

using namespace std;

int main()
{
    int client_socket;
    char buff[MAXLINE];
    int status;

    // 1. 创建socket
    client_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (client_socket < 0)
    {
        cout << "create socket failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    // 2. 设置端口号为6666
    struct sockaddr_in address;
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_port = htons(6666);

    // 3. 设置访问ip
    status = inet_pton(AF_INET, "127.0.0.1", &address.sin_addr);
    if (status <= 0)
    {
        cout << "inet_pton error! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    // 4. 连接
    status = connect(client_socket, (struct sockaddr *)&address, sizeof(address));
    if (status < 0)
    {
        cout << "connect failed! " << strerror(errno) << "错误码：" << errno << endl;
        return -1;
    }

    // 5. 接收消息
    int n = recv(client_socket, buff, MAXLINE, 0);
    buff[n] = '\0';
    cout << "recv msg from Server: " << buff << endl;

    // 6. 一直发送消息
    while (true)
    {
        cout << "Client: ";
        fgets(buff, MAXLINE, stdin);
        if (string(buff) == "quit\n")
            break;
        status = send(client_socket, buff, strlen(buff), 0);
        if (status < 0)
        {
            cout << "send msg failed! " << strerror(errno) << "错误码：" << errno << endl;
            return -1;
        }
    }

    // 7. 关闭客户端
    close(client_socket);
    return 0;
}
```

### 3.3. 运行效果

1. 编译：
``` shell
g++ Server.cpp -o Server
g++ Client.cpp -o Client
```

2. 先运行Server，再另起终端运行Client：
``` shell
./Server
```
``` shell
./Client
```

3. 在Client上面发送信息

![](/img/other/socket实现不同程序间的通信/C++运行示例.png)

## 4. C#实现socket通信

### 4.1. Server实现

``` c# Server.cs
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace tcpserver
{
    class server
    {
        [STAThread]
        static void Main(string[] args)
        {
            const int max_length = 4096;
            byte[] data = new byte[max_length]; //用于缓存客户端所发送的信息,通过socket传递的信息必须为字节数组
            // 1. 建立socket
            Socket server_socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            // 2. 设置ip和端口
            IPEndPoint ipep = new IPEndPoint(IPAddress.Any, 6666);//本机预使用的IP和端口
            // 3. 绑定端口
            server_socket.Bind(ipep);//绑定
            // 4. 监听端口
            server_socket.Listen(10);//监听
            Console.WriteLine("waiting for a client");
            // 5. 接受客户端的连接请求
            Socket client_connect = server_socket.Accept();//当有可用的客户端连接尝试时执行，并返回一个新的socket,用于与客户端之间的通信
            Console.WriteLine("some one connected!!");
            // 6. 发送消息
            string welcome = "welcome here!";
            data = Encoding.ASCII.GetBytes(welcome);   // 字符串转字节数组
            client_connect.Send(data, data.Length, SocketFlags.None);
            // 7. 一直接收消息
            while (true)
            {
                int n = client_connect.Receive(data);
                if (n == 0)     //当信息长度为0，说明客户端连接断开
                    break;
                string s = Encoding.ASCII.GetString(data, 0, n); // 字节数组转字符串
                Console.WriteLine("recv msg from client: " + s);
            }
            // 8. 关闭连接和socket
            client_connect.Close();
            server_socket.Close();
        }
    }
}
```

### 4.2. client实现

``` c# Client.cs
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace tcpclient
{
    class client
    {
        [STAThread]
        static void Main(string[] args)
        {
            const int max_length = 4096;
            byte[] data = new byte[max_length];

            // 1. 创建socket
            Socket client_socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            // 2. 设置端口和设置需要访问的ip
            IPEndPoint ie = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 6666);//服务器的IP和端口
            // 3. 连接
            try
            {
                //因为客户端只是用来向特定的服务器发送信息，所以不需要绑定本机的IP和端口。不需要监听。
                client_socket.Connect(ie);
            }
            catch (SocketException e)
            {
                Console.WriteLine("unable to connect to server");
                Console.WriteLine(e.ToString());
                return;
            }
            // 4. 接收消息
            int n = client_socket.Receive(data);
            string stringdata = Encoding.ASCII.GetString(data, 0, n);
            Console.WriteLine("recv msg from Server: " + stringdata);
            // 5. 一直发送消息
            while (true)
            {
                Console.Write("Client: ");
                string input = Console.ReadLine();
                if (input == "quit")
                    break;
                data = Encoding.ASCII.GetBytes(input);     // 字符串转字节数组
                client_socket.Send(data, data.Length, SocketFlags.None);
            }
            // 6. 关闭
            client_socket.Shutdown(SocketShutdown.Both);
            client_socket.Close();
        }
    }
}
```

### 4.3. 运行效果

1. 编译：
``` shell
mcs Server.cs
mcs Client.cs
```

2. 先运行Server，再另起终端运行Client：
``` shell
mono Server.exe
```
``` shell
mono Client.exe
```

3. 在Client上面发送信息

![](/img/other/socket实现不同程序间的通信/CSharp运行示例.png)

## 5. C++和C#之间的通信

文章开篇介绍过，socket实现的是两进程之间的通信，没必要server和client都是同一个语言实现。只要server和client之间的通信协议一致就能够保证通信。前文介绍的都是基于TCP协议的，故无论server的语言和client的语言是否一致都可以通信。

下面演示使用c++版本的server和c#版本的client进行通信。

![](/img/other/socket实现不同程序间的通信/混合通信示例.png)
