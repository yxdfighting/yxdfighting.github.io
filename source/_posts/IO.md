---
title: IO
---
##关于阻塞IO与非阻塞IO
以read为例子，共分为两步：
1、内核缓冲区数据准备好 
2、把数据从内核缓冲区复制到用户空间

>阻塞IO主要是在调用read函数后，如果内核缓冲区数据还没有准备好，那么read函数不返回，一直阻塞。
>非阻塞IO在调用read函数后，如果内核缓冲区数据还没有准备好，那么read函数立即返回负值，如果要实现轮询，需要用户自己实现这部分逻辑。

>同步IO和异步IO的主要区别是IO执行的主体不同，同步IO主要由用户自己实现IO，调用read，判断返回值，处理逻辑，所以上面所说的阻塞与非阻塞IO都是同步IO；而异步IO则不同，只需要用户调用read之后，剩余的工作交给内核，当内核缓冲区数据准备好之后，就会自己执行回调函数，完成工作。

>多路IO复用，一个线程（进程）可以监听多个文件描述符，一旦某个描述符就绪(读事件或者写事件)，能够有这种机制可以通知程序进行读写操作，但select、poll、epoll本质上都是同步IO，因为整个读写操作都是在收到通知后用户自己执行的。

>epoll、poll、select三者都能提供I/O多路复用的解决方案，在现在linux内核中都有支持，其中epoll是linux系统特有的，而select、poll是支持所有系统的。

#### select
select函数监视的文件描述符有三类，分别是readfds、writefds、exceptfds，每个描述符存储在一个fd_set数据类型中
int select(int maxfdp1,fd_set * restrict readfds,fd_set * restrict writefds,fd_set * restrict exceptfds,struct tmeval *restrict tvptr);
返回值：准备就绪的描述符数目；若超时，返回0；出错返回-1
需要对我们关心的fd进行设置，有一组四个函数进行设置，先对几个fd_set进行清零，然后可以对我们关心的描述符进行置位,select调用结束后，fd_isset查看相应为是否仍为1。select本质上是通过设置或者检查存放fd标志位的数据结构进行下一步处理。
>1、单个进程能够监听的fd有限

>2、本质上是对socket的扫描时轮询方式，即对fd_set中置位的socket进行遍历

>3、需要维护一个存放fd的数组，并且轮询需要在内核中进行，而又需要将结果传回用户空间，所以在二者之间传递该结构时会造成较大的开销

#### poll
poll函数类似于select，只是接口有所不同。
> int poll(struct pollfd fdarray[],nfds_t nfds,int timeout);

>struct pollfd{int fd;short event;short revents};

每个数组元素指定一个描述符编号以及我们对该描述符感兴趣的条件，poll将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历多次无谓的遍历。

它没有最大连接数的限制，原因是它基于链表存储的，但是同样有一个缺点：
> 存储大量fd的链表被整体复制与用户态和内核地址空间之间，不管这样的复制是不是有意义。

> poll还有一个特点是“水平触发”，如果报告了fd之后，没有被处理，那么下次poll时还会再次报告该fd。

> 随着监视的描述符数量的增长，效率直线下降。
#### epoll
epoll精巧的使用了3个方法来实现select方法要做的事：
> 
1.新建epoll描述符==epoll_create()
2.epoll_ctrl(epoll描述符，添加或者删除所有待监控的连接)
3.返回的活跃连接 ==epoll_wait（ epoll描述符 ）

 与select相比，epoll分清了频繁调用和不频繁调用的操作。例如，epoll_ctrl是不太频繁调用的，而epoll_wait是非常频繁调用的。这时，epoll_wait却几乎没有入参，这比select的效率高出一大截，而且，它也不会随着并发连接的增加使得入参越发多起来，导致内核执行效率下降。

> 
epoll三大关键要素：mmap、红黑树、链表
mmap映射区将内核的一个地址与用户空间的一个地址映射到同一块物理内存地址，减少用户态和内核态之间的数据交换。内核可以直接看到用户空间epoll监听的句柄，效率高；对于要监听的socket，epoll采用红黑树进行存储，添加或者删除epoll都是对红黑树进行操作，所以效率较高，时间复杂度为O（logn）；epoll将添加进来的新事件存储到红黑树上，所以不会出现重复添加，对于已经就绪的事件采用双向链表来存储，同时建立回调机制，每当有新的事件被触发，就会执行回调函数，把相应的事件添加到双向链表中，然后将其复制到用户空间（相对于select、poll需要将所有文件描述符从内核复制到用户空间，epoll只需要将就绪的文件描述符及其事件进行复制即可，效率得到极大提升）。

>epoll的使用
int epoll_create(int size);
int epoll_ctl ( int epfd, int op, int fd, struct epoll_event *event );
int epoll_wait ( int epfd, struct epoll_event* events, int maxevents, int timeout );
>
函数说明：
     fd：要操作的文件描述符
     op：指定操作类型
操作类型：
     EPOLL_CTL_ADD：往事件表中注册fd上的事件
     EPOLL_CTL_MOD：修改fd上的注册事件
     EPOLL_CTL_DEL：删除fd上的注册事件
     event：指定事件，它是epoll_event结构指针类型


> struct epoll_event
{
     __unit32_t events;    // epoll事件
     epoll_data_t data;     // 用户数据 
};

>typedef union epoll_data
 {
     void* ptr;              //指定与fd相关的用户数据 
     int fd;                 //指定事件所从属的目标文件描述符 
     uint32_t u32;
     uint64_t u64;
} epoll_data_t;

>epoll_wait函数说明：
     返回：成功时返回就绪的文件描述符的个数，失败时返回-1并设置errno
     timeout：指定epoll的超时时间，单位是毫秒。当timeout为-1是，epoll_wait调用将永远阻塞，直到某个时间发生。当timeout为0        时,epoll_wait调用将立即返回。
     maxevents：指定最多监听多少个事件
     events：检测到事件，将所有就绪的事件从内核事件表中复制到它的第二个参数events指向的数组中。

>参考http://blog.chinaunix.net/uid-28541347-id-4273856.html
http://www.cnblogs.com/lojunren/p/3856290.html