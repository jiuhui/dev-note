# NIO

## Java NIO 由以下几个核心部分组成

java NIO中有三个核心的概念：Channel，Buffer 和 Selector 构成了核心的API。其它组件，如Pipe和FileLock，只不过是与三个核心组件共同使用的工具类。Buffer本身就是一块内存，在底层实现上是个数组，数据的读写都是通过buffer来实现的。基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。所有数据的读写都是通过Buffer来经行的，永远不会出现直接从Channel读数据的情况或者写数据的情况。与Stream不同的是Channel是双向的，一个Stream只可能是InputStream或者OutoutStream,Channel打开后则可以进行读取、写入或者读写。由于channel是双向的，因此他们更好的反应出底层操作系统的真实情况，在Linux系统中，底层操作系统就是双向的。

Java NIO中Channel的主要实现：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。

以下是Java NIO里关键的Buffer实现：

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

![](C:\work\src\dev-note\img\overview-selectors.png)

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。



## Channel

Java NIO的通道类似流，但又有些不同：

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

从通道读取数据到缓冲区，从缓冲区写入数据到通道。

Channel的实现：

这些是Java NIO中最重要的通道的实现：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

FileChannel 从文件中读写数据。

DatagramChannel 能通过UDP读写网络中的数据。

SocketChannel 能通过TCP读写网络中的数据。

ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

下面是FIleChannel的使用：

```java
public static void main(String[] args) throws Exception {
        FileInputStream fileInputStream = new FileInputStream("NioTest2.txt");
        FileChannel channel = fileInputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(48);
        int read = channel.read(byteBuffer);

        while (-1 != read){
            System.out.println("read:"+read);
            byteBuffer.flip();
            while (byteBuffer.hasRemaining()){
                System.out.println((char) byteBuffer.get());
            }

            byteBuffer.clear();
            read = channel.read(byteBuffer);
        }

        fileInputStream.close();

    }
```

注意 buf.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据。

## Buffer

Java NIO中的Buffer用于和NIO通道进行交互。数据是从通道读入缓冲区，从缓冲区写入到通道中的。缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

### Buffer的基本用法:

使用Buffer读写数据一般遵循以下四个步骤：

1. 写入数据到Buffer
2. 调用`flip()`方法
3. 从Buffer中读取数据
4. 调用`clear()`方法或者`compact()`方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。下面是读取一个文件写入到另一个文件使用buffer的例子。

```java
    public static void main(String[] args) throws Exception{
        FileInputStream fileInputStream = new FileInputStream("input.txt");
        FileOutputStream fileOutputStream = new FileOutputStream("output.txt");

        FileChannel inputChannel = fileInputStream.getChannel();
        FileChannel outputChannel = fileOutputStream.getChannel();

        ByteBuffer buffer = ByteBuffer.allocate(10);

        while (true){
            buffer.clear();
            int read = inputChannel.read(buffer);
            if (-1 == read){
                break;
            }
            buffer.flip();
            outputChannel.write(buffer);
        }
        fileInputStream.close();
        fileOutputStream.close();

    }
```

### Buffer的capacity,position和limit

为了理解Buffer的工作原理，需要熟悉它的三个属性：

- capacity
- position
- limit

position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。

#### capacity

作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

#### position

当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

#### limit

在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

### Buffer的类型

Java NIO 有以下Buffer类型

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

### 向Buffer中写数据

写数据到Buffer有两种方式：

- 从Channel写到Buffer。 `int` `bytesRead = inChannel.read(buf); ``//read into buffer.`
- 通过Buffer的put()方法写到Buffer里。`buf.put(``127``);`

### flip()方法

flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。

换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。

- ### 从Buffer中读取数据

  从Buffer中读取数据有两种方式：

  1. 从Buffer读取数据到Channel。 `int` `bytesWritten = inChannel.write(buf);`
  2. 使用get()方法从Buffer中读取数据。`byte` `aByte = buf.get();`

### rewind()方法

Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

### clear()与compact()方法

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

### mark()与reset()方法

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。这两个方法是配合使用的。

### slice()方法

Slice Buffer和 buffer是共享相同的底层数组

