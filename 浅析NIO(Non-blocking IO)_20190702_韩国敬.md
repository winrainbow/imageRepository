# 浅谈NIO(Non-blocking I/O)

> 是一种同步非阻塞I/O模型

主要从以下几个方面谈一下对NIO的理解：

1. 在NIO之前，传统IO是什么样的，有什么弊端？
2. 同步、异步、阻塞、非阻塞的概念
3. 程序在请求网络时，到底做了什么？和IO有什么关系？
4. NIO的原理
5. NIO的示例
6. NIO的适用场景

### 1> BIO (Blocking I/O)

传统IO(Blocking I/O):阻塞IO，常用的是异步阻塞IO，使用场景：一般我们请求网络会开了个新的线程或者从线程池中选一个空闲的线程去执行网络请求，在发出请求后、响应到来前，这个线程一直是等待（不继续干别的）是阻塞的，直到响应到来后，回调给调用线程后，该线程才会完成，不在占用CPU。

下面看一个例子：伪代码如下：

```java
{
        ExecutorService executor = Executors.newFixedThreadPool(100);
        ServerSocket serverSocket = null;

        serverSocket = new ServerSocket();
        serverSocket.bind(new InetSocketAddress(8088));//监听 8088端口
        while (true) {
            Socket socket = serverSocket.accept(); // 这个是阻塞的
            executor.submit(new ConnectIOHandler(socket)); //为新的连接创建新的线程
        }
}
class ConnectIOHandler implements Runnable {
        private Socket socket;

        public ConnectIOHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            if (!socket.isClosed()) {
                int read = socket.getInputStream().read();
                ............
                socket.getOutputStream().write(bytes);
            }
        }
}
/**
socket.accept()、socket.read()、socket.write()都是同步阻塞的，当一个连接在处理I/O时，系统是阻塞的，但cpu是释放的（瓶颈在I/O）必须使用多线程，
多线程本质：1.利用多核 2. 当I/O阻塞时，多线程使用CPU资源
注：现在多线程一般使用线程池，可以让线程的创建和回收成本相对较低，在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的I/O并且编程模型简单，不用过多的考虑系统的过载，限流等问题 ，线程池本身也是一个天然的漏斗，可以缓冲一些处理不了的连接或请求。
问题 ：
最本质的问题 ，严重依赖于线程，但线程是很贵的资源，主要表现在：
1. 线程的创建和销毁成本高，在linux中，线程本身是一个进程，创建和销毁都是重量级的系统函数
2. 线程本身占用较大的内存，像Java线程栈，一般至少分配 512K~1M的空间，如果系统中的线程数过多，恐怕整个jvm的内存会被吃掉一半
3. 线程切换的成本很高，操作系统在发生线程切换时，需要保留线程的上下文，然后执行系统调用，如果线程数过多，可能执行线程切换的时间会大于线程执行的时间，这时候带来的表现往往是系统load偏高，cpu sy使用率高，导致系统几乎不可用的状态
4. 容易造成锯齿状的系统负载，因为系统负载是用活动线程数或CPU核心数，一旦线程数量高但外部网络环境不是很稳定，就很容易造成大量请求的结果同时返回，激活大量阻塞线程从而使系统负载压力过大。
*/
```

### 2> 同步，异步，阻塞、非阻塞

- 同步：关注消息通讯机制，调用者进行调用后，会等待结果，有结果后才返回：调用者检查结果是否就绪

- 异步：也是关注消息通讯机制，调用后立刻返回，可能没有结果。等有结果后，由被调用者通过知调用者：被调用者检查调用结果是否就绪

- 阻塞：就等待结果时的状态，阻塞是指在调用结果返回之前，当前线程会被挂起，调用线程只有在得到结果之后才会继续执行

- 非阻塞：等 待结果时，调用线程不被挂起，还执行其他的事情

- 同步阻塞、同步非阻塞、异步非阻塞，异步阻塞

- 同步阻塞，一个线程请求网络，并等到请求结果回来

- 同步非阻塞：一个线程请求网络后，先去执行别的，不间断的来查看网络下载结果。 

- 异步阻塞：调用线程支请求网络，一直等待请求线程的下载完成通知

- 异步非阻塞：调用请求网络线程后，去干别的，等待来通知后在继续执行对应的逻辑，我们常用的网络请求方式

