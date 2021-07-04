三歪：真没想到还有面试官会问泛型和注解...

三歪：要不是我在项目中写过，我不就凉了？

三歪：希望下次别问我NIO这种现实项目是怎么用的，我在项目中是真没怎么写过

三歪：面试官，请问现在可以开始了吗

面试官：嗯，开始吧。

面试官：这次咱们就来聊聊Java 的NIO呗？你对NIO有多少了解？

三歪：嗯，我对Java NIO还是有一定的了解的，NIO是JDK 1.4 开始有的，其目的是为了提高速度。NIO翻译成 no-blocking io 或者 new io 都无所谓啦，反正都说得通

面试官：你在真实项目中写过NIO相关的吗？

三歪：这块在我所负责的系统中，一般用不上NIO，要不我跟你讲讲NIO相关的知识点呗？

面试官：可以吧，你先来讲讲NIO和传统IO有什么区别吧

三歪：传统IO是一次一个字节地处理数据，NIO是以块（缓冲区）的形式处理数据。最主要的是，NIO可以实现非阻塞，而传统IO只能是阻塞的。

三歪：IO的实际场景是文件IO和网络IO，NIO在网络IO场景下提升就尤其明显了。

三歪：在Java NIO有三个核心部分组成。分别是Buffer（缓冲区）、Channel（管道）以及Selector（选择器）

三歪：可以简单的理解为：Buffer是存储数据的地方，Channel是运输数据的载体，而Selector用于检查多个Channel的状态变更情况，

三歪：我曾经写过一个NIO Demo，面试官可以看看。

```java
public class NoBlockServer {

    public static void main(String[] args) throws IOException {

        // 1.获取通道
        ServerSocketChannel server = ServerSocketChannel.open();

        // 2.切换成非阻塞模式
        server.configureBlocking(false);

        // 3. 绑定连接
        server.bind(new InetSocketAddress(6666));

        // 4. 获取选择器
        Selector selector = Selector.open();

        // 4.1将通道注册到选择器上，指定接收“监听通道”事件
        server.register(selector, SelectionKey.OP_ACCEPT);

        // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
        while (selector.select() > 0) {
            // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();

            // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
            while (iterator.hasNext()) {

                SelectionKey selectionKey = iterator.next();

                // 接收事件就绪
                if (selectionKey.isAcceptable()) {

                    // 8. 获取客户端的链接
                    SocketChannel client = server.accept();

                    // 8.1 切换成非阻塞状态
                    client.configureBlocking(false);

                    // 8.2 注册到选择器上-->拿到客户端的连接为了读取通道的数据(监听读就绪事件)
                    client.register(selector, SelectionKey.OP_READ);

                } else if (selectionKey.isReadable()) { // 读事件就绪

                    // 9. 获取当前选择器读就绪状态的通道
                    SocketChannel client = (SocketChannel) selectionKey.channel();

                    // 9.1读取数据
                    ByteBuffer buffer = ByteBuffer.allocate(1024);

                    // 9.2得到文件通道，将客户端传递过来的图片写到本地项目下(写模式、没有则创建)
                    FileChannel outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);

                    while (client.read(buffer) > 0) {
                        // 在读之前都要切换成读模式
                        buffer.flip();

                        outChannel.write(buffer);

                        // 读完切换成写模式，能让管道继续读取文件的数据
                        buffer.clear();
                    }
                }
                // 10. 取消选择键(已经处理过的事件，就应该取消掉了)
                iterator.remove();
            }
        }

    }
}
```