```java
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(10);

        for (int i = 0; i < buffer.capacity() ; i++) {
            buffer.put((byte)i);
        }

        // 从buffer中取出一部分称为另一个buffer  slice buffer
        buffer.position(2);
        buffer.limit(6);
        ByteBuffer slice = buffer.slice();
        // 然后修改数据
        for (int i = 0; i < slice.capacity() ; i++) {
            byte b = slice.get(i);
            b *= 2;
            slice.put(b);
        }
        // 最后从buffer中读取数据输出
        buffer.position(0);
        buffer.limit(buffer.capacity());
        while (buffer.hasRemaining()){
            System.out.println(buffer.get());
        }
    }
```

ByteBuffer是一个抽象类，它的实现有：DirectByteBuffer,DirectByteBufferR,HeapByteBuffer,HeapByteBufferR。默认实现是HeapByteBuffer。还有一个只读的实现HeapByteBufferR,就是只能读数据不能写数据。通过buffer.asReadOnlyBuffer()这个方法把一个普通的buffer变成一个只读的buffer。

### DirectBbuffer以及堆外内存和零拷贝

#### 1、关于Linux的内核态和用户态

- 内核态：控制计算机的硬件资源，并提供上层应用程序运行的环境。比如socket I/0操作或者文件的读写操作等
- 用户态：上层应用程序的活动空间，应用程序的执行必须依托于内核提供的资源。
- 系统调用：为了使上层应用能够访问到这些资源，内核为上层应用提供访问的接口。

三者的关系是：用户态运行的上层应用程序 通过系统调用 访问内核态的资源。

当我们通过JNI调用的native方法实际上就是从用户态切换到了内核态的一种方式。并且通过该系统调用使用操作系统所提供的功能。

Q：为什么需要用户进程(位于用户态中)要通过系统调用(Java中即使JNI)来调用内核态中的资源，或者说调用操作系统的服务了？
A：intel cpu提供Ring0-Ring3四种级别的运行模式，Ring0级别最高，Ring3最低。Linux使用了Ring3级别运行用户态，Ring0作为内核态。Ring3状态不能访问Ring0的地址空间，包括代码和数据。因此用户态是没有权限去操作内核态的资源的，它只能通过系统调用外完成用户态到内核态的切换，然后在完成相关操作后再有内核态切换回用户态。

#### 2、DirectByteBuffer

DirectByteBuffer是Java用于实现堆外内存的一个重要类，我们可以通过该类实现堆外内存的创建、使用和销毁。

DirectByteBuffer该类本身还是位于Java内存模型的堆中。堆内内存是JVM可以直接管控、操纵。
而DirectByteBuffer中的unsafe.allocateMemory(size);是个一个native方法，这个方法分配的是堆外内存，通过C的malloc来进行分配的。分配的内存是系统本地的内存，并不在Java的内存中，也不属于JVM管控范围，所以在DirectByteBuffer一定会存在某种方式来操纵堆外内存。
在DirectByteBuffer的父类Buffer中有个address属性：

```java
	// Used only by direct buffers
    // NOTE: hoisted here for speed in JNI GetDirectBufferAddress
    long address;
```

address只会被直接缓存给使用到。之所以将address属性升级放在Buffer中，是为了在JNI调用GetDirectBufferAddress时提升它调用的速率。
address表示分配的堆外内存的地址。

unsafe.allocateMemory(size);分配完堆外内存后就会返回分配的堆外内存基地址，并将这个地址赋值给了address属性。这样我们后面通过JNI对这个堆外内存操作时都是通过这个address来实现的了。

在前面我们说过，在linux中内核态的权限是最高的，那么在内核态的场景下，操作系统是可以访问任何一个内存区域的，所以操作系统是可以访问到Java堆的这个内存区域的。
Q：那为什么操作系统不直接访问Java堆内的内存区域了？
A：这是因为JNI方法访问的内存区域是一个已经确定了的内存区域地质，那么该内存地址指向的是Java堆内内存的话，那么如果在操作系统正在访问这个内存地址的时候，Java在这个时候进行了GC操作，而GC操作会涉及到数据的移动操作[GC经常会进行先标志在压缩的操作。即，将可回收的空间做标志，然后清空标志位置的内存，然后会进行一个压缩，压缩就会涉及到对象的移动，移动的目的是为了腾出一块更加完整、连续的内存空间，以容纳更大的新对象]，数据的移动会使JNI调用的数据错乱。所以JNI调用的内存是不能进行GC操作的。

