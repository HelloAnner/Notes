## 基础介绍

tcpdump命令是Linux中的截包命令工具，强大且易于使用。**tcpdump基于底层libpcap库开发，运行需要root权限**


tcpdump截取包默认显示数据包的头部。

基础格式：时间 数据包类型 源IP 端口/协议 > 目标IP 端口/协议 协议详细信息

```text
23:53:17.055214 IP 172.20.10.2.59694 > 112.65.66.248.https: Flags [.], ack 3411, win 1998, options [nop,nop,TS val 2090943386 ecr 1232825480], length 0

23:53:17.055309 IP 172.20.10.2.59694 > 112.65.66.248.https: Flags [.], ack 3411, win 2048, options [nop,nop,TS val 2090943386 ecr 1232825480], length 0

23:53:17.058229 IP 172.20.10.2.59690 > 17.57.145.135.5223: Flags [P.], seq 518:598, ack 3572, win 2048, options [nop,nop,TS val 3270328246 ecr 3167096378], length 80
```

```shell
tcpdump -c 10 -w /tmp/tcpdump_test.log
# -w 保存文件 tcpdump保存的文件的格式是几乎所有主流的抓包工具软件都可以读取。所以可以使用更易读的图形界面工具来查看记录文件

# -c 在收到指定的包的数目后，tcpdump就会停止
```


```text
> tcpdump -r tcpdump_test.log

reading from PCAP-NG file tcpdump_test.log

23:57:31.598352 IP bogon.60456 > 112.65.193.167.http: Flags [S], seq 3172671584, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 1088641165 ecr 0,sackOK,eol], length 0
```

-r选项读取文件

**-p** : 不让网络接口进入混杂模式。默认情况下使用 tcpdump 抓包时，会让网络接口进入混杂模式。一般计算机网卡都工作在非混杂模式下，此时网卡只接受来自网络端口的目的地址指向自己的数据。当网卡工作在混杂模式下时，网卡将来自接口的所有数据都捕获并交给相应的驱动程序。如果设备接入的交换机开启了混杂模式，使用 `-p` 选项可以有效地过滤噪声。

**-e** : 显示数据链路层信息。默认情况下 tcpdump 不会显示数据链路层信息，使用 `-e` 选项可以显示源和目的 MAC 地址，以及 VLAN tag 信息

**-l**:行缓冲模式（或使用 `-c` 选项来开启数据包缓冲模式）

```text
tcpdump -i eth0 -s0 -l port 80 | grep 'Server:'
```

---


```text
> tcpdump -D

1.en0 [Up, Running, Wireless, Associated]

2.awdl0 [Up, Running, Wireless, Associated]

3.llw0 [Up, Running, Wireless, Associated]


> tcpdump -i eth0
```

**打印出所有可工作的接口 -D**

---

-v，-vv可以显示更详细的抓包信息

-n后，tcpdump会直接显示IP地址，不会显示域名（与netstat命令相似）

## 条件过滤

#### 过滤：指定需要抓取的协议

tcpdump可以只抓某种协议的包，支持指定以下协议：ip,ip6,arp,tcp,udp,wlan等
```shell
tcpdump udp
tcpdump icmp
tcpdump tcp
tcpdump arp
```

#### 过滤：指定协议的端口号

使用port参数，用于指定端口号

```shell
tcpdump tcp port 80
```

#### 过滤：指定源与目标

src 表示源。 dst 表示目标。

```shell
tcpdump src port 8080

tcpdump dst port 80
```

#### 过滤：指定特定主机的消息包

```shell
tcpdump host 192.168.1.113
```

若使用了host参数使用了计算机名或域名。例tcpdump host shi-pc ，则无法再使用-n选项


#### 过滤：指定数据包大小

使用greater（大于）与less（小于）可以指定数据包大小的范围。

例：只抓取大于1000字节的数据包。

命令：_tcpdump greater 1000_

例：只抓取小于10字节的数据包。

命令：_tcpdump less 10_


#### Network 过滤器

