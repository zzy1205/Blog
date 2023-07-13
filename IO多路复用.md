
## 前言

对于一个 Client-Server 架构的数据库，其源码中少不了用来实现**网络通信**的部分。而涉及到网络通信，自然就涉及到 Socket 编程模型，包括**创建 Socket、监听端口、处理连接请求和读写请求。**

但是，由于基本的 Socket 编程模型**一次只能处理一个客户端连接上的请求**，所以当要处理高并发请求时，一种方案就是**使用多线程**，让每个线程负责处理一个客户端的请求，如下图：
<img src="pic/re-2.png" style="zoom:60%" align="center"/>

**每个连接使用单独的线程会导致线程开销巨大，服务端压力骤升。**

因此，为了实现高并发的网络通信，**Linux 操作系统提供了 select、poll 和 epoll 三种编程模型（都属于I/O多路复用模型的实现）**。

那么，要想理解 select、poll 和 epoll 的优势，我们需要有个对比基础，也就是基本的 Socket 编程模型。所以接下来，我们就先来了解下基本的 Socket 编程模型，以及它的不足之处。

### Socket编程模型
使用 Socket 模型实现网络通信时，需要经过创建 Socket、监听端口、处理连接和读写请求等多个步骤，现在我们就来具体了解下这些步骤中的关键操作，以此帮助我们分析 Socket 模型中的不足。

#### 为什么不使用Socket
首先，当我们需要让服务器端和客户端进行通信时，可以在服务器端通过以下三步，来创建监听客户端连接的监听套接字（Listening Socket）：
* 调用 socket 函数，创建一个套接字。我们通常把这个套接字称为主动套接字（Active Socket）；
* 调用 bind 函数，将主动套接字和当前服务器的 IP 和监听端口进行绑定；
* 调用 listen 函数，将主动套接字转换为监听套接字，开始监听客户端的连接。

在完成上述三步之后，服务器端就可以接收客户端的连接请求了。
为了能及时地收到客户端的连接请求，一般情况下会通过 while 循环处理，在循环中调用 accept 函数，用于接收客户端连接请求。

这里需要注意的是，**accept 函数是阻塞函数**，也就是说，**如果此时一直没有客户端连接请求，那么，服务器端的执行流程会一直阻塞在 accept 函数**。

一旦有客户端连接请求到达，accept 将不再阻塞，而是处理连接请求，和客户端建立连接，并返回已连接套接字（Connected Socket）。

```cpp
listenSocket = socket(); // 调用socket系统调用创建一个主动套接字
bind(listenSocket);     // 绑定地址和端口
listen(listenSocket);   // 将默认的主动套接字转换为服务器使用的被动套接字，也就是监听套接字
while(1) {             // 循环监听是否有客户端连接请求到来
   connSocket = accept(listenSocket); // 接受客户端连接
   recv(connsocket);    // 从客户端读取数据，只能同时处理一个客户端
   send(connsocket);    // 给客户端返回数据，只能同时处理一个客户端
}
```

能实现服务器与客户端的通信，但是程序每次调用一次accept函数，只能处理一个客户连接。

使用多线程，**在 accept 函数返回已连接套接字后，创建一个线程，并将已连接套接字传递给创建的线程**，由该线程负责这个连接套接字上后续的数据读写。同时，服务器端的执行流程会再次调用 accept 函数，等待下一个客户端连接。

```cpp
listenSocket = socket();    // 调用socket系统调用创建一个主动套接字
bind(listenSocket);         // 绑定地址和端口
listen(listenSocket);       // 将默认的主动套接字转换为服务器使用的被动套接字，即监听套接字
while(1) {                 // 循环监听是否有客户端连接到来
   connSocket = accept(listenSocket);       // 接受客户端连接，返回已连接套接字
   pthread_create(processData, connSocket); // 创建新线程对已连接套接字进行处理
   
}

// 处理已连接套接字上的读写请求
processData(connSocket){
   recv(connsocket);    // 从客户端读取数据，只能同时处理一个客户端
   send(connsocket);    // 给客户端返回数据，只能同时处理一个客户端
}
```

