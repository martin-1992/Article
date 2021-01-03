### 了解 Epoll
　　要理解 Epoll 的本质，需要先了解几个流程：

- 网卡接收数据的过程；
    1. 数据先发送到网卡，网卡接收；
    2. 然后，存到内存；
    3. 系统从内存中读取。
- CPU 如何中断来接收数据；
    1. 程序执行有优先级；
    2. 硬件产生的信号，优先级高，CPU 会中断当前正在执行的程序，做出相应；
    3. 当 CPU 完成对硬件的响应后，再重新执行用户程序；
    4. 网卡收到数据后，要写入内存中，会向 CPU 发出一个中断信号。CPU 通过网卡的中断信号，中断当前应用程序的执行，去将新数据写入到内存中。
- 操作系统如何调整进程调度来接收数据；
    1. 进程调度是通过阻塞实现的，Recv、Select 和 Epoll 都是阻塞方法；

#### 阻塞原理
　　操作系统执行程序时，并不是执行完一个再执行另一个的，而是每个进程各执行一段时间，来回切换。因为执行速度快，所以看上去是同时执行多个任务。比如有 3 个进程，为 A、B、C，操作系统先执行 A 进程 5ms，不管 A 进程是否执行完，会切换到 B 进程执行 5ms，然后切换到 C，再切换回 A。<br />
　　有两个队列，分别为工作队列和等待队列。在进程调度中，会将进程分为运行状态和等待状态。操作系统通过来回切换，来执行有运行状态的多个进程。而处于等待状态的进程，则放入等待队列中，为阻塞状态，系统不会执行。所以进程阻塞不会浪费 CPU 资源，CPU 会执行其他运行状态的进程。<br />
　　前面提到的中断信号，即是操作系统将运行的程序放入等待队列中，来执行写入内存的程序。当写入完成后，在唤醒等待队列中的程序，移回工作队列中，继续执行。

### 监视多个 Socket 方法
　　一个 Socket 对应一个端口号，操作系统维护端口号到 Socket 的索引结构。Select 和 Epoll 都能监视多个 Socket，Epoll 更高效。

### Select 方法
　　下面代码模拟 Select 的调用，fds 存放 Socket 列表。遍历所有 Socket 都没数据时，Select 方法会阻塞，即放到等待队列中。注意，假设有 3 个 Socket，那么该进程（Select 方法）会分别加入这三个 Socket 的等待队列中。<br />
　　当 Socket 有数据时，发出中断信号，唤醒进程，将 Select 方法从等待队列中移到工作队列中。然后遍历一遍 Socket，找到哪个 Socket 接收数据，进行处理。同样，唤醒时，会将该进程从三个 Socket 中的等待队列中移回到三个 Socket 中的工作队列中。

- 第一次遍历，看 Socket 列表中是否有数据，没数据 Select 移到等待队列中；
- 第二次遍历，Socket 有数据，Select 方法需遍历找到有数据的那个 Socket。

```c
int s = socket(AF_INET, SOCK_STREAM, 0);   
bind(s, ...) 
listen(s, ...) 
 
int fds[] =  存放需要监听的 socket 
 
while(1){ 
    int n = select(..., fds, ...) 
    for(int i=0; i < fds.count; i++){ 
        if(FD_ISSET(fds[i], ...)){ 
            //fds[i]的数据处理 
        } 
    } 
}
```

　　在调用 Select 时，遍历 Socket 列表有数据时，则不会阻塞，而是直接返回。如果有多个 Socket 有数据时，Select 返回的值就大于 1。

#### Select 低效原因

- 维护等待队列和阻塞进程，这两个步骤合在一起；
- 唤醒进程后，还需遍历 Socket。

### Epoll 方法
　　针对 Select 低效原因进行改进，第一点是拆分了功能，使用 epoll_ctl 维护等待队列，再调用 epoll_wait 阻塞进程。<br />
　　如下代码，epoll_create 创建一个 Epoll 对象 Epfd，再通过 epoll_ctl 将需要监视的 Socket 添加到 Epfd 中，最后调用 epoll_wait 等待数据。

- 某个进程调用 epoll_create 方法时，内核会创建一个 eventpoll 对象，即代码中的 epfd 对象；
    1. eventpoll 对象有个就绪列表 Rdlist，为存放收到数据 Socket 引用的列表；
    2. eventpoll 对象也是文件系统中的一员，和 Socket 一样，它也会有等待队列。
- 通过 epoll_ctl 添加要监听的 Socket，到 epfd 中；
    1. 当通过 epoll_ctl 添加 Sock1、Sock2 和 Sock3 的监视，内核会将 eventpoll 添加到这三个 Socket 的等待队列中；
    2. Socket 收到数据后，中断程序会操作 eventpoll 对象，给 eventpoll 的就绪列表 Rdlist 添加 Socket 引用。
- 程序执行到 epoll_wait 时，如果就绪列表 Rdlist 已经引用了 Socket，那么 epoll_wait 直接返回，如果 Rdlist 为空，阻塞进程。
    1. Socket 收到数据后，中断程序一方面修改 Rdlist，另一方面唤醒 eventpoll 等待队列中的进程，移到工作队列中；
    2. 没数据的话，内核会将进程放入 eventpoll 的等待队列中，阻塞进程。

　　就绪队列是使用双向链表结构实现的。Epoll 在 Select 和 Poll 的基础上引入了 eventpoll 作为中间层，使用了先进的数据结构，是一种高效的多路复用技术。

### reference 

- [Epoll 原理解析，推荐看这篇](https://blog.csdn.net/armlinuxww/article/details/92803381)
