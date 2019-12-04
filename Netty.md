#  Netty权威指南 第2版  
## 1.1 I/O划分成五种类型：  
### 1-阻塞2-非阻塞3-复用4-信号量5-异步  
这里首先要分清楚，一个完整的I/O流程大体上可以分成  
1-文件准备（Linux系统以FD代表文件对象，应用程序询问FD的状态需要发起系统调用，linux的常用的系统调用为recvfrom-文件读取阶段,
select/poll-fd询问阶段,升级版epoll）  
2-文件从内核空间去copy到用户空间（更深入的一步是从物理存储设备如HD中读取到内核空间）  

1）-阻塞（BI/O）在整个I/O过程中，都是阻塞的，通过系统调用recvfrom,发起I/O请求。  
2）-非阻塞（NI/O），在询问fd的过程中不是阻塞的，立刻返回是否准备好的结果，但是在第二阶段文件的
3）-复用模型，简单理解就是在fd询问阶段，接入selector的概念，一个selector可以同时监控多个fd的准备状态。
（这里系统调用-select是顺序执行的，并且可以监控的fd有限，linux体统了epoll，采用事件驱动机制，任一fd准备好了直接调用callback，就可以解除这些限制）  
4）-信号量，增加信号量，在fd准备好的时候，改变信号量（发起请求的同时设置信号量，不是阻塞的），相关程序进行处理。  
5）-异步(AI/O)，全程异步，只有在I/O处理完成才进行通知，其他都是fd准备好的时候进行通知后，但是还需要文件读取的阶段。  

### select与epoll的比较  
1 select监控fd的数目收到linux默认配置1024个的限制，epoll则是可以监控所有fd，所以受限的大小就为系统能容纳fd的大小。  
2 select/poll需要扫描活跃的socket对应的fd，那么要找出活跃的socket那么就需要扫描，如果接入的socket的数量巨大，那么效率肯定是底下的，epoll采用的事件
驱动，那么肯定只有活跃的socket（需要对fd操作）才会触发callback函数，从而避免了这个问题。  
3 为了避免不是必须的从内核拷贝到用户空间这一步操作，epoll采用了mmp技术，提高了效率。

### ByteBuff  
缓冲区主要使用形式（针对不同类型还会有ShortBuff等），主要的数据处理的结构化对象，提供了多个api。  
### Channel  
接收数据的对象，并且是双向的。  
### Selector  
多路复用器，主要作用是轮询channel，然后可以通过SelectionKey找出已经准备好了的channel。


### Netty编程的主要结构  
1-核心主件 ServerBootstrp  
2-两个EventLoopGroup 1)bossGroup-连接池（只有服务端会有，客户端不惜要）2）workerGroup-工作线程组（处理I/O请求）  
3-NIOServerSocketChannel，channel文件操作的容器对象。  
4-ChannelOption，channel的设置选项。  
5-ChannelHandler,处理channel数据的逻辑处理对象，采用的是事件机制，分别由channelActive，channelRead，exceptionCaught三个事件触发接口。  
6-PipeLine，channel的处理链接，用于handler的绑定。
7-Port，用于服务端最后的接口绑定bind（port）。
8-host，那么客户端就要发起对服务器的请求，connect（host，port).sync(),表示阻塞在客户端阻塞。最后要记得关闭资源，包括future跟group。
9-为了解决粘包问题，提供了Decoder，StringDecoder,LineBasedFrameDecoder，使用方式就是跟handler一起添加到channel的pipeLine。   


### TCP/IP传输过程中的拆包/粘包的处理  
简单的来说，就是需要的数据被拆分或者跟别的数据一起传输，但是对于网络传输来说是没有业务逻辑判断的，所以需要自己解决数据的问题。  


### Netty编解码技术  
java提供的默认序列化技术Serialiazable接口存在缺陷：  
1-无法跨语言，只能在java之间进行传输。  
2-编码后，流的大小，在相同的环境下，决定了传输的效率，java序列化编码之后的流很大，也就是效率低下。  
3-编解码的性能，java对应的效率也不高。  
总结：这样也侧面说明了，为什么rpc框架都有额外的序列化协议，而不是采用java默认。  
比较出名的序列化协议有Google的Protobuf,Facebook的Thrift

### 底层源码细节  
1-在构建netty代码中，用了很多反射，来构建实例，比如channel都是以class的形式传入的。  
2-有两种不同的channelHandler，NIOServerSocketChannel是在第一阶段，处理多个客户端连接网路连接bossGroup的对象类型，对应的是handler，NIOSocketChannel是在后续WorkerGroup里面的channel，对应的是childHandler。  
3-backlog这个参数用来表示连接队列中（已连接，未连接）的总数，判断的一句是tcp三次握手的不同阶段。

### ByteBuf  
1-从java的Buffer类衍生而来，对应的类叫ByteBuffer。  
2-通过两个指针ReadIndex跟WriteIndex来控制缓存区域。  
3-调用clear操作，并不会清空内存，而只是移动指针，这样可以减少操作。 

###  注意点
注册到selector是在I/O线程完成的，boss线程只是负责建立tcp链接。  


