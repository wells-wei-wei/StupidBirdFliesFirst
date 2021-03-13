<!-- TOC -->

- [1. IO](#1-io)
  - [1.1. IO种类](#11-io种类)
    - [1.1.1. 异步IO](#111-异步io)
    - [1.1.2. 信号驱动IO](#112-信号驱动io)
    - [1.1.3. 阻塞IO](#113-阻塞io)
    - [1.1.4. 非阻塞IO](#114-非阻塞io)
    - [1.1.5. IO多路复用](#115-io多路复用)
  - [1.2. 同步/异步，阻塞/非阻塞](#12-同步异步阻塞非阻塞)
  - [1.3. linux分为用户态和内核态：](#13-linux分为用户态和内核态)
  - [1.4. 套接字](#14-套接字)
  - [1.5. 文件描述符](#15-文件描述符)
- [2. Linux的IO多路复用](#2-linux的io多路复用)
  - [2.1. select](#21-select)
  - [2.2. poll](#22-poll)
  - [2.3. epoll](#23-epoll)

<!-- /TOC -->
# 1. IO
先解释一下listen和accept的一些问题。listen所输入的描述符代表的是一个监听的事件，他也只能作为监听，当有一个连接到来时会使用accept新建一个文件描述符，这个描述符是用来处理连接的，他也只能处理连接，跟前面那个listen一点关系都没有。就好像餐厅门口的礼仪小姐，当有客人来了以后会把客人交给店内的其他服务员，然后自己再继续站在门口。
## 1.1. IO种类
### 1.1.1. 异步IO
### 1.1.2. 信号驱动IO
### 1.1.3. 阻塞IO
服务端采用单线程，当accept一个请求后，在recv或send调用阻塞时，将无法accept其他请求（必须等上一个请求处recv或send完），无法处理并发。如果一定要并发，当accept一个请求后，可以开启新线程进行recv，可以完成并发处理，但随着请求数增加需要增加系统线程，大量的线程占用很大的内存空间，并且线程切换会带来很大的开销，10000个线程真正发生读写事件的线程数不会超过20%，每次accept都开一个线程也是一种资源浪费

### 1.1.4. 非阻塞IO
服务器端当accept一个请求后，加入fds集合，每次轮询一遍fds集合recv(非阻塞)数据，没有数据则立即返回错误。这种做法其实就是把所有的连接记录下来，放到一个列表里，每次轮询所有fd（就用for，包括没有发生读写事件的fd）会很浪费cpu

### 1.1.5. IO多路复用
IO多路复用是一种同步IO模型，实现一个线程可以监视多个文件句柄；一旦某个文件句柄就绪，就能够通知应用程序进行相应的读写操作；没有文件句柄就绪时会阻塞应用程序，交出cpu。多路是指网络连接，复用指的是同一个线程。多路复用是目前最常用的一种处理高并发服务器的IO模式。

## 1.2. 同步/异步，阻塞/非阻塞
拿烧水举例：

同步/异步关注的是水烧开之后需不需要我来处理。

阻塞/非阻塞关注的是在水烧开的这段时间是不是干了其他事。
	
点火后，傻等，不等到水开坚决不干任何事（阻塞），水开了关火（同步）。

点火后，去看电视（非阻塞），时不时看水开了没有，水开后关火（同步）。

按下开关后，傻等水开（阻塞），水开后自动断电（异步）。

按下开关后，该干嘛干嘛 （非阻塞），水开后自动断电（异步）。

## 1.3. linux分为用户态和内核态：
内核负责网络和文件数据的读写。

用户程序通过系统调用获得网络和文件的数据。

## 1.4. 套接字
应用程序通过系统调用socket(),建立连接，接收和发送数据（I / O）。

SOCKET 支持了非阻塞，应用程序才能非阻塞调用，支持了异步，应用程序才能异步调用

## 1.5. 文件描述符
linux中万物都是文件，文件描述符（fd）就是对于内核态中文件的引用是内核为了高效管理已被打开的文件所创建的索引，其是一个非负整数（通常是小整数），用于指代被打开的文件，所有执行I/O操作的系统调用都通过文件描述符。程序刚刚启动的时候，0是标准输入，1是标准输出，2是标准错误。如果此时去打开一个新的文件，它的文件描述符会是3。

文件描述符是系统的一个重要资源，虽然说系统内存有多少就可以打开多少的文件描述符，但是在实际实现过程中内核是会做相应的处理的，一般最大打开文件数会是系统内存的10%（以KB来计算）（称之为系统级限制），查看系统级别的最大打开文件数可以使用sysctl -a | grep fs.file-max命令查看。与此同时，内核为了不让某一个进程消耗掉所有的文件资源，其也会对单个进程最大打开文件数做默认值处理（称之为用户级限制），默认值一般是1024，使用ulimit -n命令可以查看。

每一个文件描述符会与一个打开文件相对应，同时，不同的文件描述符也会指向同一个文件。相同的文件可以被不同的进程打开也可以在同一个进程中被多次打开。系统为每一个进程维护了一个文件描述符表，该表的值都是从0开始的，所以在不同的进程中你会看到相同的文件描述符，这种情况下相同文件描述符有可能指向同一个文件，也有可能指向不同的文件。具体情况要具体分析，要理解具体其概况如何，需要查看由内核维护的3个数据结构。


# 2. Linux的IO多路复用
## 2.1. select
select的原理其实跟非阻塞IO是一样的，但是前面的NIO有连接之后自己遍历获取发生事件的客户端，自己用户态遍历切换到核心态看是否有事件，10000次的话需要核心态切换10000次，效率低。select是把所有客户端连接传递给内核，内核遍历返回可读可写的，减少了系统调用的次数，也就是可以把10000个客户端的连接传给操作系统，返回所有连接中有事件发生的连接，程序再自己读取事件(程序自己读取IO)，获取状态原来10000个连接需要10000次系统调用，select一次系统调用即可获取状态。
```c++
#include <sys/types.h> 
#include <sys/socket.h> 
#include <stdio.h> 
#include <netinet/in.h> 
#include <sys/time.h> 
#include <sys/ioctl.h> 
#include <unistd.h> 
#include <stdlib.h>

int main() 
{ 
    int server_sockfd, client_sockfd; 
    int server_len, client_len; 
    struct sockaddr_in server_address; 
    struct sockaddr_in client_address; 
    int result; 
    fd_set readfds, testfds; 
    server_sockfd = socket(AF_INET, SOCK_STREAM, 0);//建立服务器端socket 
    server_address.sin_family = AF_INET; 
    server_address.sin_addr.s_addr = htonl(INADDR_ANY); 
    server_address.sin_port = htons(8888); 
    server_len = sizeof(server_address); 
    bind(server_sockfd, (struct sockaddr *)&server_address, server_len); 
    listen(server_sockfd, 5); //监听队列最多容纳5个 
    FD_ZERO(&readfds); 
    FD_SET(server_sockfd, &readfds);//将服务器端socket加入到集合中
    while(1) 
    {
        char ch; 
        int fd; 
        int nread; 
        testfds = readfds;//将需要监视的描述符集copy到select查询队列中，select会对其修改，所以一定要分开使用变量 
        printf("server waiting\n"); 

        /*无限期阻塞，并测试文件描述符变动 */
        result = select(FD_SETSIZE, &testfds, (fd_set *)0,(fd_set *)0, (struct timeval *) 0); //FD_SETSIZE：系统默认的最大文件描述符
        if(result < 1) 
        { 
            perror("server5"); 
            exit(1); 
        } 

        /*扫描所有的文件描述符*/
        for(fd = 0; fd < FD_SETSIZE; fd++) 
        {
            /*找到相关文件描述符*/
            if(FD_ISSET(fd,&testfds)) 
            { 
                /*判断是否为服务器套接字，是则表示为客户请求连接。*/
                if(fd == server_sockfd) 
                { 
                    client_len = sizeof(client_address); 
                    client_sockfd = accept(server_sockfd, 
                    (struct sockaddr *)&client_address, &client_len); 
                    FD_SET(client_sockfd, &readfds);//将客户端socket加入到集合中
                    printf("adding client on fd %d\n", client_sockfd); 
                } 
                /*客户端socket中有数据请求时*/
                else 
                { 
                    ioctl(fd, FIONREAD, &nread);//取得数据量交给nread
                    
                    /*客户数据请求完毕，关闭套接字，从集合中清除相应描述符 */
                    if(nread == 0) 
                    { 
                        close(fd); 
                        FD_CLR(fd, &readfds); //去掉关闭的fd
                        printf("removing client on fd %d\n", fd); 
                    } 
                    /*处理客户数据请求*/
                    else 
                    { 
                        read(fd, &ch, 1); 
                        sleep(5); 
                        printf("serving client on fd %d\n", fd); 
                        ch++; 
                        write(fd, &ch, 1); 
                    } 
                } 
            } 
        } 
    } 

    return 0;
}
```

## 2.2. poll
有了select之后还有poll主要是因为selec有最大连接数的限制，但是poll没有，所以理论上可以无限使用。
```c++
#include <stdio.h>
#include <string.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <poll.h>
    
/*环境为ubuntu10.04自带c环境，无法自动引入下列宏，所以自己写在前面了*/
#define INFTIM -1
#define POLLRDNORM  0x040       /* Normal data may be read.  */
#define POLLRDBAND  0x080       /* Priority data may be read.  */
#define POLLWRNORM  0x100       /* Writing now will not block.  */
#define POLLWRBAND  0x200       /* Priority data may be written.  */
    
#define MAXLINE  1024
#define OPEN_MAX  16 //一些系统会定义这些宏
#define SERV_PORT  10001
    
int main()
{
    int i , maxi ,listenfd , connfd , sockfd ;
    int nready;
    int n;
    char buf[MAXLINE];
    socklen_t clilen;
    struct pollfd client[OPEN_MAX];
    
    struct sockaddr_in cliaddr , servaddr;
    listenfd = socket(AF_INET , SOCK_STREAM , 0);
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERV_PORT);
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    
    bind(listenfd , (struct sockaddr *) & servaddr, sizeof(servaddr));
    listen(listenfd,10);
    client[0].fd = listenfd;
    client[0].events = POLLRDNORM;
    for(i=1;i<OPEN_MAX;i++)
    {
        client[i].fd = -1;
    }
    maxi = 0;
    
    for(;;)
    {
        nready = poll(client,maxi+1,INFTIM);
        if (client[0].revents & POLLRDNORM)
        {
            clilen = sizeof(cliaddr);
            connfd = accept(listenfd , (struct sockaddr *)&cliaddr, &clilen);
            for(i=1;i<OPEN_MAX;i++)
            {
                if(client[i].fd<0)
                {
                    client[i].fd = connfd;
                    client[i].events = POLLRDNORM;
                    break;
                }
            }
            if(i==OPEN_MAX)
            {
                printf("too many clients! \n");
            }
            if(i>maxi) maxi = i;
            nready--;
            if(nready<=0) continue;
        }
    
        for(i=1;i<=maxi;i++)
        {
            if(client[i].fd<0) continue;
            sockfd = client[i].fd;
            if(client[i].revents & (POLLRDNORM|POLLERR))
            {
                n = read(client[i].fd,buf,MAXLINE);
                if(n<=0)
                {
                    close(client[i].fd);
                    client[i].fd = -1;
                }
                else
                {
                    buf[n]='\0';
                    printf("Socket %d said : %s\n",sockfd,buf);
                    write(sockfd,buf,n); //Write back to client
                }
                nready--;
                if(nready<=0) break; //no more readable descriptors
            }
        }
    }
    return 0;
}
```
	
## 2.3. epoll
epoll就更简单了，epoll使用一组函数来完成任务。首先epoll在内核中维护一个事件表，用来记录用户关心的所有文件描述符，这个文件描述符由epoll_create创建。创建之后需要把描述符加入到事件表中，这里使用epoll_ctl函数。之所以使用事件表，是因为select和poll在加入新连接的时候都需要把更新一遍描述符集合，这样每次都要把一个集合整体重新往内核态中传一遍，而事件表就直接往里加就行了。Epoll_wait函数如果检测到事件，就讲所有就绪的事件从内核事件表中复制到第二个参数里，这样就不用跟select和poll一样挨个检索了。
```c++
#include <stdio.h>
#include <string.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <poll.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <fcntl.h>
#include <sys/epoll.h>
#include <sys/time.h>
#include <sys/resource.h>
    
#define MAXLINE  1024
#define OPEN_MAX  16 //一些系统会定义这些宏
#define SERV_PORT  10001
    
int main()
{
    int i , maxi ,listenfd , connfd , sockfd ,epfd, nfds;
    int n;
    char buf[MAXLINE];
    struct epoll_event ev, events[20];  
    socklen_t clilen;
    struct pollfd client[OPEN_MAX];
    
    struct sockaddr_in cliaddr , servaddr;
    listenfd = socket(AF_INET , SOCK_STREAM , 0);
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERV_PORT);
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    
    bind(listenfd , (struct sockaddr *) & servaddr, sizeof(servaddr));
    listen(listenfd,10);
    
    epfd = epoll_create(256);
    ev.data.fd=listenfd; 
    ev.events=EPOLLIN|EPOLLET;
    epoll_ctl(epfd,EPOLL_CTL_ADD,listenfd,&ev);
    
    for(;;)
    {
        nfds=epoll_wait(epfd,events,20,500); 
        for(i=0; i<nfds; i++)
        {
            if (listenfd == events[i].data.fd)
            {
                clilen = sizeof(cliaddr);
                connfd = accept(listenfd , (struct sockaddr *)&cliaddr, &clilen);
                if(connfd < 0)  
                {  
                    perror("connfd < 0");  
                    exit(1);  
                }
                ev.data.fd=connfd; 
                ev.events=EPOLLIN|EPOLLET;
                epoll_ctl(epfd,EPOLL_CTL_ADD,connfd,&ev);                
            }
            else if (events[i].events & EPOLLIN)
            {
                if ( (sockfd = events[i].data.fd) < 0)  
                    continue;  
                n = recv(sockfd,buf,MAXLINE,0);
                if (n <= 0)   
                {    
                    close(sockfd);  
                    events[i].data.fd = -1;  
                }
                else
                {
                    buf[n]='\0';
                    printf("Socket %d said : %s\n",sockfd,buf);
                    ev.data.fd=sockfd; 
                    ev.events=EPOLLOUT|EPOLLET;
                    epoll_ctl(epfd,EPOLL_CTL_MOD,connfd,&ev);
                }
            }
            else if( events[i].events&EPOLLOUT )
            {
                sockfd = events[i].data.fd;  
                send(sockfd, "Hello!", 7, 0);  
                    
                ev.data.fd=sockfd;  
                ev.events=EPOLLIN|EPOLLET;  
                epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev); 
            }
            else 
            {
                printf("This is not avaible!");
            }
        }
    }
    close(epfd);  
    return 0;
}
```