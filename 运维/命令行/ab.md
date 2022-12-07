
```shell
ab -c 300 -k -n 1000 -T "application/json;charset=UTF-8" -H 
"authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwidGVuYW50SWQiOiJkZWZhdWx0IiwiaXNzIjoiZmFucnVhbiIsImRlc2NyaXB0aW9uIjoiMSgxKSIsImV4cCI6MTY3MDI1MTA5NCwiaWF0IjoxNjcwMjQ3NDk0LCJqdGkiOiJ4RVc5UFAzOWg5UitBYSsxZXkwdHlBc3dadTMwMjhwZTB3SlR0eGphN1JQQzA5dHoifQ.KNeBl1ZoCsGU09eOSip7KAfFno_3oHJ6k49QyxGgEu4" -C "JSESSIONID=6A90BA05BFBA9B05EE7606193B75292C; confSessionId=69b4c71ce7da45ca; tenantId=default; fine_remember_login=-1; fine_login_users=b5f0c2ee-640f-4039-a4d4-918b55354898,dec762a9-a053-4dc2-bfbc-6ff1dff7abb8; fine_auth_token=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwidGVuYW50SWQiOiJkZWZhdWx0IiwiaXNzIjoiZmFucnVhbiIsImRlc2NyaXB0aW9uIjoiMSgxKSIsImV4cCI6MTY3MDI1MTA5NCwiaWF0IjoxNjcwMjQ3NDk0LCJqdGkiOiJ4RVc5UFAzOWg5UitBYSsxZXkwdHlBc3dadTMwMjhwZTB3SlR0eGphN1JQQzA5dHoifQ.KNeBl1ZoCsGU09eOSip7KAfFno_3oHJ6k49QyxGgEu4" "http://localhost:8090/webroot/decision/v10/departments/posts/tree"
```

> socket: Too many open files (24)

在使用ab做压力测试的时候发现当并发设置为1000以上的时候就出现
出现这个问题主要是因为文件打开数的限制，默认情况下是1024,可以使用ulimit -n查看

解决方案:

```shell
ulimit -SHn 65536
echo "* soft nofile 65536" >>/etc/security/limits.conf
echo "* hard nofile 65536" >>/etc/security/limits.conf
```

第一行是暂时性修改文件打开数
第二行和第三行是修改配置文件调整文件打开数，需要重启才能生效


---

> apr_socket_recv: Connection reset by peer (54)

apr_socket_recv这个是操作系统内核的一个参数，在高并发的情况下，内核会认为系统受到了SYN flood攻击，会发送cookies（possible SYN flooding on port 80. Sending cookies）

这样会减慢影响请求的速度，所以在应用服务武器上设置下这个参数为0禁用系统保护就可以进行大并发测试了：


vim /etc/sysctl.conf

> net.ipv4.tcp_syncookies = 0


```text
#此参数是为了防止洪水攻击的，但对于大并发系统，要禁用此设置
net.ipv4.tcp_syncookies = 0  


#参数决定了SYN_RECV状态队列的数量，一般默认值为512或者1024，即超过这个数量，系统将不再接受新的TCP连接请求，一定程度上可以防止系统资源耗尽。可根据情况增加该值以接受更多的连接请求
net.ipv4.tcp_max_syn_backlog

#参数决定是否加速TIME_WAIT的sockets的回收，默认为0
net.ipv4.tcp_tw_recycle

#参数决定是否可将TIME_WAIT状态的sockets用于新的TCP连接，默认为0
net.ipv4.tcp_tw_reuse

#参数决定TIME_WAIT状态的sockets总数量，可根据连接数和系统资源需要进行设置
net.ipv4.tcp_max_tw_buckets
```


对于mac：

这个报错一般是由于使用的MacOSX默认自带的ab限制了并发数导致的
解决办法：下载最新的apache并重新编译，备份原来的ab并将新编译的ab替换到原来的路径






