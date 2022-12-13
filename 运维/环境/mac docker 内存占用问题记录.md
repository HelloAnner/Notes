Docker 用到了 Linux 的两项特性：namespaces 和 cgroups 来提供隔离与资源限制，因此无论如何在 macOS 上我们都必须通过一个虚拟机来使用 Docker

在 2021 年 4 月时，Docker for Mac（Docker Desktop）[发布了](https://link.juejin.cn?target=https%3A%2F%2Fwww.docker.com%2Fblog%2Freleased-docker-desktop-for-mac-apple-silicon%2F "https://www.docker.com/blog/released-docker-desktop-for-mac-apple-silicon/") 对 Apple Silicon 的实验性支持，它会使用 QEMU 运行一个 ARM 架构的 Linux 虚拟机，默认运行 ARM 架构的镜像，但也支持运行 x86 的镜像


[QEMU](https://link.juejin.cn?target=https%3A%2F%2Fwww.qemu.org%2Fdocs%2Fmaster%2Fabout%2Findex.html "https://www.qemu.org/docs/master/about/index.html") 是一个开源的虚拟机（Virtualizer）和仿真器（Emulator），所谓仿真器是说 QEMU 可以在没有来自硬件或操作系统的虚拟化支持的情况下，去模拟运行一台计算机，包括模拟与宿主机不同的 CPU 架构，例如在 Apple Silicon 上模拟 x86 架构的计算机。而在有硬件虚拟化支持的情况下，QEMU 也可以使用宿主机的 CPU 来直接运行，减少模拟运行的性能开销，例如使用 macOS 提供的 `Hypervisor.Framework`。

  
Docker for Mac 其实就是分别用到了 QEMU 的这两种能力来在 ARM 虚拟机上运行 x86 镜像，和在 Mac 上运行 ARM 虚拟机。