```java
public class NoBlockClient {

    public static void main(String[] args) throws IOException {

        // 1. 获取通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 6666));

        // 1.1切换成非阻塞模式
        socketChannel.configureBlocking(false);

        // 1.2获取选择器
        Selector selector = Selector.open();

        // 1.3将通道注册到选择器中，获取服务端返回的数据
        socketChannel.register(selector, SelectionKey.OP_READ);

        // 2. 发送一张图片给服务端吧
        FileChannel fileChannel = FileChannel.open(Paths.get("X:\\Users\\ozc\\Desktop\\面试造火箭\\1.png"), StandardOpenOption.READ);

        // 3.要使用NIO，有了Channel，就必然要有Buffer，Buffer是与数据打交道的呢
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        // 4.读取本地文件(图片)，发送到服务器
        while (fileChannel.read(buffer) != -1) {

            // 在读之前都要切换成读模式
            buffer.flip();

            socketChannel.write(buffer);

            // 读完切换成写模式，能让管道继续读取文件的数据
            buffer.clear();
        }


        // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
        while (selector.select() > 0) {
            // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();

            // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
            while (iterator.hasNext()) {

                SelectionKey selectionKey = iterator.next();

                // 8. 读事件就绪
                if (selectionKey.isReadable()) {

                    // 8.1得到对应的通道
                    SocketChannel channel = (SocketChannel) selectionKey.channel();

                    ByteBuffer responseBuffer = ByteBuffer.allocate(1024);

                    // 9. 知道服务端要返回响应的数据给客户端，客户端在这里接收
                    int readBytes = channel.read(responseBuffer);

                    if (readBytes > 0) {
                        // 切换读模式
                        responseBuffer.flip();
                        System.out.println(new String(responseBuffer.array(), 0, readBytes));
                    }
                }

                // 10. 取消选择键(已经处理过的事件，就应该取消掉了)
                iterator.remove();
            }
        }
    }

}
```

面试官：这都是些API相关的知识，能看得出来你有一定的基础

面试官：但是你又没有在工作中实际用NIO，那我就考考相关的概念原理呗

三歪：好吧...

面试官：你知道IO模型有几种吗

三歪：在Unix下IO模型分别有：阻塞IO、非阻塞IO、IO复用、信号驱动以及异步I/O。在开发中碰得最多的就是阻塞IO、非阻塞IO以及IO复用。

面试官：嗯，来重点讲讲IO复用模型吧

三歪：我就以Linux系统为例好了，我们都知道Linux对文件的操作实际上就是通过文件描述符(fd)

三歪：IO复用模型指的就是：通过一个进程监听多个文件描述符，一旦某个文件描述符准备就绪，就去通知程序做相对应的处理

三歪：这种以通知的方式，优势并不是对于单个连接能处理得更快，而是在于它能处理更多的连接。

三歪：在Linux下IO复用模型用的函数有select/poll和epoll

面试官：那你来讲讲这select和epoll函数的区别呗？

三歪：嗯，先说select吧。

三歪：select函数它支持最大的连接数是1024或2048，因为在select函数下要传入fd_set参数，这个fd_set的大小要么1024或2048（其实就看操作系统的位数）

三歪：fd_set就是bitmap的数据结构，可以简单理解为只要位为0，那说明还没数据到缓冲区，只要位为1，那说明数据已经到缓冲区。

三歪：而select函数做的就是每次将fd_set遍历，判断标志位有没有发现变化，如果有变化则通知程序做中断处理。

三歪：下面来说下epoll

三歪：epoll 是在Linux2.6内核正式提出，完善了select 的一些缺点。

三歪：它定义了epoll_event结构体来处理，不存在最大连接数的限制。

三歪：并且它不像select函数每次把所有的文件描述符(fd)都遍历，简单理解就是epoll把就绪的文件描述符(fd)专门维护了一块空间，每次从就绪列表里边拿就好了，不再进行对所有文件描述符(fd)进行遍历。

面试官：嗯。你知道什么叫做零拷贝吗？

三歪：知道的。我们以读操作为例，假设用户程序发起一次读请求。

三歪：其实会调用read相关的「系统函数」，然后会从用户态切换到内核态，随后CPU会告诉DMA去磁盘把数据拷贝到内核空间。

三歪：等到「内核缓冲区」真正有数据之后，CPU会把「内核缓存区」数据拷贝到「用户缓冲区」，最终用户程序才能获取到。

三歪：稍微解释一下：为了保证内核的安全，操心系统将虚拟空间划分为「用户空间」和「内核空间」，所以在读系统数据的时候会有状态切换

三歪：因为应用程序不能直接去读取硬盘的数据，从上面描述可知需要依赖「内核缓冲区」

三歪：一次读操作会让DMA将磁盘数据拷贝到内核缓冲区，CPU将内核缓冲区数据拷贝到用户缓冲区。

三歪：所谓的零拷贝就是将「CPU将内核缓冲区数据拷贝到用户缓冲区」这次CPU拷贝给省去，来提高效率和性能

三歪：常见的零拷贝技术有mmap（内核缓冲区与用户缓冲区的共享）、sendfile（系统底层函数支持）。

三歪：零拷贝可以提高数据传输的性能，这块在Kafka等框架也有相关的实践。

面试官：嗯，了解。要不今天面试就到这里为止了，你有什么想问我吗？

三歪：我还有机会吗？

面试官：