不过，虽然这种方法能提升服务器端的并发处理能力，遗憾的是，该方式非常消耗线程资源，**线程上下文切换的开销也非常大**。

<img src="pic/re-3.png" style="zoom:60%" align="center"/>

可以用线程池避免频繁创建线程
<img src="pic/re-4.png" style="zoom:60%" align="center"/>
线程池虽然解决了频繁创建线程的问题，但是，当出现很多长连接的时候，线程池资源很快被消耗殆尽，并且久久不能释放，于是相同的问题又出现了

这就要用到操作系统提供的 **IO 多路复用功能**了。可以让程序通过**调用多路复用函数，同时监听多个套接字上的请求**。

这里既可以包括监听套接字上的连接请求，也可以包括已连接套接字上的读写请求。
这样当有一个或多个套接字上有请求时，多路复用函数就会返回。此时，程序就可以处理这些就绪套接字上的请求，比如读取就绪的已连接套接字上的请求内容。

### I/O多路复用
<img src="pic/re-5.png" style="zoom:60%" align="center"/>

所谓 IO多路复用指的是用**一个用户线程就可以查询所有套接字（文件描述符）数据是否准备就绪**，所以，这里看到会分两步走：

* 先查询哪些文件描述符对应的 IO 事件准备就绪了，这里的查询可以通过 select、poll、epoll 等系统调用
* 最后将准备就绪的套接字，通过 read 或者 write 等系统调用进行实际的读写操作。

在深入了解之前，我们需要知道，对于一种 IO 多路复用机制来说，我们需要掌握哪些要点，这样可以帮助我们快速抓住不同机制的联系与区别：

* 第一，多路复用机制会监听套接字上的哪些事件？
* 第二，多路复用机制可以监听多少个套接字？
* 第三，当有套接字就绪时，多路复用机制要如何找到就绪的套接字？

#### select机制实现IO多路复用
定义
```cpp
int select (int __nfds, fd_set *__readfds, fd_set *__writefds, fd_set *__exceptfds, struct timeval *__timeout)
```
参数
* 监听的文件描述符数量 __nfds
* 被监听描述符的三个集合 __readfds、__writefds 和 __exceptfds
* 监听时阻塞等待的超时时长 __timeout。

Unix 哲学中，一切皆文件。 
因此，**Linux 针对每一个套接字都会有一个文件描述符，也就是一个非负整数，用来唯一标识该套接字**。

所以，在多路复用机制的函数中，Linux 通常会**用文件描述符作为参数**。有了文件描述符，函数也就能找到对应的套接字，进而进行监听、读写等操作。

在方法参数中，被监听描述符的集合，其实就是被监听套接字的集合。那么，为什么会有三个集合呢？
这就和刚才提出的第一个问题相关，也就是多路复用机制会监听哪些事件。

**select 函数使用三个集合，表示监听的三类事件，分别是 读数据事件、写数据事件、异常事件。**

我们进一步可以看到，三个被监听的集合的类型是 fd_set 结构体，它主要定义部分如下所示：
```cpp
typedef struct {
   …
   __fd_mask  __fds_bits[__FD_SETSIZE / __NFDBITS];
   …
} fd_set
```
其中
* __fd_mask 类型是 long int 类型的别名
* __FD_SETSIZE 和 __NFDBITS 这两个宏定义的大小默认为 1024 和 32

fd_set 结构体的定义，其实就是一个 long int 类型的数组，该数组中一共有 32 个元素（1024/32=32），每个元素是 32 位（long int 类型的大小），而每一位可以用来表示一个文件描述符的状态

意味着，**select 函数对每一个描述符集合，都可以监听 1024 个描述符。**

#### select 机制实现网络通信
**首先，通过 select 查询哪些套接字处于就绪状态**
首先，我们在调用 select 函数前，可以先创建好传递给 select 函数的描述符集合，然后再创建监听套接字。

而为了让创建的监听套接字能被 select 函数监控，我们需要把这个套接字的描述符加入到创建好的描述符集合中。

然后，我们就可以调用 select 函数，并把创建好的描述符集合作为参数传递给 select 函数。程序在调用 select 函数后，会发生阻塞。