Q：如上面所说，JNI调用的内存是不能进行GC操作的，那该如何解决了？
A：①堆内内存与堆外内存之间数据拷贝的方式(并且在将堆内内存拷贝到堆外内存的过程JVM会保证不会进行GC操作)：比如我们要完成一个从文件中读数据到堆内内存的操作，即FileChannelImpl.read(HeapByteBuffer)。这里实际上File I/O会将数据读到堆外内存中，然后堆外内存再讲数据拷贝到堆内内存，这样我们就读到了文件中的内存。而写操作则反之，我们会将堆内内存的数据线写到对堆外内存中，然后操作系统会将堆外内存的数据写入到文件中。

② 直接使用堆外内存，如DirectByteBuffer：这种方式是直接在堆外分配一个内存(即，native memory)来存储数据，程序通过JNI直接将数据读/写到堆外内存中。因为数据直接写入到了堆外内存中，所以这种方式就不会再在JVM管控的堆内再分配内存来存储数据了，也就不存在堆内内存和堆外内存数据拷贝的操作了。这样在进行I/O操作时，只需要将这个堆外内存地址传给JNI的I/O的函数就好了。	

### MappedByteBuffer介绍

MappedByteBuffer 是Java NIO中引入的一种硬盘物理文件和内存映射方式，当物理文件较大时，采用MappedByteBuffer，读写性能较高，其内部的核心实现是DirectByteBuffer(JVM 堆外直接物理内存)。

FileChannel提供了map方法把磁盘文件映射到虚拟内存，通常情况可以映射整个文件，如果文件比较大，可以进行分段映射。

```java
public static void main(String[] args) throws Exception{
        RandomAccessFile randomAccessFile = new RandomAccessFile("NioTest6.txt","rw");
        FileChannel fileChannel = randomAccessFile.getChannel();

        MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE,0,5);

        mappedByteBuffer.put(0,(byte)'a');
        mappedByteBuffer.put(3,(byte)'b');

        randomAccessFile.close();;
    }
```

运行前：

helloworld 

运行后：

aelboworld

MappedByteBuffer在处理大文件时的确性能很高，但也存在一些问题，其所对应的内存使用的是JVM堆外内存，JVM young gc和CMS gc并不能触发回收MappedByteBuffer对应的内存，只有full gc（stop the world的方式）可以使其回收内存，堆外直接内存会根据自己的情况（当需要新分配直接内存时，如果所剩堆外内存空间不够，第一次产生OutOfMemoryError时）来触发 System.gc()，此处有坑，若JVM配置了参数-XX:**DisableExplicitGC**，System.gc()将不会触发full gc，最终导致内存泄漏。而且触发其内存回收的时间点是不确定的。Java api文档中标注：

A mapped byte buffer and the file mapping that it represents remain valid until the buffer itself is garbage-collected.

在应用程序频繁使用堆外内存时，还可以通过-XX:MaxDirectMemorySize来指定最大的堆外内存大小，当使用达到了阈值的时候将调用System.gc来做一次full gc，以此来回收掉游离状态的堆外内存。
因此，在使用堆外内存高性能的福利的同时，及时的回收掉废弃掉的内存是十分关键的。

###  Scatter/Gather

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

Scattering Reads：

Scattering Reads是指数据从一个channel读取到多个buffer中。注意buffer首先被插入到数组，然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(译者注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

Gathering Writes:

Gathering Writes是指数据从多个buffer写入到同一个channel

buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。

```java
    public static void main(String[] args)throws Exception {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        InetSocketAddress address = new InetSocketAddress(8899);
        serverSocketChannel.socket().bind(address);
        SocketChannel socketChannel = serverSocketChannel.accept();

        int messageLength = 2+3+4;
        ByteBuffer[] buffers = new ByteBuffer[3];
        buffers[0] = ByteBuffer.allocate(2);
        buffers[1] = ByteBuffer.allocate(3);
        buffers[2] = ByteBuffer.allocate(4);

        while (true){
            int bytesRead = 0;
            while (bytesRead < messageLength){
                long r = socketChannel.read(buffers);
                bytesRead += r;
                System.out.println("bytesRead:"+bytesRead);

                Arrays.asList(buffers).stream()
                        .map(buffer -> "position:"+buffer.position()+",limit"+buffer.limit())
                        .forEach(System.out::println);
            }
            // 每个buffer执行flip然后输出
            Arrays.asList(buffers).forEach(buffer ->{
                buffer.flip();
            });
            long bytesWritten = 0;
            while (bytesWritten < messageLength){
                long r = socketChannel.write(buffers);
                bytesWritten += r;
            }
            Arrays.asList(buffers).forEach(buffer ->{
                buffer.clear();
            });
        }

    }
```

