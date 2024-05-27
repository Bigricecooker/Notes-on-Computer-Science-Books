## 套接字通信

### 服务端与客户端用到的函数

**服务端函数**

1.socket: 创建流式socket
```c
#include <sys/socket.h>
int socket(int domin, int type, int protocol); 
//成功时返回文件描述符，失败时返回-1
```

2.bind: 指定用于通信的IP地址和端口
```c
#include <sys/socket.h>
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
//成功时返回0，失败时返回-1
```

3.listen: 将socket设置为监听模式
```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
//成功时返回0，失败时返回-1
```

4.accept: 接收客户端的连接(在接收到客户端请求前会造成阻塞)
```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
//成功时返回文件描述符，失败时返回-1
```

5.close: 关闭socket连接,释放资源
```c
#include <unistd.h>
int close(int fd);
//成功时返回0，失败时返回-1
->fd 需要关闭的文件或者套接字的文件描述符
```

**客户端函数**

相较于服务端,没有了bind、listen和accept,多出了connect

connect: 向服务端发起连接请求
```c
#include <sys/socket.h>
int connect(int sockfd, struct sockaddr *serv_ddr, socklen_t addrlen);
//成功时返回0，失败时返回-1
```

### 基于Linux的文件操作
对于Linux来说,socket也被认为是文件的一种,因此在Linux的socket通信中可以使用与文件相关的I/O函数,但在Window上文件与socket是不同的东西.

文件和套接字一般经过创建过程才会被分配文件描述符,下面是标准输入输出和标准错误的文件描述符
| 文件描述符 | 对象                     |
| :--------- | ------------------------ |
| 0          | 标准输入 Standard Input  |
| 1          | 标准输出 Standard Output |
| 2          | 标准错误 Standard Error  |

文件描述符是按最小的为未分配的值来为文件或套接字分配的,而上表中的文件描述符也可以关闭,所以文件或套接字的文件描述符有可能为0-2

对于与文件相关的I/O函数直接去看书