###3> 程序在请求网络时，到底做了什么？和IO有什么关系？

![IO](https://raw.githubusercontent.com/winrainbow/imageRepository/master/IO.jpg)



###4> NIO原理

> 1. 所有的系统的I/O都分为两个阶段：等待就绪和操作 如：读分为等待系统可读和真正的读，写分为等待网卡可以和真正的写
>
> 2. 需要说明的是等待就绪的阻塞是不使用cpu的，在空等，而真正的读写操作的阻塞是使用cpu的，是真正干活的，而且这个过程非常快，属于memory copy 带宽通常是在1GB/s级别以上，可以理解为基本不耗时
>
> 3. socket.read() 在BIO中，socket.read()如果TCP recvBuffer里没有数据，函数会一直阻塞，直到收到数据，返回读到的数据
>
>    对于NIO,如果TCP RecvBuffer有数据，就把数据从网卡读到内存中，并且返回给用户；反之直接返回0，永远不会阻塞
>
>    AIO（Async I/O）中，不但等待就绪是非阻塞的，连数据从网卡到内存的过程也是异步的
>
>    BIO:我要读；NIO:我可以读了 ；AIO:我读完了
>
> 4. NIO重要特点：socket主要的读、写、注册和接收函数，在等待就绪阶段都是非阻塞的，真正的I/O操作是同步阻塞的（消耗CPU但性能非常高）
>
> 5. BIO模型中，之所以需要多线程，是因为进行I/O操作时，一是没有办法知道能不能写，能不能读，只能“傻等”，即使通过各种估算，算出来操作系统没有能力进行读写，也没有办法在socket.read()和socket.write()方法中返回，这两个方法无法进行有效的中断所以除了多开线程另起炉灶，没有好的办法利用CPU
>
> 6. NIO的读写方法可以立刻返回，这就给了我们不开线程利用CPU的最好机会，如果一个连接不能读写（socket.read()或者socket.write()返回0） 我们就可以把这件事记下来，记录的方式通常是在 Selector上注册标记位，然后切换到其他就绪的连接（channel）继续进行读写
>
> 7. NIO几个事件：读就绪、写就绪、有新连接到来
>
>    1. 首先注册当这几个事件到来的时候所对应的处理器，然后在合适的时机告诉事件选择器：我对这个事件有兴趣；
>       * 对于写操作，就是写不出去的时候对写事件有兴趣，
>       * 对于读操作，就是完成连接和系统没有办法承载新读入的数据时
>       * 对于accept,一般是服务器刚启动的时候，
>       * 对于connect，一般是connect失败需要重连或者直接异步调用connect的时候
>
>     2. 用一个死循环选择就绪的事件，会执行系统调用（epoll,Windows:IOCP），还会阻塞等待事件的到来，新事件到来的时候，会在selector上注册标记位，标示可读、可写、有连接到来
>
>       select是阻塞的，无论是通过操作系统的通知，还是不停的轮询，这个函数是阻塞的，所以可以放心大胆的在一个while(true)里面调用这个方法，而不用担心cpu空转
>

###5> NIO示例

```java
package com.company;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;
/**
 * NIO server
 */
public class Main {
    public static void main(String[] args) throws IOException {
	    System.out.println("hello World");
	    new Server().start();
    }
    static class Server {
        /**
         * selector
         */
        private Selector selector;
        private ByteBuffer readBuffer = ByteBuffer.allocate(1024);
        private ByteBuffer sendBuffer = ByteBuffer.allocate(1024);
        private String tempContent;

        /**
         * 开启服务
         * @throws IOException
         */
        public void start() throws IOException{
            /**
             * 打开服务器 套接字通道
             */
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            /**
             * 设置为非阻塞
             */
            serverSocketChannel.configureBlocking(false);
            /**
             * 绑定要监听的端口号
             */
            serverSocketChannel.bind(new InetSocketAddress("localhost",8001));
            /**
             * 找到selector
             */
            selector = Selector.open();
            /**
             * socketServerChannel告之selector 我对 接入感兴趣，如果有准备好的接入，通知我
             */
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

            while (!Thread.currentThread().isInterrupted()){
                selector.select(); // 阻塞的
                Set<SelectionKey> selectionKeys = selector.selectedKeys(); // 准备好的事件，接入、读、写
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()){
                    SelectionKey key = iterator.next();
                    if(!key.isValid()){
                        return;
                    }
                    if(key.isAcceptable()){
                        accept(key);
                    }else if(key.isReadable()){
                        read(key);
                    }else if(key.isWritable()){
                        write(key);
                    }

                    iterator.remove();// 事件处理完，丢弃
                }
            }
        }

        private void write(SelectionKey key) throws IOException {
            SocketChannel channel = (SocketChannel) key.channel(); // 标识客户端过来的一个链接

            sendBuffer.clear(); // 清空
            sendBuffer.put(tempContent.getBytes());
            sendBuffer.flip(); // 改变position位置

            channel.write(sendBuffer); // 写数据
            channel.register(selector,SelectionKey.OP_READ); // 监听该链接 客户端发过来的数据
            System.out.println("发出:"+tempContent); // 要响应的内容
        }
        private void read(SelectionKey key) throws IOException {
            SocketChannel channel = (SocketChannel) key.channel(); // 标识客户端过来的一个链接
            tempContent = "";
            readBuffer.clear();
            int readLength = channel.read(readBuffer);// 读数据
            tempContent = new String(readBuffer.array(),0,readLength);
            System.out.println("收到："+tempContent);
            readBuffer.flip();
            channel.register(selector,SelectionKey.OP_WRITE); // 监听该链接，开始写数据
        }


        private void accept(SelectionKey key) throws IOException {
            ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
            SocketChannel channel = serverSocketChannel.accept();
            channel.configureBlocking(false);
            channel.register(selector,SelectionKey.OP_READ);
            System.out.println("a new client connected "+channel.getRemoteAddress());
        }
    }
}
```

NioClient

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;
import java.util.Set;
public class Main {
    public static void main(String[] args) throws IOException {
        System.out.println("I am client !");
        new Client().start();
    }
    static class Client {
        ByteBuffer readBuffer = ByteBuffer.allocate(1024);
        ByteBuffer writeBuffer = ByteBuffer.allocate(1024);

        public void start() throws IOException{
            SocketChannel  sc = SocketChannel.open();
            sc.configureBlocking(false);
            sc.connect(new InetSocketAddress("localhost",8001));
            Selector selector = Selector.open();
            sc.register(selector, SelectionKey.OP_CONNECT);
            Scanner scanner = new Scanner(System.in);
            while(true){
                selector.select();
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                System.out.println("selectionKeys.length:"+selectionKeys.size());
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()){
                    SelectionKey key = iterator.next();

                    if(key.isConnectable()){
                        System.out.println("selectionKeys_connectable");
                        // 连接成功
                        sc.finishConnect();
                        sc.register(selector,SelectionKey.OP_WRITE);
                        System.out.println("连接成功");

                    }else if(key.isWritable()){
                        System.out.println("selectionKeys_writable");
                        SocketChannel channel = (SocketChannel) key.channel();
                        InetSocketAddress localAddress = (InetSocketAddress) channel.getLocalAddress();

                        String ip = localAddress.getAddress().getLocalHost().toString();
                        String input = scanner.nextLine();
                        String writeStr = "hello i am client:["+ip+"]"+input;
                        writeBuffer.clear();
                        writeBuffer.put(writeStr.getBytes());
                        writeBuffer.flip();
                        channel.write(writeBuffer);
                        channel.register(selector,SelectionKey.OP_READ);
                    }else if(key.isReadable()){
                        System.out.println("selectionKeys_readable");
                        SocketChannel channel = (SocketChannel) key.channel();
                        readBuffer.clear();
                        int readLength = channel.read(readBuffer);
                        System.out.println("收到:"+new String(readBuffer.array(),0,readLength));
                        sc.register(selector,SelectionKey.OP_WRITE);
                    }
                    iterator.remove();
                }
            }
        }
    }
}

