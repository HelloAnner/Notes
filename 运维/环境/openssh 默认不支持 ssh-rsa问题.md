

#### 临时解决 - 不安全
```shell
Host doc
    HostName 192.168.101.202
    User root
    Port 22
    ServerAliveInterval 120
    HostKeyAlgorithms=+ssh-rsa
    IdentityFile ~/.ssh/doc
```

```shell
ssh -v -oHostKeyAlgorithms=+ssh-rsa root@192.168.101.202
```

#### 原因



#### 正确做法




#### 参考

https://zhuanlan.zhihu.com/p/419745598