Network 过滤器用来过滤某个网段的数据，使用的是 [CIDR](https://link.juejin.cn/?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FClassless_Inter-Domain_Routing "https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing") 模式。可以使用四元组（x.x.x.x）、三元组（x.x.x）、二元组（x.x）和一元组（x）

四元组就是指定某个主机，三元组表示子网掩码为 `255.255.255.0`，二元组表示子网掩码为 `255.255.0.0`，一元组表示子网掩码为 `255.0.0.0`

```text

抓取所有发往网段 `192.168.1.x` 或从网段 `192.168.1.x` 发出的流量
tcpdump net 192.168.1

抓取所有发往网段 `10.x.x.x` 或从网段 `10.x.x.x` 发出的流量：
tcpdump net 10

tcpdump src net 172.16.0.0/12


```


#### Proto 过滤器

```text
tcpdump -n icmp
```


## 查看数据包完整内容

tcpdump默认不显示数据包的详细内容

```text
21:27:06.995846 IP (tos 0x0, ttl 64, id 45646, offset 0, flags [DF], proto TCP (6), length 64) 192.168.1.106.56166 > 124.192.132.54.80: Flags [S], cksum 0xa730 (correct), seq 992042666, win 65535, options [mss 1460,nop,wscale 4,nop,nop,TS val 663433143 ecr 0,sackOK,eol], length 0 

21:27:07.030487 IP (tos 0x0, ttl 51, id 0, offset 0, flags [DF], proto TCP (6), length 44) 124.192.132.54.80 > 192.168.1.106.56166: Flags [S.], cksum 0xedc0 (correct), seq 2147006684, ack 992042667, win 14600, options [mss 1440], length 0 

21:27:07.030527 IP (tos 0x0, ttl 64, id 59119, offset 0, flags [DF], proto TCP (6), length 40) 192.168.1.106.56166 > 124.192.132.54.80: Flags [.], cksum 0x3e72 (correct), ack 2147006685, win 65535, length 0
```

上面的三条数据还是 tcp 协议的三次握手过程，第一条就是 `SYN` 报文

-   `[S]` : SYN（开始连接）
-   `[.]` : 没有 Flag
-   `[P]` : PSH（推送数据）
-   `[F]` : FIN （结束连接）
-   `[R]` : RST（重置连接

第二条数据的 `[S.]` 表示 `SYN-ACK`，就是 `SYN` 报文的应答报文


#### 使用-A参数能以ASCII码显示数据包。

例：只抓取1个数据包，并显示其内容。

命令：_tcpdump -c 1 -A_

```text
sh-3.2# tcpdump -c 1 -A

tcpdump: data link type PKTAP

tcpdump: verbose output suppressed, use -v[v]... for full protocol decode

listening on pktap, link-type PKTAP (Apple DLT_PKTAP), snapshot length 524288 bytes

00:08:15.175762 IP bogon.62083 > 112.65.193.167.http: Flags [S], seq 3157787744, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 2645711400 ecr 0,sackOK,eol], length 0

E..@..@.@.R...

.pA.....P.8.`.......................

..Z(........

1 packet captured

21 packets received by filter

0 packets dropped by kernel
```

#### 使用-X参数能16进制数与ASCII码共同显示数据包。

例：只抓取1个数据包，并显示其内容。

命令：_tcpdump -c 1 -X_


```text
sh-3.2# tcpdump -c 1 -X

tcpdump: data link type PKTAP

tcpdump: verbose output suppressed, use -v[v]... for full protocol decode

listening on pktap, link-type PKTAP (Apple DLT_PKTAP), snapshot length 524288 bytes

00:09:44.039466 IP bogon.62297 > 112.65.193.167.http: Flags [S], seq 3498369155, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 3939669770 ecr 0,sackOK,eol], length 0

0x0000:  4500 0040 0000 4000 4006 52b9 ac14 0a02  E..@..@.@.R.....

0x0010:  7041 c1a7 f359 0050 d084 e083 0000 0000  pA...Y.P........

0x0020:  b002 ffff 286d 0000 0204 05b4 0103 0306  ....(m..........

0x0030:  0101 080a ead2 970a 0000 0000 0402 0000  ................

1 packet captured

47 packets received by filter

0 packets dropped by kernel
```

## 参数列表


```text
-a  将网络地址和广播地址转变成名字；

-c  在收到指定的包的数目后，tcpdump就会停止；

-d  将匹配信息包的代码以人们能够理解的汇编格式给出；以可阅读的格式输出。

-dd  将匹配信息包的代码以c语言程序段的格式给出；

-ddd  将匹配信息包的代码以十进制的形式给出；

-e  在输出行打印出数据链路层的头部信息；

-f  将外部的Internet地址以数字的形式打印出来；

-l  使标准输出变为缓冲行形式；

-n  直接显示IP地址，不现实名称；

-nn  端口名称显示为数字形式，不现实名称；

-t  在输出的每一行不打印时间戳；

-v  输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息；

-vv  输出详细的报文信息；

-F  从指定的文件中读取表达式,忽略其它的表达式；

-i  指定监听的网络接口；

-r  从指定的文件中读取包(这些包一般通过-w选项产生)；

-w  直接将包写入文件中，并不分析和打印出来；

-T  将监听到的包直接解释为指定的类型的报文，常见的类型有rpc （远程过程调用）和snmp（简单 网络管理协议；）

-s    抓取数据包时默认抓取长度为68字节。加上 -s 0 后可以抓到完整的数据包
```

## 实战记录

```text

tcpdump -i eth0 port 80 -s 1024 -l -A


# 从 HTTP 请求头中提取 HTTP 用户代理

tcpdump -nn -A -s1500 -l | grep "User-Agent:"
tcpdump -nn -A -s1500 -l | egrep -i 'User-Agent:|Host:'

# 抓取 HTTP GET 流量
tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'

# 抓取 HTTP POST 请求流量
tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'

# 该方法不能保证抓取到 HTTP POST 有效数据流量，因为一个 POST 请求会被分割为多个 TCP 数据包


# 提取 HTTP 请求的主机名和路径
tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"

# 从 HTTP POST 请求中提取密码和主机名
tcpdump -s 0 -A -n -l | egrep -i "POST /|pwd=|passwd=|password=|Host:"

# 提取 Cookies
tcpdump -nn -A -s0 -l | egrep -i 'Set-Cookie|Host:|Cookie:'

# 查看网络上的所有 ICMP 数据包
tcpdump -n icmp

# 提取电子邮件的收件人
tcpdump -nn -l port 25 | grep -i 'MAIL FROM\|RCPT TO'

# 发包最多的 IP

tcpdump -nnn -t -c 200 | cut -f 1,2,3,4 -d '.' | sort | uniq -c | sort -nr | head -n 20

# 抓取 80 端口的 HTTP 有效数据包，排除 TCP 连接建立过程的数据包（SYN / FIN / ACK
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'



```

上述两个表达式中的十六进制将会与 GET 和 POST 请求的 `ASCII` 字符串匹配。例如，`tcp[((tcp[12:1] & 0xf0) >> 2):4]` 首先会[确定我们感兴趣的字节的位置](https://link.juejin.cn?target=https%3A%2F%2Fsecurity.stackexchange.com%2Fquestions%2F121011%2Fwireshark-tcp-filter-tcptcp121-0xf0-24 "https://security.stackexchange.com/questions/121011/wireshark-tcp-filter-tcptcp121-0xf0-24")（在 TCP header 之后），然后选择我们希望匹配的 4 个字节


通常 `Wireshark`（或 tshark）比 tcpdump 更容易分析应用层协议。一般的做法是在远程服务器上先使用 `tcpdump` 抓取数据并写入文件，然后再将文件拷贝到本地工作站上用 `Wireshark` 分析



## 参考

https://juejin.cn/post/6844904084168769549

https://www.cnblogs.com/shijiaqi1066/p/3898248.html

https://blog.51cto.com/masters/1870141