而当 select 函数检测到有描述符就绪后，就会结束阻塞，并返回就绪的文件描述符个数。那么此时，我们就可以在描述符集合中查找哪些描述符就绪了。然后，我们对已就绪描述符对应的套接字进行处理。

**然后，通过系统调用读、写数据**
比如，如果是 __readfds 集合中有描述符就绪，这就表明这些就绪描述符对应的套接字上，有读事件发生，此时，我们就在该套接字上读取数据。

而因为 select 函数一次可以监听 1024 个文件描述符的状态，所以 select 函数在返回时，也可能会一次返回多个就绪的文件描述符。这样一来，我们就可以使用一个循环流程，依次对就绪描述符对应的套接字进行读写或异常处理操作。

下面的代码展示的是使用 select 函数，进行并发客户端处理的关键步骤和主要函数调用：
```cpp
int sock_fd, conn_fd;    // 监听套接字和已连接套接字的变量
sock_fd = socket()      // 创建套接字
bind(sock_fd)           // 绑定套接字
listen(sock_fd)         // 在套接字上进行监听，将套接字转为监听套接字

fd_set rset;            // 被监听的描述符集合，关注描述符上的读事件
 
int max_fd = sock_fd

// 初始化rset数组，使用FD_ZERO宏设置每个元素为0 
FD_ZERO(&rset);
// 使用FD_SET宏设置rset数组中位置为sock_fd的文件描述符为1，表示需要监听该文件描述符
FD_SET(sock_fd, &rset);

// 设置超时时间 
struct timeval timeout;
timeout.tv_sec = 3;
timeout.tv_usec = 0;
 
while(1) {
   // 调用select函数，检测rset数组保存的文件描述符是否已有读事件就绪，返回就绪的文件描述符个数
   n = select(max_fd + 1, &rset, NULL, NULL, &timeout);
 
   // 调用FD_ISSET宏，在rset数组中检测sock_fd对应的文件描述符是否就绪
   if (FD_ISSET(sock_fd, &rset)) {
       // 如果sock_fd已经就绪，表明已有客户端连接；调用accept函数建立连接
       conn_fd = accept();
       // 设置rset数组中位置为conn_fd的文件描述符为1，表示需要监听该文件描述符
       FD_SET(conn_fd, &rset);
   }

   // 依次检查已连接套接字的文件描述符
   for (i = 0; i < maxfd; i++) {
        // 调用FD_ISSET宏，在rset数组中检测文件描述符是否就绪
       if (FD_ISSET(i, &rset)) {
         // 有数据可读，进行读数据处理
       }
   }
}
```

**不足**：
* 首先，select 函数对单个进程能监听的文件描述符数量是有限制的，它能监听的文件描述符个数由 __FD_SETSIZE 决定，默认值是 1024。
* 其次，当 select 函数返回后，我们需要遍历描述符集合，才能找到具体是哪些描述符就绪了。这个遍历过程会产生一定开销，从而降低程序的性能。

#### poll机制实现IO多路复用
```cpp
int poll (struct pollfd *__fds, nfds_t __nfds, int __timeout);
```
其中
* __fds 是 pollfd 结构体数组
* __nfds 表示的是 *__fds 数组的元素个数
* __timeout 表示 poll 函数阻塞的超时时间。

pollfd 结构体中包含了三个成员变量 fd、events 和 revents，分别表示要监听的文件描述符、要监听的事件类型和实际发生的事件类型。

```cpp
struct pollfd {
    int fd;         //进行监听的文件描述符
    short int events;       //要监听的事件类型
    short int revents;      //实际发生的事件类型
};
```

其中，要监听和实际发生的事件类型，是通过以下三个宏定义来表示的，分别是 POLLRDNORM、POLLWRNORM 和 POLLERR，它们分别表示可读、可写和错误事件。

```cpp
#define POLLRDNORM  0x040       //可读事件
#define POLLWRNORM  0x100       //可写事件
#define POLLERR     0x008       //错误事件
```

网络通信流程
* 第一步，创建 pollfd 数组和监听套接字，并进行绑定；
* 第二步，将监听套接字加入 pollfd 数组，并设置其监听读事件，也就是客户端的连接请求；
* 第三步，循环调用 poll 函数，检测 pollfd 数组中是否有就绪的文件描述符。

