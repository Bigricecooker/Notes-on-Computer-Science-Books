## 地址族与数据序列


### IP地址与端口号

**网络地址**

IP时Internet Protocol（网络协议的简称），是为收发网络数据而分配给计算机的地址值。端口号并非赋予计算机的值，而是为了区分程序中创建的套接字而分配给套接字的序号

IP地址分类
- IPv4（Internet Protocol version 4）  4字节地址族
- IPv6 （Internet Protocol version 6）  16字节地址族

分类什么的直接看书

**端口号**

IP地址用于区分计算机，只要有IP地址就可以向目标主机传输数据，假如一台计算机只能运行一个套接字，那么没问题，计算机只需把接收到的数据发送给该套接字就可以。但事实却是，一台计算机往往存在多个套接字，为了区分数据是传递给哪台主机的哪个套接字，只有IP地址是不够的，还需要端口号，一个端口号对应一个套接字。因此IP地址+端口号，就可以给指定计算机的指定端口号发送数据了。

计算机中一般配有NIC（Network InterFace Card, 网络接口卡）数据传输设备。通过NIC向计算机内部传输数据时会用到IP。操作系统负责把传递到内部的数据适当分配给套接字，这时就要利用端口号。也就是说，通过NIC接收的数据内有端口号，操作系统正是参考次端口号把数据传输给相应的套接字。

端口号由16位构成，可分配的端口号范围为：0到65535。但0到1203是知名端口（Well-know PORT），一般分配给特定应用程序，所以应当分配次范围外的值。另外，TCP套接字和UDP套接字可以使用同样的端口号，并不会冲突。


### 表示IPv4地址的结构体

**结构体sockaddr_in**
```c
struct sockaddr_in{
  sa_family_t 		sin_family;    //地址族（Address Family）
  uint16_t    		sin_port;			 //16位TCP/UDP端口号
  struct in_addr  sin_addr;			 //32位IP地址
  char 						sin_zero[8]     //不使用
};

struct in_addr{
  in_addr_t  			s_addr;         //32位IPv4地址
};
```

此结构体将作为地址信息传给bind函数

**成员分析**

1.sin_family

每种协议族适用的地址族均不同，比如IPv4使用4字节地址族，IPv6使用16字节地址族。可以参考下表保存sin_family地址信息。

| 地址族（Address Family） | 含义                             |
| ------------------------ | -------------------------------- |
| AF_INET                  | IPv4网络协议中使用的地址族       |
| AF_INET6                 | IPv6网络协议中使用的地址族       |
| AF_LOCAL                 | 本地通信中采用的UNIX协议的地址族 |

AF_LOCAL只是为了说明具有多种地址族而添加。

注意.socket函数里的参数是协议族

2.sin_port

该成员保存16位端口号，重点在于，它以网络字节序保存。

3.sin_addr

该成员保存32位IP地址信息，且也以网络字节序保存。为理解好该成员，应同时观察结构体in_addr。但结构体in_addr声明为uint32_t，因此只需当作32位整数型即可。

4.sin_zero

无特殊含义。只是为了使结构体sockaddr_in的大小与sockaddr结构体保持一致而插入的成员。必须填充为0，否则无法得到想要的结果。


sockaddr_in结构体变量地址值将以如下方式传递给bind函数。稍后将给出关于bind函数的详细说明，希望各位重点关注参数传递和类型转换部分代码

```c
struct sockaddr_in serv_addr;
....
if(bind(serv_addr, (struct sockaddr*)&serv_addr,sizeof(serv_addr))==-1)
  error_handling("bind() error");
....
```

有一些细节知识直接去看书

### 网络字节序与地址变换

**字节序与网络字节序**

有关字节序和网络字节序,感觉知道网络字节序是大端序,且通信基本都是用网络字节序就可以了


**字节序转换**

- unsigned short htons(unsigned short);
- unsigned short ntohs(unsigned short);
- unsigned long htonl(unsigned long);
- unsigned long ntohl(unsigned long);

h host 主机

to 转换

n network 网络

s short 2字节16位整数

l long 4字节32位整数

例: htons 将short型数据从主机字节序转化为网络字节序

哪怕系统默认是网络字节序,最好还是也转换一下

**将字符串信息转换为网络字节序的整数型**

sockaddr_in中保存地址信息的成员为32位整数型，因此为了分配IP地址，需要将我们熟悉的字符串类型，转换为4字节整数型数据。

对于IP地址，我们熟悉的是点分十进制表示法，有一个函数可以帮我们把字符串形式的IP地址转换成32位整数型数据，此函数在转换类型的同时进行网络字节序转换。

```c
#include <arpa/inet.h>
in_addr_t inet_addr(const char* string);
//成功时返回32位大端序整数型值
//失败时返回INADDR_NONE
```

下一个函数是inet_aton，inet_aton函数与inet_addr函数在功能上完全相同，也将字符串形式IP地址信息带入sockaddr_in结构体中声明的in_addr结构体变量。而inet_aton函数则不需要此过程，因为它会在转换后，再把结果填入该结构体变量。

```c
#include <arpa/inet.h>
int inet_aton(const char* string,struct in_addr * addr);
//成功时返回1
//失败时返回0

string 含有需转换的IP地址信息的字符串地址值
addr 将保存转换结果的in_addr结构体变量的地址值
```


下面函数执行与inet_aton函数正好相反的函数，此函数可以把网络字节序整数型IP地址转换成为我们熟悉的字符串形式。

```c
#include <arpa/inet.h>

char* inet_ntoa(struct in_addr adr);
//成功时返回转换的字符串地址值，失败时返回-1
```

**网络地址初始化**

```c
struct sockaddr_in addr;
char* serv_ip = "211.217.168.13";     //声明IP地址字符串
char* serv_port = "9190";             //声明端口号字符串
memset(&addr, 0 ,sizeof(addr));       //结构体变量addr的所有成员初始化为0
addr.sin_family = AF_INET;            //指定地址族
addr.sin_addr.s_addr = inet_addr(serv_ip);  //基于字符串的IP地址初始化
addr.sin_port = htons(atoi(serv_port));     //基于字符串的端口号初始化

```


**INADDR_ANY**

```c
struct sockaddr_in addr;
char* serv_port = "9190";
memset(&addr, 0, sizeof(addr));
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = htonl(INADDR_ANY);
addr.sin_port = htons(atoi(serv_port));
```

可以将常数INADDR_ANY直接看作是服务端的IP地址,且它不是字符串


**向套接字分配网络地址(服务器端)**

```c
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr* myaddr, socklen_t addrlen);
//成功时返回0，失败时返回-1。
/*
	sockfd 要分配地址信息（IP地址和端口号）的套接字文件描述符
	myaddr 存有地址信息的结构体变量地址值
	addrlen 第二个结构体变量的长度
*/
```



下面给出服务器常见套接字初始化过程

```c
int serv_sock;
struct sockaddr_in serv_addr;
char* serv_port = "9190";

/*创建服务器端套接字（监听套接字）*/
serv_sock = socket(PF_INET, SOCK_STREAM, 0);

/*地址信息初始化*/
memset(&serv_addr, 0, sizeof(serv_addr));
serv_addr.sin_family = AF_INET;
serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
serv_addr.sin_port = htons(atoi(serv_port));

/*分配地址信息*/
bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
```



服务器端代码结构默认如上，当然还有未显示的异常处理代码。
