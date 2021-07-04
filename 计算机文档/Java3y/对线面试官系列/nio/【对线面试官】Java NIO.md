![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2ibfwu39j30ku112dk0.jpg)



![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2if0fr9nj30ku112wj8.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2igjspsmj30ku112jwk.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2ihvp3qvj30ku112n4k.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2ik8fbryj30ku11279k.jpg)

服务端：

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

客户端：

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

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2imlo2qrj30ku1127aw.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2iobd6u5j30ku112q8p.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2iqgmudkj30ku112jxt.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2ismuskzj30ku112n3w.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2iuvupmzj30ku112tg2.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2iwsnbq7j30ku112gs2.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2iz6tdkgj30ku112dm7.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2j2h9oirj30ku1120zr.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2j4puk3aj30ku112dm8.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwgy1gm2jf17dihj30ku112n2c.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm2jn7mivqj30ku1127cd.jpg)

**文章以纯面试的角度去讲解，所以有很多的细节是未铺垫的。**

比如说NIO的API使用讲解等等等...这些在【**Java3y**】都有过详细的基本教程甚至电子书，我就不再详述了。



欢迎关注我的微信公众号【**面试造火箭**】来聊聊Java面试，求点赞和关注，对我真的很重要！

![](https://tva1.sinaimg.cn/large/0081Kckwly1gluiplup1xj3076076glt.jpg)