而在第三步的循环过程中，其处理逻辑又分成了两种情况：

* 如果是连接套接字就绪，这表明是有客户端连接，我们可以调用 accept 接受连接，并创建已连接套接字，并将其加入 pollfd 数组，并监听读事件；
* 如果是已连接套接字就绪，这表明客户端有读写请求，我们可以调用 recv/send 函数处理读写请求。

**总的来说，流程上和 select 类似，先查询哪些套接字处于就绪状态（使用poll方法查询），然后通过系统调用进行事件的读写。**

```cpp
int sock_fd,conn_fd; //监听套接字和已连接套接字的变量
sock_fd = socket() //创建套接字
bind(sock_fd)   //绑定套接字
listen(sock_fd) //在套接字上进行监听，将套接字转为监听套接字

//poll函数可以监听的文件描述符数量，可以大于1024
#define MAX_OPEN = 2048

//pollfd结构体数组，对应文件描述符
struct pollfd client[MAX_OPEN];

//将创建的监听套接字加入pollfd数组，并监听其可读事件
client[0].fd = sock_fd;
client[0].events = POLLRDNORM; 
maxfd = 0;

//初始化client数组其他元素为-1
for (i = 1; i < MAX_OPEN; i++)
    client[i].fd = -1; 

while(1) {
   //调用poll函数，检测client数组里的文件描述符是否有就绪的，返回就绪的文件描述符个数
   n = poll(client, maxfd+1, &timeout);
   //如果监听套件字的文件描述符有可读事件，则进行处理
   if (client[0].revents & POLLRDNORM) {
       //有客户端连接；调用accept函数建立连接
       conn_fd = accept();

       //保存已建立连接套接字
       for (i = 1; i < MAX_OPEN; i++){
         if (client[i].fd < 0) {
           client[i].fd = conn_fd; //将已建立连接的文件描述符保存到client数组
           client[i].events = POLLRDNORM; //设置该文件描述符监听可读事件
           break;
          }
       }
       maxfd = i; 
   }
   
   //依次检查已连接套接字的文件描述符
   for (i = 1; i < MAX_OPEN; i++) {
       if (client[i].revents & (POLLRDNORM | POLLERR)) {
         //有数据可读或发生错误，进行读数据处理或错误处理
       }
   }
}

```
**其实，和 select 函数相比，poll 函数的改进之处主要就在于，它允许一次监听超过 1024 个文件描述符。但是当调用了 poll 函数后，我们仍然需要遍历每个文件描述符，检测该描述符是否就绪，然后再进行处理。**

### epoll 机制实现 IO 多路复用
epoll 在 Linux2.6 内核正式提出，是基于事件驱动的 I/O 方式，相对于 select 来说，epoll 没有描述符个数限制，使用一个文件描述符管理多个描述符，将用户关心的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的 COPY 只需一次。

* 函数创建一个epoll句柄，参数size表明内核要监听的描述符数量：
```cpp
int epoll_create(int size);
```
* epoll_ctl 函数注册要监听的事件类型
```cpp
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
  1. epfd 表示 epoll 句柄
  2. op 表示 fd 操作类型，有如下3种
    EPOLL_CTL_ADD 注册新的 fd 到 epfd 中
    EPOLL_CTL_MOD 修改已注册的 fd 的监听事件
    EPOLL_CTL_DEL 从 epfd 中删除一个 fd
  3. fd 是要监听的描述符
  4. event 表示要监听的事件
```
* epoll_wait 函数等待事件的就绪，成功时返回就绪的事件数目，调用失败时返回 -1，等待超时返回 0：
```cpp
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
* epfd 是 epoll 句柄
* events 表示从内核得到的就绪事件集合
* maxevents 告诉内核 events 的大小
* timeout 表示等待的超时事件
```

首先，epoll 机制是使用 epoll_event 结构体，来记录待监听的文件描述符及其监听的事件类型的，这和 poll 机制中使用 pollfd 结构体比较类似。