```

[NIOClient](https://github.com/winrainbow/imageRepository/raw/master/ClientMainTecent.zip "NIO")

### 6> NIO适用场景

如果你需要管理数以千计的同时打开的连接，每一个只发送一点点的数据，例如聊天服务器，在NIO中实现服务器可能是一个优势。 同样，如果你需要保持与其他计算机的大量开放连接，例如 在P2P网络中，使用单个线程来管理所有出站连接可能是一个优势







# 其他内容：

### 优化线程模型

NIO由原来的阻塞读写（占用线程）变程单线程轮询事件，找到可以进行读写的网络描述符进行读写，除了事件的轮询是阻塞的（没有可干的事情，必须阻塞），剩余的I/O操作都是纯CPU操作 ，没有必要开启多线程

并且由于线程的节约，连接数大的时候因为线程切带来的性 能问题也解决了，进而为海量连接提供了可能

单线程处理I/O的效率确实非常高，没有线程切换，只是拼命的读、写、选择事件。但现在的服务器一般是多核的处理器，如果能利用多核心进行I/O无疑对效率会有更大的提升

我们需要的线程：

1. 事件分发器：单线程选择就绪的事件
2. I/O处理器：包括connect、read、write等，这种纯cpu操作，一般开启的线程的个数和cpu核心个数相同
3. 业务线程：在处理完I/O后，业务一般会有自己的业务逻辑，有的还会有其他的阻塞I/O，如DB操作、RPC等，只要有阻塞，就需要单独的线程
4. Java的selector对于Linux系统来说有个限制，同一个channel的select不能被并发的调用。因此，如果有多个I/O线程，必须保证：一个socket只能属于一个IOthread,而一个IOThread可以管理多个socket。

### 事件分发器

一般情况下，I/O 复用机制需要事件分发器（event dispatcher）。 事件分发器的作用，即将那些读写事件源分发给各读写事件的处理者，就像送快递的在楼下喊: 谁谁谁的快递到了， 快来拿吧！开发人员在开始的时候需要在分发器那里注册感兴趣的事件，并提供相应的处理者（event handler)，或者是回调函数；事件分发器在适当的时候，会将请求的事件分发给这些handler或者回调函数。

涉及到事件分发器的两种模式称为：Reactor和Proactor。 Reactor模式是基于同步I/O的，而Proactor模式是和异步I/O相关的。在Reactor模式中，事件分发器等待某个事件或者可应用或个操作的状态发生（比如文件描述符可读写，或者是socket可读写），事件分发器就把这个事件传给事先注册的事件处理函数或者回调函数，由后者来做实际的读写操作。

而在Proactor模式中，事件处理者（或者代由事件分发器发起）直接发起一个异步读写操作（相当于请求），而实际的工作是由操作系统来完成的。发起时，需要提供的参数包括用于存放读到数据的缓存区、读的数据大小或用于存放外发数据的缓存区，以及这个请求完后的回调函数等信息。事件分发器得知了这个请求，它默默等待这个请求的完成，然后转发完成事件给相应的事件处理者或者回调。举例来说，在Windows上事件处理者投递了一个异步IO操作（称为overlapped技术），事件分发器等IO Complete事件完成。这种异步模式的典型实现是基于操作系统底层异步API的，所以我们可称之为“系统级别”的或者“真正意义上”的异步，因为具体的读写是由操作系统代劳的。

举个例子，将有助于理解Reactor与Proactor二者的差异，以读操作为例（写操作类似）。

> 在Reactor中实现读

- 注册读就绪事件和相应的事件处理器。
- 事件分发器等待事件。
- 事件到来，激活分发器，分发器调用事件对应的处理器。
- 事件处理器完成实际的读操作，处理读到的数据，注册新的事件，然后返还控制权。

>  在Proactor中实现读：

- 处理器发起异步读操作（注意：操作系统必须支持异步IO）。在这种情况下，处理器无视IO就绪事件，它关注的是完成事件。
- 事件分发器等待操作完成事件。
- 在分发器等待过程中，操作系统利用并行的内核线程执行实际的读操作，并将结果数据存入用户自定义缓冲区，最后通知事件分发器读操作完成。
- 事件分发器呼唤处理器。
- 事件处理器处理用户自定义缓冲区中的数据，然后启动一个新的异步操作，并将控制权返回事件分发器。


###EPoll（linux大于 2.6） 和 Poll(linux 小于2.6)

底层的实现，具体可以去看操作系统的底层实现

JavaNio 在java自身的语言层面上，使用了缓存和内存映射来提高IO的读写效率

###read()和write()

* read()两个阶段：1，等待数据准备（时间长）2.将数据从内核内存空间拷贝到进程的内存空间中


###Buffer的创建，及各种操作

### Netty:



