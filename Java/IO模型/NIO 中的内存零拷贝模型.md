
首先说明一下内存中的拷贝优化和外部的[[BIO NIO AIO]] 无关，仅仅是准备数据到 application buffer 的过程

下图中左侧才是BIO等的区别；

---


首先澄清，零拷贝与内存直接映射并不是 Java 中独有的概念，并且这两个技术并不是等价的。
零拷贝是指避免在用户态 (User-space) 与内核态 (Kernel-space) 之间来回拷贝数据的技术。

## 传统 IO

传统 IO 读取数据并通过网络发送的流程，如下图 
![](https://ask.qcloudimg.com/http-save/yehe-5086501/2fgxt74486.jpeg?imageView2/2/w/1620)

  
1.  read() 调用导致上下文从用户态切换到内核态。内核通过 sys_read()（或等价的方法）从文件读取数据。DMA 引擎执行第一次拷贝：**从文件读取数据并存储到内核空间的缓冲区**。

2.  请求的数据从内核的读缓冲区拷贝到用户缓冲区，然后 read() 方法返回。read() 方法返回导致上下文从内核态切换到用户态。现在待读取的数据已经存储在用户空间内的缓冲区。至此，完成了一次 IO 的读取过程。

3.  send() 调用导致上下文从用户态切换到内核态。第三次拷贝数据从用户空间重新拷贝到内核空间缓冲区。但是，这一次，数据被写入一个不同的缓冲区，一个与目标套接字相关联的缓冲区。

4.  send() 系统调用返回导致第四次上下文切换。当 DMA 引擎将数据从内核缓冲区传输到协议引擎缓冲区时，第四次拷贝是独立且异步的。


内存缓冲数据 (上图中的 read buffer 和 socket buffer)，主要是为了提高性能，内核可以预读部分数据，当所需数据小于内存缓冲区大小时，将极大的提高性能。
  

![](https://ask.qcloudimg.com/http-save/yehe-5086501/hx89ujzu6x.jpeg?imageView2/2/w/1620)

  

磁盘到内核空间属于 DMA 拷贝，**用户空间与内核空间之间的数据传输并没有类似 DMA 这种可以不需要 CPU 参与的传输方式**，因此用户空间与内核空间之间的数据传输是需要 CPU 全程参与
  

DMA 拷贝即直接内存存取，原理是外部设备不通过 CPU 而直接与系统内存交换数据

所以也就有了使用零拷贝技术，避免不必要的 CPU 数据拷贝过程。

  

## NIO 的零拷贝

NIO 的零拷贝由 transferTo 方法实现。transferTo 方法将数据从 FileChannel 对象传送到可写的字节通道（如 Socket Channel 等）。在 transferTo 方法内部实现中，由 native 方法 transferTo0 来实现，它依赖底层操作系统的支持。在 UNIX 和 Linux 系统中，调用这个方法会引起 sendfile() 系统调用，==实现了数据直接从内核的读缓冲区传输到套接字缓冲区，避免了用户态 (User-space) 与内核态 (Kernel-space) 之间的数据拷贝==。

  

![](https://ask.qcloudimg.com/http-save/yehe-5086501/7lmjkz6ahu.jpeg)

  

使用 NIO 零拷贝，流程简化为两步：

1.  transferTo 方法调用触发 DMA 引擎将文件上下文信息拷贝到内核读缓冲区，接着内核将数据从内核缓冲区拷贝到与套接字相关联的缓冲区。

2.  **DMA 引擎将数据从内核套接字缓冲区传输到协议引擎（第三次数据拷贝）**。

  

内核态与用户态切换如下图：

  

![](https://ask.qcloudimg.com/http-save/yehe-5086501/1vdvwcllpb.jpeg)

  

相比传统 IO，使用 NIO 零拷贝后改进的地方：

  

1.  我们已经将上下文切换次数从 4 次减少到了 2 次；

2.  将数据拷贝次数从 4 次减少到了 3 次（其中只有 1 次涉及了 CPU，另外 2 次是 DMA 直接存取）。

  

如果底层 NIC（网络接口卡）支持 gather 操作，可以进一步减少内核中的数据拷贝。在 Linux 2.4 以及更高版本的内核中，socket 缓冲区描述符已被修改用来适应这个需求。这种方式不但减少上下文切换，同时消除了需要 CPU 参与的重复的数据拷贝。

  

![](https://ask.qcloudimg.com/http-save/yehe-5086501/8wb81khozb.jpeg)

  

用户这边的使用方式不变，依旧通过 transferTo 方法，但是方法的内部实现发生了变化：

  

1.  transferTo 方法调用触发 DMA 引擎将文件上下文信息拷贝到内核缓冲区。

2.  数据不会被拷贝到套接字缓冲区，只有数据的描述符（包括数据位置和长度）被拷贝到套接字缓冲区。DMA 引擎直接将数据从内核缓冲区拷贝到协议引擎，这样**减少了最后一次需要消耗 CPU 的拷贝操作**。

  

NIO 零拷贝适用于以下场景：

  

1.  文件较大，读写较慢，追求速度

2.  JVM 内存不足，不能加载太[大数据](https://cloud.tencent.com/solution/bigdata?from=10680)

3.  内存带宽不够，即存在其他程序或线程存在大量的 IO 操作，导致带宽本来就小

  

### NIO 的零拷贝代码示例

```java
public static void fileCopyWithFileChannel(File fromFile, File toFile) {
    try (
        FileChannel fileChannelInput = new FileInputStream(fromFile).getChannel();

        FileChannel fileChannelOutput = new FileOutputStream(toFile).getChannel()) {


        fileChannelInput.transferTo(0, fileChannelInput.size(), fileChannelOutput);
    } catch (IOException e) {
        e.printStackTrace();
    }
}

static final int BUFFER_SIZE = 1024;

public static void bufferedCopy(File fromFile,File toFile) throws IOException {
    try(BufferedInputStream bis = new BufferedInputStream(new FileInputStream(fromFile));
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(toFile))){
        byte[] buf = new byte[BUFFER_SIZE];
        while ((bis.read(buf)) != -1) {
            bos.write(buf);
        }
    }
}
```


在不需要进行数据文件操作时，可以使用 NIO 的零拷贝。但如果既需要 IO 速度，又需要进行数据操作，则需要使用 NIO 的直接内存映射。

  

Linux 提供的 mmap 系统调用, 它可以将一段用户空间内存映射到内核空间, 当映射成功后, 用户对这段内存区域的修改可以直接反映到内核空间；同样地， 内核空间对这段区域的修改也直接反映用户空间。正因为有这样的映射关系, 就不需要在用户态 (User-space) 与内核态(Kernel-space) 之间拷贝数据， 提高了数据传输的效率，这就是内存直接映射技术。

  

## NIO 的直接内存映射

  

JDK1.4 加入了 NIO 机制和直接内存，目的是防止 Java 堆和 Native 堆之间数据复制带来的性能损耗，此后 NIO 可以使用 Native 的方式直接在 Native 堆分配内存。

  

背景：堆内数据在 flush 到远程时，会先复制到 Native 堆，然后再发送；直接移到堆外就更快了。

在 JDK8，Native Memory 包括元空间和 Native 堆。更多有关 JVM 的知识，点击查看 [JVM 内存模型和垃圾回收机制](https://mp.weixin.qq.com/s?__biz=MzUyNzgyNzAwNg==&mid=2247483849&idx=1&sn=9731f89b7086c0138bcb0586bf3eb5bf&scene=21#wechat_redirect)

  

![](https://ask.qcloudimg.com/http-save/yehe-5086501/8wofb0x7lk.jpeg)

  

### 直接内存的创建

  

在 ByteBuffer 有两个子类，HeapByteBuffer 和 DirectByteBuffer。前者是存在于 JVM 堆中的，后者是存在于 Native 堆中的。

  

![](https://ask.qcloudimg.com/http-save/yehe-5086501/teei1hs15p.png)

  

申请堆内存

```java
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    return new HeapByteBuffer(capacity, capacity);
}
```

  

申请直接内存

 ```java
  public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}
```

  

### 使用直接内存的原因

  

1.  对垃圾回收停顿的改善。因为 full gc 时，垃圾收集器会对所有分配的堆内内存进行扫描，垃圾收集对 Java 应用造成的影响，跟堆的大小是成正比的。过大的堆会影响 Java 应用的性能。如果使用堆外内存的话，堆外内存是直接受操作系统管理。这样做的结果就是能保持一个较小的 JVM 堆内存，以减少垃圾收集对应用的影响。（full gc 时会触发堆外空闲内存的回收。）

2.  减少了数据从 JVM 拷贝到 native 堆的次数，在某些场景下可以提升程序 I/O 的性能。

3.  可以突破 JVM 内存限制，操作更多的物理内存。

  

当直接内存不足时会触发 full gc，排查 full gc 的时候，一定要考虑。

  

有关 JVM 和 GC 的相关知识，请点击查看 [JVM 内存模型和垃圾回收机制](https://mp.weixin.qq.com/s?__biz=MzUyNzgyNzAwNg==&mid=2247483849&idx=1&sn=9731f89b7086c0138bcb0586bf3eb5bf&scene=21#wechat_redirect)

  

### 使用直接内存的问题

  

1.  堆外内存难以控制，如果内存泄漏，那么很难排查（VisualVM 可以通过安装插件来监控堆外内存）。

2.  堆外内存只能通过序列化和反序列化来存储，保存对象速度比堆内存慢，不适合存储很复杂的对象。一般简单的对象或者扁平化的比较适合。

3.  直接内存的访问速度（读写方面）会快于堆内存。在申请内存空间时，堆内存速度高于直接内存。

  

直接内存适合申请次数少，访问频繁的场合。如果内存空间需要频繁申请，则不适合直接内存。

  

### NIO 的直接内存映射

  

NIO 中一个重要的类：MappedByteBuffer——java nio 引入的文件内存映射方案，读写性能极高。MappedByteBuffer 将文件直接映射到内存。可以映射整个文件，如果文件比较大的话可以考虑分段进行映射，只要指定文件的感兴趣部分就可以。

  

由于 MappedByteBuffer 申请的是直接内存，因此不受 Minor GC 控制，只能在发生 Full GC 时才能被回收，因此 Java 提供了 DirectByteBuffer 类来改善这一情况。它是 MappedByteBuffer 类的子类，同时它实现了 DirectBuffer 接口，维护一个 Cleaner 对象来完成内存回收。因此它既可以通过 Full GC 来回收内存，也可以调用 clean() 方法来进行回收

  

### NIO 的直接内存映射的函数调用

  

FileChannel 提供了 map 方法来把文件映射为内存对象：

  

MappedByteBuffer map(int mode,long position,long size);

  

可以把文件的从 position 开始的 size 大小的区域映射为内存对象，mode 指出了 可访问该内存映像文件的方式

  

-   READ_ONLY,（只读）： 试图修改得到的缓冲区将导致抛出 ReadOnlyBufferException.(MapMode.READ_ONLY)

-   READ_WRITE（读 / 写）： 对得到的缓冲区的更改最终将传播到文件；该更改对映射到同一文件的其他程序不一定是可见的。 (MapMode.READ_WRITE)

-   PRIVATE（专用）： 对得到的缓冲区的更改不会传播到文件，并且该更改对映射到同一文件的其他程序也不是可见的；相反，会创建缓冲区已修改部分的专用副本。 (MapMode.PRIVATE)

  

使用参数 - XX:MaxDirectMemorySize=10M，可以指定 DirectByteBuffer 的大小最多是 10M。

  

### 直接内存映射代码示例

```java

static final int BUFFER_SIZE = 1024;


public static void fileReadWithMmap(File file) {

    long begin = System.currentTimeMillis();
byte[] b = new byte[BUFFER_SIZE];
int len = (int) file.length();
MappedByteBuffer buff;
try (FileChannel channel = new FileInputStream(file).getChannel()) {

    buff = channel.map(FileChannel.MapMode.READ_ONLY, 0, channel.size());
    for (int offset = 0; offset < len; offset += BUFFER_SIZE) {
        if (len - offset > BUFFER_SIZE) {
            buff.get(b);
        } else {
            buff.get(new byte[len - offset]);
        }
    }
} catch (IOException e) {
    e.printStackTrace();
}
long end = System.currentTimeMillis();
System.out.println("time is:" + (end - begin));
}


public static void fileReadWithByteBuffer(File file) {

    long begin = System.currentTimeMillis();
try(FileChannel channel = new FileInputStream(file).getChannel();) {

    ByteBuffer buff = ByteBuffer.allocate(BUFFER_SIZE);
    while (channel.read(buff) != -1) {
        buff.flip();
        buff.clear();
    }
} catch (IOException e) {
    e.printStackTrace();
}
long end = System.currentTimeMillis();
System.out.println("time is:" + (end - begin));
}
```