那么，对于 epoll_event 结构体来说，其中包含了 epoll_data_t 联合体变量，以及整数类型的 events 变量。
```cpp
typedef union epoll_data
{
    ...
    int fd;  // 记录文件描述符
    ...
} epoll_data_t;

struct epoll_event
{
    // EPOLLIN：读事件，表示文件描述符对应套接字有数据可读。
    // EPOLLOUT：写事件，表示文件描述符对应套接字有数据要写。
    // EPOLLERR：错误事件，表示文件描述符对于套接字出错。
    uint32_t events;   // epoll监听的事件类型
    epoll_data_t data; // 应用程序数据
};

```

epoll_data_t 联合体中有记录文件描述符的成员变量 fd，而 events 变量会取值使用不同的宏定义值，来**表示 epoll_data_t 变量中的文件描述符所关注的事件类型**，比如一些常见的事件类型包括以下这几种。

步骤：
epoll_create 函数创建一个 epoll 句柄，由内核进行维护。
epoll_ctl 函数注册要监听的套接字、事件类型。
epoll_wait 函数等待事件的就绪，成功时返回就绪的事件数目，并且通过引用返回就绪状态的 fd。

epoll 实例内部维护了两个结构，分别是记录「要监听的文件描述符」和「已经就绪的文件描述符」，而对于已经就绪的文件描述符来说，它们会被返回给用户程序进行处理（引用返回）。

值得说明的是，**这里「已就绪的文件描述符」是由内核主动添加至就绪文件描述符集合中，我们从用户态调用 epoll_wait 就直接查询该集合是否有就绪 I/O 事件，这样下来，就减少了全遍历所有文件描述符的操作。**

所以，我们在使用 epoll 机制时，就不用像使用 select 和 poll 一样，遍历查询哪些文件描述符已经就绪了。这样一来， epoll 的效率就比 select 和 poll 有了更高的提升。

```cpp
int sock_fd,conn_fd; // 监听套接字和已连接套接字的变量
sock_fd = socket()   // 创建套接字
bind(sock_fd)        // 绑定套接字
listen(sock_fd)      // 在套接字上进行监听，将套接字转为监听套接字
    
epfd = epoll_create(EPOLL_SIZE); // 创建epoll实例，
// 创建epoll_event结构体数组，保存套接字对应文件描述符和监听事件类型    
ep_events = (epoll_event* )malloc(sizeof(epoll_event) * EPOLL_SIZE);

// 创建epoll_event变量
struct epoll_event ee
// 监听读事件
ee.events = EPOLLIN;
// 监听的文件描述符是刚创建的监听套接字
ee.data.fd = sock_fd;

// 将监听套接字加入到监听列表中    
epoll_ctl(epfd, EPOLL_CTL_ADD, sock_fd, &ee); 
    
while (1) {
   // 等待返回已经就绪的描述符 
   n = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1); 
   // 遍历所有就绪的描述符     
   for (int i = 0; i < n; i++) {
       // 如果是监听套接字描述符就绪，表明有一个新客户端连接到来 
       if (ep_events[i].data.fd == sock_fd) { 
          conn_fd = accept(sock_fd); // 调用accept()建立连接
          ee.events = EPOLLIN;  
          ee.data.fd = conn_fd;
          // 添加对新创建的已连接套接字描述符的监听，监听后续在已连接套接字上的读事件      
          epoll_ctl(epfd, EPOLL_CTL_ADD, conn_fd, &ee); 
                
       } else { //如果是已连接套接字描述符就绪，则可以读数据
           ...//读取数据并处理
       }
   }
}

```
实际上，也正是因为 epoll 能自定义监听的描述符数量，以及可以直接返回就绪的描述符，我们 在设计和实现网络通信框架时，就可以基于 epoll 机制中的 epoll_create、epoll_ctl 和 epoll_wait 等函数和读写事件，进行封装和开发，实现用于网络通信的事件驱动框架，这是目前高性能 IO 的必要条件

### 总结
**select 的出现解决资源高效利用，一个线程可以搞定多个线程做的事情。
poll 解决 select 中出现的1024文件描述符的限制。
epoll 的出现则解决poll 中需要遍历所有文件描述符的问题。**