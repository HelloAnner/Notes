
JVM 生成的 native code 存放的内存空间称之为 Code Cache；JIT 编译、JNI 等都会编译代码到 native code，其中 JIT 生成的 native code 占用了 Code Cache 的绝大部分空间。

### 概念

JVM 生成的 native code 存放的内存空间称之为 Code Cache；JIT 编译、JNI 等都会编译代码到 native code，其中 JIT 生成的 native code 占用了 Code Cache 的绝大部分空间。

如果 CodeCache 这部分内存空间耗尽的话，jvm 就无法继续生成新的 native code，就会导致性能大幅下降，**现象表现为各线程执行都阻塞在一些 CPU 密集的方法，例如计算 hash、运行正则表达式、实例化新对象等等**。

### 问题暴露

正常情况下 java 的默认 CodeCache 配置是能够满足需求的，此处不会产生性能瓶颈，但是经过调查发现，**在 arm64 的 cpu 平台上，jvm 默认指定的 CodeCache 值会过小**。

以 oracle 的 jdk1.8 为例，正常 x86_64 平台上默认的 CodeCache 大小为 250m 左右： 

而 arm64 平台上默认为 50m 左右（影响因素可能包括不限于 cpu 类型）：

这样就导致 arm64 服务器上运行报表系统更容易出现 CodeCache 耗尽导致性能下降的问题，同时由于 jit 的动态编译、清理机制，CodeCache 耗尽需要运行一段时间后才会出现，所以表现出来的现象为**正常运行好几天甚至个把月后出现整体性的访问性能下降，但是重启后恢复**。

截至目前我们已经发现至少 3 个客户存在过这样的现象，并且客户使用的都是 arm64 的服务器。

同时 arm64 平台的【代码密度】（即编译出的汇编指令大小）更小，因此编译出来的 native code 需要占用更大的空间（网上的资料说明 arm64 的汇编指令大小比 x86_64 大 30% 左右），进一步加剧了 CodeCache 不够用的问题。

### 问题定位 & 优化方案

当已用大小达到或即将 90% 的最大值时，一般认为可能导致性能下降，优化方案就是指定一个更大的 CodeCache 空间，就像 jvm 内存一样。

添加 jvm 参数：==**-XX:ReservedCodeCacheSize=<期望的大小>**==，一般来说指定到 250m 就够了（如：-XX:ReservedCodeCacheSize=250m）。 

### 参考文档
[https://www.jianshu.com/p/b064274536ed](https://www.jianshu.com/p/b064274536ed)
[https://cloud.tencent.com/developer/article/1879628](https://cloud.tencent.com/developer/article/1879628)