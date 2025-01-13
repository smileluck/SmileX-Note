##### <font size="4" color="red">01. Java中Netty集成Tcp服务器端(自定义编解码器)(1)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建`CarProtocol.java`文件**

`CarProtocol.java`文件中内容：

```java
import java.util.Arrays;

public class CarProtocol {

    //消息头
    private int head_data = 0x7878;
    //包长度
    private int packageLength;
    //协议号
    private int protocol;
    //消息内容
    private byte[] content;
    //消息结束
    private int end_data = 0x0d0a;

    public CarProtocol() {

    }

    public CarProtocol(int head_data, int packageLength, int protocol, byte[] content, int end_data) {
        this.head_data = head_data;
        this.packageLength = packageLength;
        this.protocol = protocol;
        this.content = content;
        this.end_data = end_data;
    }

    public int getHead_data() {
        return head_data;
    }

    public void setHead_data(int head_data) {
        this.head_data = head_data;
    }

    public int getPackageLength() {
        return packageLength;
    }

    public void setPackageLength(int packageLength) {
        this.packageLength = packageLength;
    }

    public int getProtocol() {
        return protocol;
    }

    public void setProtocol(int protocol) {
        this.protocol = protocol;
    }

    public byte[] getContent() {
        return content;
    }

    public void setContent(byte[] content) {
        this.content = content;
    }

    public int getEnd_data() {
        return end_data;
    }

    public void setEnd_data(int end_data) {
        this.end_data = end_data;
    }

    @Override
    public String toString() {
        return "CarProtocol [head_data=" + head_data + ", packageLength=" + packageLength + ", protocol=" + protocol
            + ", content=" + Arrays.toString(content) + ", end_data=" + end_data + "]";
    }

}
```

**(3) 创建`MyDecoder.java`文件**

`MyDecoder.java`文件中内容：

```java
import java.util.List;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ByteToMessageDecoder;

public class MyDecoder extends ByteToMessageDecoder {

    //协议开始的标准head_data，int类型，占据4个字节. 
    //表示数据的长度contentLength，int类型，占据4个字节. 
    //数据包基础长度
    private final int base_len = 10;

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        //可读长度大于基本数据长度
        if (in.readableBytes() >= base_len) {
            // 因为，太大的数据，是不合理的  
            if (in.readableBytes() > 2048) {  
                in.skipBytes(in.readableBytes());  
            } 
            //记录包头位置
            int beginIdx; 
            while (true) {
                //获取包头开始的index
                beginIdx = in.readerIndex();
                //标记包头开始的index
                in.markReaderIndex();
                //读到了协议的开始标志，结束while循环
                if (in.readShort() == 0x7878) {
                    break;
                }
                // 未读到包头，略过一个字节
                // 每次略过，两个字节个字节，去读取，包头信息的开始标记
                in.resetReaderIndex();
                in.readByte();
                // 当略过，一个字节之后，
                // 数据包的长度，又变得不满足
                // 此时，应该结束。等待后面的数据到达
                if (in.readableBytes() < base_len) {
                    return;
                }
            }
            //读取消息长度只占一位
            int length = in.readByte();
            //协议号
            int protocol = in.readByte();
            //判断请求数据包数据是否到齐
            if (in.readableBytes() < length) {  
                //还原读指针  
                in.readerIndex(beginIdx);
                return; 
            } 
            //读取data数据  
            byte[] content = new byte[length];  
            in.readBytes(content);
            System.out.println(content);
            CarProtocol carProtocol = new CarProtocol();
            carProtocol.setProtocol(protocol);
            carProtocol.setContent(content);
            System.out.println(carProtocol);
            out.add(carProtocol);
        }
    }
}
```

**(4) 创建`MyEncoder.java`文件**

`MyEncoder.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MyEncoder extends MessageToByteEncoder<CarProtocol> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, CarProtocol carProtocol, ByteBuf out) throws Exception {
        out.writeShort(carProtocol.getHead_data());
        out.writeByte(carProtocol.getProtocol());
        out.writeBytes(carProtocol.getContent());
        out.writeShort(carProtocol.getEnd_data());
    }
}
```

**(5) 创建`nettyTcp.java`文件**

`nettyTcp.java`文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class nettyTcp {

    public void TcpServer() {
        //线程数
        int boss_os_num = 8;
        //数据接收线程组
        EventLoopGroup bossLoop = new NioEventLoopGroup(boss_os_num);
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            ServerBootstrap serverBoot = new ServerBootstrap();
            //配置Tcp参数
            serverBoot.group(bossLoop,eventLoop)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 1024 * 1024)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new TcpChannel());
            ChannelFuture cf = serverBoot.bind(8080).sync();
            //等待线程池结束
            cf.channel().closeFuture().sync();
        }
        catch(Exception e) {

            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
        finally {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }  
    }
}
```

**(6) 创建`TcpChannel.java`文件**

`TcpChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;

public class TcpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline()
        //自定义解码器
        .addLast(new MyDecoder())
        //自定义编码器
        .addLast(new MyEncoder())
        .addLast(new TcpHandler());
    }
}
```

**(7) 创建`TcpHandler.java`文件**

`TcpHandler.java`文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class TcpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        CarProtocol carProtocol = (CarProtocol) msg;  
        ctx.writeAndFlush(carProtocol);  
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }

}
```

**(8) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {
    public static void main(String[] args) throws Exception {
        new nettyTcp().TcpServer();
    }

}
```

***

##### <font size="4" color="red">02. Java中Netty集成Tcp客户端(自定义编解码器)(1)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建`CarProtocol.java`文件**

`CarProtocol.java`文件中内容：

```java
import java.util.Arrays;

public class CarProtocol {

    //消息头
    private int head_data = 0x7878;
    //包长度
    private int packageLength;
    //协议号
    private int protocol;
    //消息内容
    private byte[] content;
    //消息结束
    private int end_data = 0x0d0a;

    public CarProtocol() {

    }

    public CarProtocol(int head_data, int packageLength, int protocol, byte[] content, int end_data) {
        this.head_data = head_data;
        this.packageLength = packageLength;
        this.protocol = protocol;
        this.content = content;
        this.end_data = end_data;
    }

    public int getHead_data() {
        return head_data;
    }

    public void setHead_data(int head_data) {
        this.head_data = head_data;
    }

    public int getPackageLength() {
        return packageLength;
    }

    public void setPackageLength(int packageLength) {
        this.packageLength = packageLength;
    }

    public int getProtocol() {
        return protocol;
    }

    public void setProtocol(int protocol) {
        this.protocol = protocol;
    }

    public byte[] getContent() {
        return content;
    }

    public void setContent(byte[] content) {
        this.content = content;
    }

    public int getEnd_data() {
        return end_data;
    }

    public void setEnd_data(int end_data) {
        this.end_data = end_data;
    }

    @Override
    public String toString() {
        return "CarProtocol [head_data=" + head_data + ", packageLength=" + packageLength + ", protocol=" + protocol
            + ", content=" + Arrays.toString(content) + ", end_data=" + end_data + "]";
    }
}
```

**(3) 创建`MyDecoder.java`文件**

`MyDecoder.java`文件中内容：

```java
import java.util.List;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ByteToMessageDecoder;

public class MyDecoder extends ByteToMessageDecoder {

    //协议开始的标准head_data，int类型，占据4个字节. 
    //表示数据的长度contentLength，int类型，占据4个字节. 
    //数据包基础长度
    private final int base_len = 10;

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        //可读长度大于基本数据长度
        if (in.readableBytes() >= base_len) {
            // 因为，太大的数据，是不合理的  
            if (in.readableBytes() > 2048) {  
                in.skipBytes(in.readableBytes());  
            } 
            //记录包头位置
            int beginIdx; 
            while (true) {
                //获取包头开始的index
                beginIdx = in.readerIndex();
                //标记包头开始的index
                in.markReaderIndex();
                //读到了协议的开始标志，结束while循环
                if (in.readShort() == 0x7878) {
                    break;
                }
                // 未读到包头，略过一个字节
                // 每次略过，两个字节个字节，去读取，包头信息的开始标记
                in.resetReaderIndex();
                in.readByte();
                // 当略过，一个字节之后，
                // 数据包的长度，又变得不满足
                // 此时，应该结束。等待后面的数据到达
                if (in.readableBytes() < base_len) {
                    return;
                }
            }
            //读取消息长度只占一位
            int length = in.readByte();
            //协议号
            int protocol = in.readByte();
            //判断请求数据包数据是否到齐
            if (in.readableBytes() < length) {  
                //还原读指针  
                in.readerIndex(beginIdx);
                return; 
            } 
            //读取data数据  
            byte[] content = new byte[length];  
            in.readBytes(content);
            System.out.println(content);
            CarProtocol carProtocol = new CarProtocol();
            carProtocol.setProtocol(protocol);
            carProtocol.setContent(content);
            System.out.println(carProtocol);
            out.add(carProtocol);
        }
    }
}
```

**(4) 创建`MyEncoder.java`文件**

`MyEncoder.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MyEncoder extends MessageToByteEncoder<CarProtocol> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, CarProtocol carProtocol, ByteBuf out) throws Exception {
        out.writeShort(carProtocol.getHead_data());
        out.writeByte(carProtocol.getProtocol());
        out.writeBytes(carProtocol.getContent());
        out.writeShort(carProtocol.getEnd_data());
    }
}
```

**(5) 创建`nettyTcp.java`文件**

`nettyTcp.java`文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

public class nettyTcp {

    public void TcpClient() {
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            Bootstrap ClientBoot = new Bootstrap();
            //配置Tcp参数
            ClientBoot.group(eventLoop)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .handler(new TcpChannel());
            ChannelFuture cf = ClientBoot.connect("127.0.0.1",8006).sync();
        }
        catch(Exception e) {
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
    }

}
```

**(6) 创建`TcpChannel.java`文件**

`TcpChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;

public class TcpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline()
        //自定义解码器
        .addLast(new MyDecoder())
        //自定义编码器
        .addLast(new MyEncoder())
        .addLast(new TcpHandler());
    }
}
```

**(7) 创建`TcpHandler.java`文件**

`TcpHandler.java`文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class TcpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        CarProtocol carProtocol = (CarProtocol) msg;  
        ctx.writeAndFlush(carProtocol);  
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }

}
```

**(8) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {
    public static void main(String[] args) throws Exception {
        new nettyTcp().TcpClient();
    }
}
```

***

##### <font size="4" color="red">03. Java中Netty集成Tcp服务器端(自定义编解码器)(2)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建`MyDecoder.java`文件**

`MyDecoder.java`文件中内容：

```java
import java.nio.charset.Charset;
import java.util.List;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ByteToMessageDecoder;

public class MyDecoder extends ByteToMessageDecoder {

    //数据包基础长度
    private final int base_len = 4;

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf in, List<Object> out) throws Exception {

        //基础长度不足，我们设定基础长度为4
        if (in.readableBytes() < base_len) {
            return;
        }

        int beginIdx; //记录包头位置

        while (true) {
            // 获取包头开始的index
            beginIdx = in.readerIndex();
            // 标记包头开始的index
            in.markReaderIndex();
            // 读到了协议的开始标志，结束while循环
            if (in.readByte() == 0x02) {
                break;
            }
            // 未读到包头，略过一个字节
            // 每次略过，一个字节，去读取，包头信息的开始标记
            in.resetReaderIndex();
            in.readByte();
            // 当略过，一个字节之后，
            // 数据包的长度，又变得不满足
            // 此时，应该结束。等待后面的数据到达
            if (in.readableBytes() < base_len) {
                return;
            }
        }

        //剩余长度不足可读取数量[没有内容长度位]
        int readableCount = in.readableBytes();
        if (readableCount <= 1) {
            in.readerIndex(beginIdx);
            return;
        }

        //长度域占4字节，读取int
        ByteBuf byteBuf = in.readBytes(1);
        String msgLengthStr = byteBuf.toString(Charset.forName("GBK"));
        int msgLength = Integer.parseInt(msgLengthStr);

        //剩余长度不足可读取数量[没有结尾标识]
        readableCount = in.readableBytes();
        if (readableCount < msgLength + 1) {
            in.readerIndex(beginIdx);
            return;
        }

        ByteBuf msgContent = in.readBytes(msgLength);
        //如果没有结尾标识，还原指针位置[其他标识结尾]
        byte end = in.readByte();
        if (end != 0x03) {
            in.readerIndex(beginIdx);
            return;
        }
        out.add(msgContent.toString(Charset.forName("GBK")));
    }
}
```

**(3) 创建`MyEncoder.java`文件**

`MyEncoder.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MyEncoder extends MessageToByteEncoder<Object> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, Object in, ByteBuf out) throws Exception {
        String msg = in.toString();
        byte[] bytes = msg.getBytes();
        byte[] send = new byte[bytes.length + 2];
        System.arraycopy(bytes, 0, send, 1, bytes.length);
        send[0] = 0x02;
        send[send.length - 1] = 0x03;
        out.writeInt(send.length);
        out.writeBytes(send);
    }
}
```

**(4) 创建`nettyTcp.java`文件**

`nettyTcp.java`文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class nettyTcp {

    public void TcpServer() {
        //线程数
        int boss_os_num = 8;
        //数据接收线程组
        EventLoopGroup bossLoop = new NioEventLoopGroup(boss_os_num);
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            ServerBootstrap serverBoot = new ServerBootstrap();
            //配置Tcp参数
            serverBoot.group(bossLoop,eventLoop)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 1024 * 1024)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new TcpChannel());
            ChannelFuture cf = serverBoot.bind(8080).sync();
            //等待线程池结束
            cf.channel().closeFuture().sync();
        }
        catch(Exception e) {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
        finally {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }  
    }
}
```

**(5) 创建`TcpChannel.java`文件**

`TcpChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
public class TcpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline()
        //自定义解码器
        .addLast(new MyDecoder())
        //自定义编码器
        .addLast(new MyEncoder())
        .addLast(new TcpHandler());

    }
}
```

**(6) 创建`TcpHandler.java`文件**

**`TcpHandler.java`文件**

`TcpHandler.java`文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class TcpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        //地址标识
        System.out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()) + " 接收到消息：" + msg);
        ctx.writeAndFlush("hi I'm ok");
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }
}
```

**(7) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        new nettyTcp().TcpServer();
    }

}
```

***

##### <font size="4" color="red">04. Java中Netty集成Tcp客户端(自定义编解码器)(2)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建`MyDecoder.java`文件**

`MyDecoder.java`文件中内容：

```java
import java.nio.charset.Charset;
import java.util.List;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ByteToMessageDecoder;

public class MyDecoder extends ByteToMessageDecoder {

    //数据包基础长度
    private final int base_len = 4;

    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf in, List<Object> out) throws Exception {

        //基础长度不足，我们设定基础长度为4
        if (in.readableBytes() < base_len) {
            return;
        }

        int beginIdx; //记录包头位置

        while (true) {
            // 获取包头开始的index
            beginIdx = in.readerIndex();
            // 标记包头开始的index
            in.markReaderIndex();
            // 读到了协议的开始标志，结束while循环
            if (in.readByte() == 0x02) {
                break;
            }
            // 未读到包头，略过一个字节
            // 每次略过，一个字节，去读取，包头信息的开始标记
            in.resetReaderIndex();
            in.readByte();
            // 当略过，一个字节之后，
            // 数据包的长度，又变得不满足
            // 此时，应该结束。等待后面的数据到达
            if (in.readableBytes() < base_len) {
                return;
            }
        }

        //剩余长度不足可读取数量[没有内容长度位]
        int readableCount = in.readableBytes();
        if (readableCount <= 1) {
            in.readerIndex(beginIdx);
            return;
        }

        //长度域占4字节，读取int
        ByteBuf byteBuf = in.readBytes(1);
        String msgLengthStr = byteBuf.toString(Charset.forName("GBK"));
        int msgLength = Integer.parseInt(msgLengthStr);

        //剩余长度不足可读取数量[没有结尾标识]
        readableCount = in.readableBytes();
        if (readableCount < msgLength + 1) {
            in.readerIndex(beginIdx);
            return;
        }

        ByteBuf msgContent = in.readBytes(msgLength);
        //如果没有结尾标识，还原指针位置[其他标识结尾]
        byte end = in.readByte();
        if (end != 0x03) {
            in.readerIndex(beginIdx);
            return;
        }
        out.add(msgContent.toString(Charset.forName("GBK")));
    }
}
```

**(3) 创建`MyEncoder.java`文件**

`MyEncoder.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MyEncoder extends MessageToByteEncoder<Object> {

    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, Object in, ByteBuf out) throws Exception {
        String msg = in.toString();
        byte[] bytes = msg.getBytes();
        byte[] send = new byte[bytes.length + 2];
        System.arraycopy(bytes, 0, send, 1, bytes.length);
        send[0] = 0x02;
        send[send.length - 1] = 0x03;
        out.writeInt(send.length);
        out.writeBytes(send);
    }
}
```

**(4) 创建`nettyTcp.java`文件**

`nettyTcp.java`文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

public class nettyTcp {

    public void TcpClient() {
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            Bootstrap ClientBoot = new Bootstrap();
            //配置Tcp参数
            ClientBoot.group(eventLoop)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .handler(new TcpChannel());
            ChannelFuture cf = ClientBoot.connect("127.0.0.1",8006).sync();
        }
        catch(Exception e) {
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
    }
}
```

**(5) 创建`TcpChannel.java`文件**

`TcpChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
public class TcpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline()
        //自定义解码器
        .addLast(new MyDecoder())
        //自定义编码器
        .addLast(new MyEncoder())
        .addLast(new TcpHandler());

    }
}
```

**(6) 创建`TcpHandler.java`文件**

**`TcpHandler.java`文件**

`TcpHandler.java`文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class TcpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        //地址标识
        System.out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()) + " 接收到消息：" + msg);
        ctx.writeAndFlush("hi I'm ok");
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }
}
```

**(7) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        new nettyTcp().TcpClient();
    }
}
```

***

##### <font size="4" color="red">05. Java中余弦推荐算法</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.34</version>
</dependency>
```

**(2) `Mysql`数据库中创建`testdb`数据库**

**(3) `testdb`中创建用户信息`member_user`表**

```sql
create table member_user
(
    USER_ID   int(10) auto_increment
        primary key,
    USER_NAME varchar(20) null
)
    engine = MyISAM
    charset = utf8;

INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (1, '郑成功');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (2, '小红');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (7, '小李');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (19, '郑晖');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (10, '张三');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (11, '二龙湖浩哥');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (12, '张三炮');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (13, '赵四');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (14, '刘能');
INSERT INTO testdb.member_user (USER_ID, USER_NAME) VALUES (15, '刘能逗');
```

**(4) `testdb`中创建订单记录`product_order`表**

```sql
create table product_order
(
    ORDER_ID     int auto_increment
        primary key,
    USER_ID      int          not null,
    PRODUCT_ID   int          not null,
    GWCOUNT      int          null,
    out_trade_no varchar(100) null
);

INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (1, 1, 1, 15, '202001');
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (2, 2, 3, 42, '202002');
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (3, 3, 4, 2, '202003');
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (4, 4, 4, 20, '202004');
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (5, 1, 2, 21, '202005');
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (6, 5, 1, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (7, 5, 2, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (8, 5, 3, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (9, 6, 2, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (10, 6, 5, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (11, 7, 1, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (12, 7, 2, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (13, 7, 5, null, null);
INSERT INTO testdb.product_order (ORDER_ID, USER_ID, PRODUCT_ID, GWCOUNT, out_trade_no) VALUES (14, 3, 1, null, null);
```

**(5) `testdb`中创建商品信息`product_table`表**

```sql
create table product_table
(
    productID    int auto_increment comment '商品ID'
        primary key,
    product_name varchar(200) charset utf8 null comment '商品名字',
    price        double                    null comment '商品金额',
    volume       int                       null comment '成交数量',
    shopp_name   varchar(100) charset utf8 null comment '商店名称',
    location     varchar(100) charset utf8 null comment '生产地',
    evaluate     int                       null comment '好评数量',
    collect      int default 0             null comment '收藏数量'
)
    engine = MyISAM
    collate = utf8_unicode_ci;

INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (1, '杨梅新鲜现摘现发正宗仙居东魁大杨梅孕妇农家时令水果6斤装包邮', 198, 1328, '浙仙旗舰店', '浙江 台州', 240, 1397);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (2, '正宗仙居东魁杨梅新鲜现摘特级大东魁现摘现发农家时令水5斤精选', 268, 497, '浙仙旗舰店', '浙江 台州', 159, 619);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (3, '新鲜洋葱10斤紫皮红皮农家2020年葱头应季蔬菜圆头整箱批发包邮', 18.8, 64, '悠鲜源旗舰店', '云南 昆明', 7, 51);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (4, '轩农谷仙居东魁杨梅新鲜水果浙江现摘现发大杨梅礼盒预定7A6斤', 358, 222, '轩农谷旗舰店', '浙江 台州', 553, 1163);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (5, '轩农谷正宗仙居杨梅新鲜当季水果特级东魁大杨梅5A级6斤高山现摘', 258, 2939, '轩农谷旗舰店', '浙江 台州', 4270, 8737);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (6, '水果小黄瓜新鲜6斤当季山东小青瓜生吃农家蔬菜助白玉女瓜10批发5', 23.8, 4, '喜人喜食品旗舰店', '山东 潍坊', 21685, 10511);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (7, '王小二 新鲜马蹄荸荠地梨孛荠当季马蹄莲5斤饽荠农家自种蔬菜包邮', 19.9, 1780, '王小二旗舰店', '湖北 宜昌', 1428, 858);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (8, '王小二 云南新鲜蚕豆农家罗汉豆带壳生兰花豆胡豆豌豆蔬菜包邮5斤', 29.9, 54, '王小二旗舰店', '云南 昆明', 72, 68);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (9, '王小二 小米椒新鲜红辣椒蔬菜包邮红尖椒灯笼椒朝天包邮农家5斤', 29.9, 601, '王小二旗舰店', '山东 潍坊', 435, 686);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (10, '王小二 湖北土辣椒新鲜青椒农家长辣椒蔬菜包邮尖椒批发特产5斤', 29.9, 86, '王小二旗舰店', '湖北 襄阳', 33, 25);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (11, '小黄瓜水果黄瓜新鲜5斤小青瓜东北旱海阳白玉生吃10山东农家蔬菜', 13.8, 7500, '田园茂旗舰店', '山东 烟台', 80081, 79562);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (12, '圣女果千禧小番茄新鲜西红柿千禧长果5斤农家时令蔬菜包邮水果', 18.8, 10000, '时卉源旗舰店', '河南 郑州', 3014, 2682);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (13, '高端货!新疆沙瓤西红柿新鲜自然熟番茄水果普罗旺斯农家顺丰包邮', 58.8, 232, '奇迹汇食品旗舰店', '新疆 吐鲁番', 91, 63);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (14, '绿宝石甜瓜小香瓜新鲜水果应季10东北瓜果包邮5斤助农当季整箱', 24.8, 15000, '梦强旗舰店', '山东 临沂', 34173, 37618);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (15, '5斤新西兰贝贝南瓜板栗小南瓜板栗味老瓜栗子板粟10农家新鲜带箱', 14.8, 10000, '刘小牛旗舰店', '山东 日照', 18590, 6991);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (16, '【5A特大】仙居杨梅新鲜孕妇水果现摘现发正宗农家东魁杨梅6斤装', 238, 260, '巨浪食品专营店', '浙江 台州', 3775, 8498);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (17, '宜兴新鲜百合2500g包邮宜兴特产百合纯农家食用大白合5斤30个左右', 52, 975, '镓荣旗舰店', '江苏 无锡', 2767, 2348);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (18, '杨梅新鲜正宗仙居东魁孕妇现摘现发农家时令水果6斤装东魁杨梅', 95, 5000, '集果邦旗舰店', '浙江 台州', 385, 10413);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (19, '新鲜自然熟黄金籽西红柿农家水果老品种5斤非铁皮博士番茄沙瓤', 49, 2164, '黄金籽旗舰店', '山东 潍坊', 4250, 13763);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (20, '云南小土豆新鲜10斤马铃薯农产品蔬菜红皮洋芋批发迷你小黄心土豆', 19.8, 10000, '红高粱食品旗舰店', '云南 昆明', 51861, 40936);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (21, '现挖云南紫皮洋葱5斤新鲜红皮大圆葱洋葱头农家自种特产蔬菜包邮', 9.9, 249, '红高粱食品旗舰店', '云南 红河', 268, 212);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (22, '新鲜芋头10斤芋艿小芋头香芋免邮包邮农家粉糯荔浦毛芋头整箱紫', 29.8, 7500, '红高粱食品旗舰店', '山东 潍坊', 49152, 45661);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (23, '云南紫皮洋葱新鲜10斤包邮洋葱头农家自种当季蔬菜红皮大圆葱整箱', 19.8, 8500, '红高粱食品旗舰店', '云南 红河', 5733, 2604);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (24, '盒马河南焦作铁棍山药净重5斤当季农家蔬菜温县新鲜山药包邮', 39.9, 1645, '盒马鲜生旗舰店', '河南 焦作', 1185, 1476);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (25, '盒马山东大葱净重5斤新鲜长葱当季时令蔬菜去叶白香葱产地农产品', 24.9, 49, '盒马鲜生旗舰店', '上海', 36, 47);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (26, '盒马山东玉菇甜瓜2粒装单粒1kg起当季时令水果新鲜甜瓜蜜瓜', 24.9, 566, '盒马鲜生旗舰店', '山东 潍坊', 97, 69);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (27, '【芭芭农场】新疆普罗旺斯西红柿净重5斤当季番茄自然熟水果蔬菜', 39.9, 7500, '盒马鲜生旗舰店', '陕西 西安', 2262, 1697);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (28, '芋头新鲜蔬菜小芋头毛芋头香芋农家自种芋头芋艿非荔浦芋头5斤10', 18.5, 4888, '果鲜萌旗舰店', '山东 潍坊', 14571, 13733);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (29, '农家自种新鲜小米辣椒5斤红辣椒朝天椒蔬菜泡椒特辣小米椒鲜辣椒', 28.8, 4867, '果品康旗舰店', '海南 海口', 9453, 9562);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (30, '2020年新鲜现挖小洋葱带箱10斤 红皮紫皮洋葱头圆葱农家自种蔬菜', 10.8, 15000, '果恋韵旗舰店', '云南 红河', 23575, 15821);
INSERT INTO testdb.product_table (productID, product_name, price, volume, shopp_name, location, evaluate, collect) VALUES (31, '海南新鲜辣椒小米椒5斤朝天小米辣红辣椒农家土蔬菜可剁泡椒包邮', 29.9, 1936, '果绰旗舰店', '海南 海口', 2413, 2962);
```

**(6) 创建`Entity`包，并创建`MemberUser.java`文件**

`MemberUser.java`文件中内容：

```java
//会员信息
public class MemberUser {

    private Integer user_id;//用户id
    private String user_name;//用户名


    public MemberUser() {

    }

    public MemberUser(Integer user_id, String user_name) {
        this.user_id = user_id;
        this.user_name = user_name;
    }

    public Integer getUser_id() {
        return user_id;
    }

    public void setUser_id(Integer user_id) {
        this.user_id = user_id;
    }

    public String getUser_name() {
        return user_name;
    }

    public void setUser_name(String user_name) {
        this.user_name = user_name;
    }

    @Override
    public String toString() {
        return "MemberUser{" +
                "user_id=" + user_id +
                ", user_name='" + user_name + '\'' +
                '}';
    }
}
```

**(7) 在`Entity`包中创建`ProductOrder.java`文件**

``ProductOrder.java`文件中内容：

```java
//商品订单
public class ProductOrder {

    private Integer order_id;//订单id
    private Integer user_id;//所购买的用户id
    private Integer product_id;//商品id
    private Integer gwcount;//购买数量

    public ProductOrder() {
    }

    public ProductOrder(Integer order_id, Integer user_id, Integer product_id, Integer gwcount) {
        this.order_id = order_id;
        this.user_id = user_id;
        this.product_id = product_id;
        this.gwcount = gwcount;
    }

    public Integer getOrder_id() {
        return order_id;
    }

    public void setOrder_id(Integer order_id) {
        this.order_id = order_id;
    }

    public Integer getUser_id() {
        return user_id;
    }

    public void setUser_id(Integer user_id) {
        this.user_id = user_id;
    }

    public Integer getProduct_id() {
        return product_id;
    }

    public void setProduct_id(Integer product_id) {
        this.product_id = product_id;
    }

    public Integer getGwcount() {
        return gwcount;
    }

    public void setGwcount(Integer gwcount) {
        this.gwcount = gwcount;
    }

    @Override
    public String toString() {
        return "ProductOrder{" +
            "order_id=" + order_id +
            ", user_id=" + user_id +
            ", product_id=" + product_id +
            ", gwcount=" + gwcount +
            '}';
    }
}
```

**(8) 在`Entity`包中创建`ProductTable.java`文件**

``ProductTable.java`文件中内容：

```java
/**
 * @Auther: truedei
 * @Date: 2020 /20-6-13 21:05
 * @Description:商品记录表
 */
public class ProductTable {

    private Integer productID   ; //商品ID'
    private String product_name; //ll comment '商品名字'
    private Double price       ; //商品金额'
    private Integer volume      ; //成交数量'
    private String shopp_name  ; //ll comment '商店名称'
    private String location    ; //ll comment '生产地'
    private Integer evaluate    ; //好评数量'
    private Integer collect     ; //收藏数量'

    public ProductTable(Integer productID, String product_name, Double price, Integer volume, String shopp_name, String location, Integer evaluate, Integer collect) {
        this.productID = productID;
        this.product_name = product_name;
        this.price = price;
        this.volume = volume;
        this.shopp_name = shopp_name;
        this.location = location;
        this.evaluate = evaluate;
        this.collect = collect;
    }

    public ProductTable() {
    }

    public Integer getProductID() {
        return productID;
    }

    public void setProductID(Integer productID) {
        this.productID = productID;
    }

    public String getProduct_name() {
        return product_name;
    }

    public void setProduct_name(String product_name) {
        this.product_name = product_name;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public Integer getVolume() {
        return volume;
    }

    public void setVolume(Integer volume) {
        this.volume = volume;
    }

    public String getShopp_name() {
        return shopp_name;
    }

    public void setShopp_name(String shopp_name) {
        this.shopp_name = shopp_name;
    }

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    public Integer getEvaluate() {
        return evaluate;
    }

    public void setEvaluate(Integer evaluate) {
        this.evaluate = evaluate;
    }

    public Integer getCollect() {
        return collect;
    }

    public void setCollect(Integer collect) {
        this.collect = collect;
    }

    @Override
    public String toString() {
        return "ProductTable{" +
                "productID=" + productID +
                ", product_name='" + product_name + '\'' +
                ", price=" + price +
                ", volume=" + volume +
                ", shopp_name='" + shopp_name + '\'' +
                ", location='" + location + '\'' +
                ", evaluate=" + evaluate +
                ", collect=" + collect +
                '}';
    }
}
```

**(9) 在`Entity`包中创建`UserR.java`文件**

`UserR.java`文件中内容：

```java
import java.util.Arrays;

/**
 * @Auther: truedei
 * @Date: 2020 /20-6-13 22:53
 * @Description:
 */
public class UserR {

    private String userName;

    private Integer userId;

    private Integer[] ProductIds;

    private Double cos_th;

    public Double getCos_th() {
        return cos_th;
    }

    public void setCos_th(Double cos_th) {
        this.cos_th = cos_th;
    }


    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Integer[] getProductIds() {
        return ProductIds;
    }

    public void setProductIds(Integer[] productIds) {
        ProductIds = productIds;
    }

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    @Override
    public String toString() {
        return "UserR{" +
                "userName='" + userName + '\'' +
                ", userId=" + userId +
                ", ProductIds=" + Arrays.toString(ProductIds) +
                ", cos_th=" + cos_th +
                '}';
    }
}
```

**(10) 创建`Db`包，并创建`DBHelp.java`文件**

`DBHelp.java`文件中内容：

```java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import Entity.*;

public class DBHelp {

    static String url = "jdbc:mysql://127.0.0.1:3358/testdb?useUnicode=true&characterEncoding=UTF-8";
    static String user= "mysql";
    static String password= "123456";

    static {
        try {
            Class.forName("com.mysql.jdbc.Driver");

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() {
        try {
            return DriverManager.getConnection(url, user, password);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }


    static Connection conn = DBHelp.getConnection();
    static Statement st = null;
    static ResultSet rs = null;

    /**
     * 获取所有的商品信息
     * @return
     * @param sqlId
     */
    public static List<ProductTable> getProductList(String sqlId){

        List<ProductTable> productTables = new ArrayList<>();

        try {
            st = conn.createStatement();
            rs = st.executeQuery("select * from product_table where productID in ("+sqlId+")");

            while (rs.next()){
                productTables.add(new ProductTable(
                    rs.getInt("productID"),
                    rs.getString("product_name"),
                    rs.getDouble("price"),
                    rs.getInt("volume"),
                    rs.getString("shopp_name"),
                    rs.getString("location"),
                    rs.getInt("evaluate"),
                    rs.getInt("collect")));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }

        return productTables;
    }

    //获取用户订单信息
    public static List<ProductOrder> getProductOrderList(Integer userId){
        List<ProductOrder> productTables = new ArrayList<>();

        //        String sql = "select * from product_order where USER_ID=(select USER_ID from member_user where USER_NAME=\""+name+"\")";

        String sql = "select * from product_order "+(userId==null?"":"where USER_ID="+userId);
        //        System.out.println("执行的 sql: "+sql);
        try {
            st = conn.createStatement();
            rs = st.executeQuery(sql);

            while (rs.next()){
                productTables.add(new ProductOrder(
                    rs.getInt("order_id"),
                    rs.getInt("user_id"),
                    rs.getInt("product_id"),
                    rs.getInt("gwcount")));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }

        return productTables;
    }

    //获取用户信息
    public static List<MemberUser> getMemberUserList(){
        List<MemberUser> productTables = new ArrayList<>();

        try {
            st = conn.createStatement();
            rs = st.executeQuery("select * from member_user");

            while (rs.next()){
                productTables.add(new MemberUser(
                    rs.getInt("user_id"),
                    rs.getString("user_name")));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }

        return productTables;
    }
}
```

**(11) 创建`test`包，并创建`ArrayUtil.java`文件**

`ArrayUtil.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class ArrayUtil {
    /**
     * 求并集
     *
     * @param m
     * @param n
     * @return
     */
    public static Integer[] getB(Integer[] m, Integer[] n)
    {
        // 将数组转换为set集合
        Set<Integer> set1 = new HashSet<Integer>(Arrays.asList(m));
        Set<Integer> set2 = new HashSet<Integer>(Arrays.asList(n));

        // 合并两个集合
        set1.addAll(set2);

        Integer[] arr = {};
        return set1.toArray(arr);
    }

    /**
     * 求交集
     *
     * @param m
     * @param n
     * @return
     */
    public static Integer[] getJ(Integer[] m, Integer[] n)
    {
        List<Integer> rs = new ArrayList<Integer>();
        // 将较长的数组转换为set
        Set<Integer> set = new HashSet<Integer>(Arrays.asList(m.length > n.length ? m : n));

        // 遍历较短的数组，实现最少循环
        for (Integer i : m.length > n.length ? n : m)
        {
            if (set.contains(i))
            {
                rs.add(i);
            }
        }
        Integer[] arr = {};
        return rs.toArray(arr);
    }



    /**
     * 求差集
     *
     * @param m
     * @param n
     * @return
     */
    public static Integer[] getC(Integer[] m, Integer[] n)
    {
        // 将较长的数组转换为set
        Set<Integer> set = new HashSet<Integer>(Arrays.asList(m.length > n.length ? m : n));

        // 遍历较短的数组，实现最少循环
        for (Integer i : m.length > n.length ? n : m)
        {
            // 如果集合里有相同的就删掉，如果没有就将值添加到集合
            if (set.contains(i))
            {
                set.remove(i);
            } else
            {
                set.add(i);
            }
        }
        Integer[] arr = {};
        return set.toArray(arr);
    }
}
```

**(12) 在`test`包中创建`MapSortUtil.java`文件**

`MapSortUtil.java`文件中内容：

```java
import java.util.Comparator;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.stream.Collectors;

public class MapSortUtil {

    private static Comparator<Map.Entry> comparatorByKeyAsc = (Map.Entry o1, Map.Entry o2) -> {
        if (o1.getKey() instanceof Comparable) {
            return ((Comparable) o1.getKey()).compareTo(o2.getKey());
        }
        throw new UnsupportedOperationException("键的类型尚未实现Comparable接口");
    };


    private static Comparator<Map.Entry> comparatorByKeyDesc = (Map.Entry o1, Map.Entry o2) -> {
        if (o1.getKey() instanceof Comparable) {
            return ((Comparable) o2.getKey()).compareTo(o1.getKey());
        }
        throw new UnsupportedOperationException("键的类型尚未实现Comparable接口");
    };


    private static Comparator<Map.Entry> comparatorByValueAsc = (Map.Entry o1, Map.Entry o2) -> {
        if (o1.getValue() instanceof Comparable) {
            return ((Comparable) o1.getValue()).compareTo(o2.getValue());
        }
        throw new UnsupportedOperationException("值的类型尚未实现Comparable接口");
    };


    private static Comparator<Map.Entry> comparatorByValueDesc = (Map.Entry o1, Map.Entry o2) -> {
        if (o1.getValue() instanceof Comparable) {
            return ((Comparable) o2.getValue()).compareTo(o1.getValue());
        }
        throw new UnsupportedOperationException("值的类型尚未实现Comparable接口");
    };

    /**
     * 按键升序排列
     */
    public static <K, V> Map<K, V> sortByKeyAsc(Map<K, V> originMap) {
        if (originMap == null) {
            return null;
        }
        return sort(originMap, comparatorByKeyAsc);
    }

    /**
     * 按键降序排列
     */
    public static <K, V> Map<K, V> sortByKeyDesc(Map<K, V> originMap) {
        if (originMap == null) {
            return null;
        }
        return sort(originMap, comparatorByKeyDesc);
    }


    /**
     * 按值升序排列
     */
    public static <K, V> Map<K, V> sortByValueAsc(Map<K, V> originMap) {
        if (originMap == null) {
            return null;
        }
        return sort(originMap, comparatorByValueAsc);
    }

    /**
     * 按值降序排列
     */
    public static <K, V> Map<K, V> sortByValueDesc(Map<K, V> originMap) {
        if (originMap == null) {
            return null;
        }
        return sort(originMap, comparatorByValueDesc);
    }

    private static <K, V> Map<K, V> sort(Map<K, V> originMap, Comparator<Map.Entry> comparator) {
        return originMap.entrySet()
                .stream()
                .sorted(comparator)
                .collect(
                        Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, (e1, e2) -> e2,
                                LinkedHashMap::new));
    }

}
```

**(13) 在`test`包中创建`RecommenderSystem.java`文件**

`RecommenderSystem.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import Db.*;
import Entity.*;

public class RecommenderSystem {

    RecommenderSystem(){
        login(1);
    }

    //推荐算法开始

    /**
     * 登录后推荐接口
     * @param userId 模拟登录的用户ID
     */
    public void login(Integer userId){

        //1，使用该用户的名字获取订单信息
        System.out.println("----------------");
        //查询登录用户的订单信息
        List<ProductOrder> productOrderList = DBHelp.getProductOrderList(userId);
        //存储个人 购买的所有的商品id
        Integer[] ints = new Integer[productOrderList.size()];
        //存储个人信息，封装成对象，方便计算
        UserR userR = new UserR();

        //筛选出来个人订单中的商品的id
        System.out.println("个人的：");
        for (int i = 0; i < productOrderList.size(); i++) {
            ints[i] = productOrderList.get(i).getProduct_id();
            System.out.println(productOrderList.get(i).toString());
        }
        userR.setUserId(productOrderList.get(0).getUser_id());
        userR.setProductIds(ints);

        //2,拿到所有用户的订单信息
        List<ProductOrder> productOrderLists = DBHelp.getProductOrderList(null);
        //存储所有人的订单信息
        List<UserR> userRS = new ArrayList<>();
        //利用map的机制，计算出来其余用户的所有的购买商品的id  Map<用户id，商品ID拼接的字符串(1,2,3,4)>
        Map<Integer,String> map = new HashMap<>();
        System.out.println("所有人的：");
        //筛选出来订单中的商品的id
        for (int i = 0; i < productOrderLists.size(); i++) {
            System.out.println(productOrderLists.get(i).toString());
            map.put(productOrderLists.get(i).getUser_id(),
                    map.containsKey(productOrderLists.get(i).getUser_id())?
                    map.get(productOrderLists.get(i).getUser_id())+","+productOrderLists.get(i).getProduct_id():
                    productOrderLists.get(i).getProduct_id()+"");
        }

        //开始封装每个人的数据
        for (Integer key:map.keySet() ) {
            //new出来一个新的个人的对象，后面要塞到list中
            UserR userR2 = new UserR();
            //把其他每个人购买的商品的id 分割成数组
            String[] split = map.get(key).split(",");
            //转换成int数组 进行存储，方便后期计算
            Integer[] ints1 = new Integer[split.length];
            for (int i = 0; i < split.length; i++) {
                ints1[i] = Integer.valueOf(split[i]);
            }
            //用户id 就是key
            userR2.setUserId(key);
            //用户购买的商品id的数组
            userR2.setProductIds(ints1);

            //塞到list中
            userRS.add(userR2);
        }

        //二值化 处理数据
        List<UserR> userRList = jisuan(userR, userRS);

        System.out.println("得出的结果：");
        for (int i = 0; i < userRList.size(); i++) {
            System.out.println(userRList.get(i).toString());
        }

        System.out.println("过滤处理数据之后：");
        //过滤处理
        String sqlId = chuli(userRList, userR);

        System.out.println("推荐的商品：");
        //通过拿到的拼接的被推荐商品的id，去查数据库
        List<ProductTable> productList = DBHelp.getProductList(sqlId);
        //最终拿到被推荐商品的信息
        for (int i = 0; i < productList.size(); i++) {
            System.out.println(productList.get(i).toString());
        }

    }

    /**
     * 过滤处理
     * @param userRList 所有用户的订单数据
     * @param userR 当前登录用户的订单数据
     * @return
     */
    private String chuli(List<UserR> userRList,UserR userR) {

        //为了方便下面过滤数据，预先把登录用户的订单购物的商品的id做一个map，在过滤的时候，只需要查一下map中是否存在key就ok
        Map<Integer,Integer> map1 = new HashMap<>();
        for (int i = 0; i < userR.getProductIds().length; i++) {
            map1.put(userR.getProductIds()[i],userR.getProductIds()[i]);
        }


        //盛放最终过滤出来的数据 Map<商品id,出现的次数>
        Map<Integer,Integer> map = new HashMap<>();

        for (int i = 0; i < userRList.size(); i++) {
            //userRList.get(i).getCos_th()>0：过滤掉相似度等于0，也就是完全不匹配的
            //userRList.get(i).getUserId()!=userR.getUserId()：过滤掉当前用户的订单信息
            if(userRList.get(i).getCos_th()>0 && userRList.get(i).getUserId()!=userR.getUserId()){
                //求当前登录用户的购买商品的id和其他用户的所购买商品的差集，例如：A=[1, 2],B=[1, 2, 3]  那么这个3就是最终想要的结果
                Integer[] j = ArrayUtil.getC(userRList.get(i).getProductIds(), userR.getProductIds());

                //遍历求差集之后的结果
                for (int i1 = 0; i1 < j.length; i1++) {
                    //如果其余的用户所购买撒谎那个品的id不在当前用的所购买商品的id，那么就存起来
                    if(!map1.containsKey(j[i1])){
                        //存储时，数量每次都+1，方便后面排序，出现的次数多，说明被推荐的机会越高
                        map.put(j[i1],map.containsKey(j[i1])?(map.get(j[i1])+1):1);
                    }
                }
            }
        }


        System.out.println("处理之后的map：");
        for (Integer key:map.keySet()) {
            System.out.println("商品id="+key+"--用户所购数量="+map.get(key));
        }

        //把map进行降序排序
        Map<Integer, Integer> map2 = MapSortUtil.sortByKeyDesc(map);
        System.out.println("按降序" + map2);


        //拼接成一个sql，方便去查数据库
        String sqlId = "";
        for (Integer key:map2.keySet()) {
            sqlId = sqlId+key +",";
        }

        sqlId = sqlId.substring(0,sqlId.length()-1);

        System.out.println("最终拿到的被推荐给当前用户的商品id--->"+sqlId);

        return sqlId;
    }

    /**
     * 二值化 处理数据
     * @param userR 当前登录用户的订单信息
     * @param userRS 其他用户的订单信息
     * @return 二值化处理之后的结果
     */
    private List<UserR> jisuan(UserR userR, List<UserR> userRS) {

        //对个人做二值化处理，为了好计算 [0,0,0,0,0,1,1,0,1]这种
        //个人的
        int userErzhihua[] = new int[100];
        System.out.println(userR.getProductIds().length);
        for (int i = 0; i < userR.getProductIds().length; i++) {
            userErzhihua[userR.getProductIds()[i]]=1;
        }
        //库里所有人的
        int erzhihua[] = new int[100];
        //对其他人，做二值化处理，为了好计算 [0,0,0,0,0,1,1,0,1]这种
        for (int i = 0; i < userRS.size(); i++) {
            UserR product = userRS.get(i);
            for (int j = 0; j < product.getProductIds().length; j++) {
                erzhihua[product.getProductIds()[j]]=1;
            }
            //计算当前登录用户与其余每个人的余弦值 cos_th
            Double compare = compare(erzhihua,userErzhihua);
            product.setCos_th(compare);
            //把计算好的值，重新塞到原来的位置，替换到旧的数据
            userRS.set(i,product);

            //防止数组中的值重复，起到清空的作用
            erzhihua = new int[100];
        }

        return userRS;

    }

    /**
     * 代码核心内容
     * @param o1 当前登录用户的
     * @param o2 其他用户的 n1 n2 n3 n4 n....
     * @return
     */
    private static Double compare(int[] o1, int[] o2) {
        //分子求和
        Double fenzi = 0.0 ;

        for (int i = 0; i < o1.length; i++) {
            fenzi += o1[i]*o2[i];
        }
        //分母第一部分
        Double fenmu1 = 0.0;
        for (int i = 0; i < o1.length; i++) {
            fenmu1 += o1[i] * o1[i];
        }
        fenmu1 = Math.sqrt(fenmu1);
        //分母第二部分
        Double fenmu2 = 0.0;
        for (int i = 0; i < o2.length; i++) {
            fenmu2 += o2[i] * o2[i];
        }
        fenmu2 = Math.sqrt(fenmu2);
        return fenzi / (fenmu1 * fenmu2);
    }
}
```

**(14) 在`test`包中创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        new RecommenderSystem();
    }
}
```

***

##### <font size="4" color="red">06. Java中拆解协议包</font>

```java
import java.util.ArrayList;
import java.util.List;

public class test {

    private static int startBit = 0x7e;
    private static int endBit   = 0x7e;
    //排除包头+包尾+协议号+长度
    private static int limit = 20;

    public static List<String> splitPackage(String hex_str,boolean repeat){
        ArrayList<String> pac_list = null;
        ArrayList<Integer> start_index = new ArrayList<Integer>();
        ArrayList<Integer> end_index = new ArrayList<Integer>();
        //遍历截取值
        String pos_val = null;
        int len = hex_str.length();
        int hex_item = 0;
        //遍历字符串
        for(int i=0;i<len;i+=2){
            pos_val = hex_str.substring(i,i+2);
            hex_item = Integer.parseInt(pos_val,16);
            if(startBit == hex_item) {
                start_index.add(i);
            }
            if(endBit == hex_item) {
                end_index.add(i);
            }
        }
        //索引长度
        int start_len = start_index.size();
        int end_len = end_index.size();
        if(!start_index.isEmpty() && !end_index.isEmpty()) {
            //初始化
            pac_list = new ArrayList<String>();
            for(int i=0;i<start_len;i++) {
                for(int j=0;j<end_len;j++) {
                    if(i > 0) {
                        int start_a = start_index.get(i-1);
                        int start_b = start_index.get(i);
                        //排除包头重复数字例如0x7878
                        if(repeat) {
                            if(start_b - start_a == 2) {
                                int start_pos = start_index.get(i - 1);
                                int end_pos = end_index.get(j);
                                //过滤非正常结尾数据包7e
                                if(end_pos - start_pos > limit * 2) {
                                    pos_val = hex_str.substring(start_pos,end_pos + 2);
                                    pac_list.add(pos_val);
                                    break;
                                }
                            }
                        }
                        else {
                            if(start_b - start_a != 2) {
                                int start_pos = start_index.get(i - 1);
                                int end_pos = end_index.get(j);
                                //过滤非正常结尾数据包
                                if(end_pos - start_pos > limit * 2) {
                                    pos_val = hex_str.substring(start_pos,end_pos + 2);
                                    pac_list.add(pos_val);
                                    break;
                                }
                            }
                        }
                    }
                }
            }
            start_index.clear();
            end_index.clear();
        }
        return pac_list;
    } 
    public static void main(String[] args) throws Exception {
        String hex_str = "";
        List<String> arr = splitPackage(hex_str,false);
    }
}
```

****

##### <font size="4" color="red">07. Java中SnowFlower自增Id(雪花算法)</font>

**(1) 创建`SnowFlower.java`文件**

`Snowflower.java`文件中内容：

```java
public class SnowFlake {
    // 起始的时间戳
    private final static long START_STMP = 1577808000000L; //2020-01-01
    // 每一部分占用的位数，就三个
    private final static long SEQUENCE_BIT = 12; //序列号占用的位数
    private final static long MACHINE_BIT = 5; //机器标识占用的位数
    private final static long DATACENTER_BIT = 5; //数据中心占用的位数
    // 每一部分最大值
    private final static long MAX_DATACENTER_NUM = -1L ^ (-1L << DATACENTER_BIT);
    private final static long MAX_MACHINE_NUM = -1L ^ (-1L << MACHINE_BIT);
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);
    // 每一部分向左的位移
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATACENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTMP_LEFT = DATACENTER_LEFT + DATACENTER_BIT;
    private long datacenterId; //数据中心
    private long machineId; //机器标识
    private long sequence = 0L; //序列号
    private long lastStmp = -1L; //上一次时间戳

    public SnowFlake(long datacenterId, long machineId) {
        if (datacenterId > MAX_DATACENTER_NUM || datacenterId < 0) {
            throw new IllegalArgumentException("datacenterId can't be greater than MAX_DATACENTER_NUM or less than 0");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("machineId can't be greater than MAX_MACHINE_NUM or less than 0");
        }
        this.datacenterId = datacenterId;
        this.machineId = machineId;
    }

    //产生下一个ID
    public synchronized long nextId() {
        long currStmp = timeGen();
        if (currStmp < lastStmp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currStmp == lastStmp) {
            //if条件里表示当前调用和上一次调用落在了相同毫秒内，只能通过第三部分，序列号自增来判断为唯一，所以+1.
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大，只能等待下一个毫秒
            if (sequence == 0L) {
                currStmp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            //执行到这个分支的前提是currTimestamp > lastTimestamp，说明本次调用跟上次调用对比，已经不再同一个毫秒内了，这个时候序号可以重新回置0了。
            sequence = 0L;
        }

        lastStmp = currStmp;
        //就是用相对毫秒数、机器ID和自增序号拼接
        return (currStmp - START_STMP) << TIMESTMP_LEFT //时间戳部分
            | datacenterId << DATACENTER_LEFT       //数据中心部分
            | machineId << MACHINE_LEFT             //机器标识部分
            | sequence;                             //序列号部分
    }

    private long getNextMill() {
        long mill = timeGen();
        while (mill <= lastStmp) {
            mill = timeGen();
        }
        return mill;
    }

    private long timeGen() {
        return System.currentTimeMillis();
    }
}
```

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        SnowFlake idWorker = new SnowFlake(0, 0);

        for (int i = 0; i < 100; i++) {
            long id = idWorker.nextId();
            System.out.println(Long.toBinaryString(id));
            System.out.println(id);
        }
    }
}
```

***

##### <font size="4" color="red">08. Java中amr录音文件转mp3文件</font>

**1.`ffmpeg`格式转换**

**(1) 安装`ffmpeg`并设置成环境变量**

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
import java.io.File;

public class test {

    public static void ToMp3(String sourcePath){  
        File file = new File(sourcePath); 
        //转换后文件的存储地址，直接将原来的文件名后加mp3后缀名 
        String name = file.getName();
        name = name.substring(0,name.lastIndexOf("."));
        String targetPath = name +".mp3";
        System.out.println(sourcePath);
        Runtime run = null;    
        try {    
            run = Runtime.getRuntime();    
            long start=System.currentTimeMillis();  
            //执行ffmpeg.exe,前面是ffmpeg.exe的地址，中间是需要转换的文件地址，后面是转换后的文件地址。-i是转换方式，意思是可编码解码，mp3编码方式采用的是libmp3lame
            Process p=run.exec("ffmpeg -i "+ sourcePath +" -acodec libmp3lame "+ targetPath);
            //释放进程    
            p.getOutputStream().close();    
            p.getInputStream().close();    
            p.getErrorStream().close();    
            p.waitFor();    
            long end=System.currentTimeMillis();    
            System.out.println(sourcePath+" convert success, costs:"+(end-start)+"ms");    
            //删除原来的文件    
            if(file.exists()){    
                file.delete();    
            }    
        } catch (Exception e) {    
            e.printStackTrace();    
        }finally{    
            //run调用lame×××最后释放内存    
            run.freeMemory();    
        }  
    }

    public static void main(String[] args) throws Exception {
        ToMp3("test.amr");
    }
}
```

**2.依赖包转换**

**(1) 依赖包**

```xml
<dependency>
    <groupId>com.github.dadiyang</groupId>
    <artifactId>jave</artifactId>
    <version>1.0.3</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-nop</artifactId>
    <version>1.7.25</version>
</dependency>
```

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
import java.io.File;
import it.sauronsoftware.jave.AudioUtils;

public class test {

    public static void main(String[] args) throws Exception {
        File source = new File("test.amr");
        File target = new File("testAudio.mp3");
        AudioUtils.amrToMp3(source, target);
    }
}
```

***

##### <font size="4" color="red">09. Java中人民币大写转换</font>

**(1) 创建`MoneyUtil.java`文件**

`MoneyUtil.java`文件中内容：

```java
public class MoneyUtil {

    /** 大写数字 */  
    private static final String[] NUMBERS = { "零", "壹", "贰", "叁", "肆", "伍",  
                                             "陆", "柒", "捌", "玖" };  

    /** 整数部分的单位 */  
    private static final String[] IUNIT = { "元", "拾", "佰", "仟", "万", "拾", "佰",  
                                           "仟", "亿", "拾", "佰", "仟", "万", "拾", "佰", "仟" };  

    /** 小数部分的单位 */  
    private static final String[] DUNIT = { "角", "分", "厘" };  

    /** 
     * 得到大写金额。 
     */  
    public static String toChinese(String str) {  
        str = str.replaceAll(",", "");// 去掉","  
        String integerStr;// 整数部分数字  
        String decimalStr;// 小数部分数字  
        // 初始化：分离整数部分和小数部分  
        if (str.indexOf(".") > 0) {  
            integerStr = str.substring(0, str.indexOf("."));  
            decimalStr = str.substring(str.indexOf(".") + 1);  
        } else if (str.indexOf(".") == 0) {  
            integerStr = "";  
            decimalStr = str.substring(1);  
        } else {  
            integerStr = str;  
            decimalStr = "";  
        }  
        // integerStr去掉首0，不必去掉decimalStr的尾0(超出部分舍去)  
        if (!integerStr.equals("")) {  
            integerStr = Long.toString(Long.parseLong(integerStr));  
            if (integerStr.equals("0")) {  
                integerStr = "";  
            }  
        }  
        // overflow超出处理能力，直接返回  
        if (integerStr.length() > IUNIT.length) {  
            System.out.println(str + ":超出处理能力");  
            return str;  
        }  

        int[] integers = toArray(integerStr);// 整数部分数字  
        boolean isMust5 = isMust5(integerStr);// 设置万单位  
        int[] decimals = toArray(decimalStr);// 小数部分数字  
        return getChineseInteger(integers, isMust5)  
            + getChineseDecimal(decimals);  
    }  

    /** 
     * 整数部分和小数部分转换为数组，从高位至低位 
     */  
    private static int[] toArray(String number) {  
        int[] array = new int[number.length()];  
        for (int i = 0; i < number.length(); i++) {  
            array[i] = Integer.parseInt(number.substring(i, i + 1));  
        }  
        return array;  
    }  

    /** 
     * 得到中文金额的整数部分。 
     */ 
    private static String getChineseInteger(int[] integers, boolean isMust5) {  
        StringBuffer chineseInteger = new StringBuffer("");  
        int length = integers.length;  
        for (int i = 0; i < length; i++) {  
            // 0出现在关键位置：1234(万)5678(亿)9012(万)3456(元)  
            // 特殊情况：10(拾元、壹拾元、壹拾万元、拾万元)  
            String key = "";  
            if (integers[i] == 0) {  
                if ((length - i) == 13)// 万(亿)(必填)  
                    key = IUNIT[4];  
                else if ((length - i) == 9)// 亿(必填)  
                    key = IUNIT[8];  
                else if ((length - i) == 5 && isMust5)// 万(不必填)  
                    key = IUNIT[4];  
                else if ((length - i) == 1)// 元(必填)  
                    key = IUNIT[0];  
                // 0遇非0时补零，不包含最后一位  
                if ((length - i) > 1 && integers[i + 1] != 0)  
                    key += NUMBERS[0];  
            }  
            chineseInteger.append(integers[i] == 0 ? key  
                                  : (NUMBERS[integers[i]] + IUNIT[length - i - 1]));  
        }  
        return chineseInteger.toString();  
    }  

    /** 
     * 得到中文金额的小数部分。 
     */  
    private static String getChineseDecimal(int[] decimals) {  
        StringBuffer chineseDecimal = new StringBuffer("");  
        for (int i = 0; i < decimals.length; i++) {  
            // 舍去3位小数之后的  
            if (i == 3)  
                break;  
            chineseDecimal.append(decimals[i] == 0 ? ""  
                                  : (NUMBERS[decimals[i]] + DUNIT[i]));  
        }  
        return chineseDecimal.toString();  
    }  

    /** 
     * 判断第5位数字的单位"万"是否应加。 
     */  
    private static boolean isMust5(String integerStr) {  
        int length = integerStr.length();  
        if (length > 4) {  
            String subInteger = "";  
            if (length > 8) {  
                // 取得从低位数，第5到第8位的字串  
                subInteger = integerStr.substring(length - 8, length - 4);  
            } else {  
                subInteger = integerStr.substring(0, length - 4);  
            }  
            return Integer.parseInt(subInteger) > 0;  
        } else {  
            return false;  
        }  
    }
}
```

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        System.out.println(MoneyUtil.toChinese("5000.23"));  
    }
}
```

***

##### <font size="4" color="red">10. Java中音频录制</font>

**(1) 创建`EngineeCore.java`文件**

`EngineeCore.java`文件中内容：

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.IOException;
import javax.sound.sampled.AudioFileFormat;
import javax.sound.sampled.AudioFormat;
import javax.sound.sampled.AudioInputStream;
import javax.sound.sampled.AudioSystem;
import javax.sound.sampled.DataLine;
import javax.sound.sampled.TargetDataLine;

public class EngineeCore {

    private String filePath;

    public EngineeCore(String filePath) {
        this.filePath = filePath;
    }

    AudioFormat audioFormat;
    TargetDataLine targetDataLine;
    boolean flag = true;


    private void stopRecognize() {
        flag = false;
        targetDataLine.stop();
        targetDataLine.close();
    }private AudioFormat getAudioFormat() {
        float sampleRate = 16000;
        // 8000,11025,16000,22050,44100
        int sampleSizeInBits = 16;
        // 8,16
        int channels = 1;
        // 1,2
        boolean signed = true;
        // true,false
        boolean bigEndian = false;
        // true,false
        return new AudioFormat(sampleRate, sampleSizeInBits, channels, signed, bigEndian);
    }// end getAudioFormat


    public void startRecognize() {
        try {
            // 获得指定的音频格式
            audioFormat = getAudioFormat();
            DataLine.Info dataLineInfo = new DataLine.Info(TargetDataLine.class, audioFormat);
            targetDataLine = (TargetDataLine) AudioSystem.getLine(dataLineInfo);

            // Create a thread to capture the microphone
            // data into an audio file and start the
            // thread running. It will run until the
            // Stop button is clicked. This method
            // will return after starting the thread.
            flag = true;
            new CaptureThread().start();
        } catch (Exception e) {
            e.printStackTrace();
        } // end catch
    }// end captureAudio method

    class CaptureThread extends Thread {

        public void run() {
            AudioFileFormat.Type fileType = null;
            File audioFile = new File(filePath);
            fileType = AudioFileFormat.Type.WAVE;
            //声音录入的权值
            int weight = 2;
            //判断是否停止的计数
            int downSum = 0;
            ByteArrayInputStream bais = null;
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            AudioInputStream ais = null;
            try {
                targetDataLine.open(audioFormat);
                targetDataLine.start();
                byte[] fragment = new byte[1024];

                ais = new AudioInputStream(targetDataLine);
                while (flag) {

                    targetDataLine.read(fragment, 0, fragment.length);
                    //当数组末位大于weight时开始存储字节（有声音传入），一旦开始不再需要判断末位
                    if (Math.abs(fragment[fragment.length-1]) > weight || baos.size() > 0) {
                        baos.write(fragment);
                        System.out.println("守卫："+fragment[0]+",末尾："+fragment[fragment.length-1]+",lenght"+fragment.length);
                        //判断语音是否停止
                        if(Math.abs(fragment[fragment.length-1])<=weight){
                            downSum++;
                        }else{
                            System.out.println("重置奇数");
                            downSum=0;
                        }
                        //计数超过20说明此段时间没有声音传入(值也可更改)
                        if(downSum>20){
                            System.out.println("停止录入");
                            break;
                        }

                    }
                }

                //取得录音输入流
                audioFormat = getAudioFormat();
                byte audioData[] = baos.toByteArray();
                bais = new ByteArrayInputStream(audioData);
                ais = new AudioInputStream(bais, audioFormat, audioData.length / audioFormat.getFrameSize());
                //定义最终保存的文件名
                System.out.println("开始生成语音文件");
                AudioSystem.write(ais, fileType, audioFile);
                downSum = 0;
                stopRecognize();

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                //关闭流
                try {
                    ais.close();
                    bais.close();
                    baos.reset();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}
```

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        String filePath = "D://test.wav";
        EngineeCore engineeCore = new EngineeCore(filePath);
        engineeCore.startRecognize();
    }
}
```

***

##### <font size="4" color="red">11. Java中播放Wav文件</font>

**(1) 创建`WavUtil.java`文件**

`WavUtil.java`文件中内容：

```java
import java.io.File;
import javax.sound.sampled.AudioFormat;
import javax.sound.sampled.AudioInputStream;
import javax.sound.sampled.AudioSystem;
import javax.sound.sampled.DataLine;
import javax.sound.sampled.SourceDataLine;

public class WavUtil {

    private static AudioFormat audioFormat = null;
    private static SourceDataLine sourceDataLine = null;
    private static DataLine.Info dataLine_info = null;
    private static AudioInputStream audioInputStream = null;

    public static void play(String file) throws Exception {
        audioInputStream = AudioSystem.getAudioInputStream(new File(file));
        //audioInputStream=AudioSystem.getAudioInputStream(new URL(file));
        audioFormat = audioInputStream.getFormat();
        System.out.println("每秒播放帧数："+audioFormat.getSampleRate());
        System.out.println("总帧数："+audioInputStream.getFrameLength());
        System.out.println("音频时长（秒）："+audioInputStream.getFrameLength()/audioFormat.getSampleRate());
        dataLine_info = new DataLine.Info(SourceDataLine.class, audioFormat);
        sourceDataLine = (SourceDataLine) AudioSystem.getLine(dataLine_info);
        byte[] b = new byte[1024];
        int len = 0;
        sourceDataLine.open(audioFormat, 1024);
        sourceDataLine.start();
        while ((len = audioInputStream.read(b)) > 0) {
            sourceDataLine.write(b, 0, len);
        }
        audioInputStream.close();
        sourceDataLine.drain();
        sourceDataLine.close();
    }

}
```

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        WavUtil.play("test.wav");
    }
}
```

***

##### <font size="4" color="red">12. Java中读取wav音频为base64字符串</font>

```java
import java.io.File;
import java.io.FileInputStream;
import javax.xml.bind.DatatypeConverter;

public class test {

    public static byte[] readFile(String url) throws Exception {
        File file = new File(url);
        byte[] buff = new byte[(int)file.length()];
        FileInputStream epo = new FileInputStream(file);
        epo.read(buff);
        epo.close();
        return buff;
    }

    public static String byteToBase64(byte[] buff) {
        String baseStr = DatatypeConverter.printBase64Binary(buff);
        return baseStr;
    }

    public static void main(String[] args) throws Exception {
        byte[] buf = readFile("test.wav");
        String str = byteToBase64(buf);
        System.out.println(str.length());
    }
}
```

***

##### <font size="4" color="red">13. Java中读写文件byte内容</font>

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class test {

    public static byte[] readFile(String url) throws Exception {
        File file = new File(url);
        byte[] buff = new byte[(int)file.length()];
        FileInputStream epo = new FileInputStream(file);
        epo.read(buff);
        epo.close();
        return buff;
    }

    private static void fileByteWrite(File file,byte[] file_con) {
        //写入对象初始化
        FileOutputStream fos = null;
        try {
            //写入对象初始化
            fos = new FileOutputStream(file,true);
            //写入文件
            fos.write(file_con);
            //释放对象
            fos.close();
            fos = null;
        }
        catch (IOException e) {
            //释放对象
            fos = null;
        }
    } 

    public static void main(String[] args) throws Exception {
        byte[] buf = readFile("test.wav");
    }
}
```

***

##### <font size="4" color="red">14. Java中Base64字符串转换</font>

```java
import javax.xml.bind.DatatypeConverter;

public class test {

    public static void main(String[] args) throws Exception {
        byte[] buff = "hello".getBytes();
        //字符串转base64
        String baseStr = DatatypeConverter.printBase64Binary(buff);
        System.out.println(baseStr);
        //base64转字符串
        byte[]  bufarray = DatatypeConverter.parseBase64Binary(baseStr);
        System.out.println(new String(bufarray));
    }
}
```

***

##### <font size="4" color="red">15. Java中十六进制字符串和byte转换函数</font>

```java
//十六进制编码数组
private final char[] hex_char = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e','f' };

public String Byte_To_Hex(byte[] bytes) {
    char[] buf = new char[bytes.length * 2];
    int index = 0;
    byte[] arrayOfByte = bytes;
    int j = bytes.length;
    for (int i = 0; i < j; i++) {
        byte b = arrayOfByte[i];
        buf[(index++)] = hex_char[(b >>> 4 & 0xF)];
        buf[(index++)] = hex_char[(b & 0xF)];
    }
    String hexStr = new String(buf);
    //释放对象
    buf = null;
    bytes = null;
    return hexStr;
}

public byte[] Hex_To_Byte(String hex_str) {
    if ((hex_str == null) || (hex_str.trim().equals(""))) {
        return new byte[0];
    }
    byte[] bytes = new byte[hex_str.length() / 2];
    for (int i = 0; i < hex_str.length() / 2; i++) {
        String subhexStr = hex_str.substring(i * 2, i * 2 + 2);
        bytes[i] = ((byte) Integer.parseInt(subhexStr, 16));
        //释放对象
        subhexStr = null;
    }
    //释放对象
    hex_str = null;
    return bytes;
}
```

***

##### <font size="4" color="red">16. Java中Ascall码和字符串互转函数</font>

```java
public String Ascall_To_Str(String ascall_str) {
    StringBuffer str = new StringBuffer();
    for (int i = 0; i < ascall_str.length(); i += 2) {
        String tem_str = ascall_str.substring(i, i + 2);
        char hex = (char) Integer.parseInt(tem_str, 16);
        str.append(hex);
        //释放对象
        tem_str = null;
    }
    String asc_str = str.toString();
    asc_str = asc_str.replace("\u0000", "");
    //释放对象
    str.delete(0, str.length());
    str = null;
    return asc_str;
}

public String Str_To_Ascall(String hex_str) {
    StringBuffer str = new StringBuffer();
    char[] chars = hex_str.toCharArray();
    for (int i = 0; i < chars.length; i++) {
        int tem = chars[i];
        String tem_hex = Integer.toHexString(tem);
        str.append(tem_hex);
        //释放对象
        tem_hex = null;
    }
    String asc_str = str.toString();
    //释放对象
    str.delete(0, str.length());
    str = null;
    return asc_str;
}
```

***

##### <font size="4" color="red">17. Java中Unicode码和字符串互转函数(1)</font>

```java
public String Str_To_Unicode(String hex_str) {
    StringBuffer sb = new StringBuffer();
    char [] source_char = hex_str.toCharArray();
    String unicode = null;
    for (int i=0;i<source_char.length;i++) {
        unicode = Integer.toHexString(source_char[i]);
        if (unicode.length() <= 2) {
            unicode = "00" + unicode;
        }
        sb.append(unicode);
    }
    unicode = sb.toString();
    //释放对象
    sb.delete(0, sb.length());
    sb = null;
    source_char = null;
    return unicode;
}

public String Unicode_To_Str(String unicode_str) {
    StringBuilder str = new StringBuilder();
    for (int i = 0; i < unicode_str.length(); i += 2) {
        int hex = Integer.parseInt(unicode_str.substring(i, i + 2), 16);
        char hex_char = (char) hex;
        if (hex_char != 0) {
            //追加数据
            str.append((char) hex);
        }
    }
    String unc_str = str.toString();
    //释放对象
    str.delete(0,str.length());
    str = null;
    return unc_str;
}
```

***

##### <font size="4" color="red">18. Java中道格拉斯普克算法</font>

**(1) 创建`Gps.java`文件**

`Gps.java`文件中内容：

```java
public class Gps {

    //索引
    private  int index;
    //经度
    private double lat;
    //纬度
    private double lon;
    public Gps(){

    }

    public Gps(int index, double lat, double lon) {
        this.index = index;
        this.lat = lat;
        this.lon = lon;
    }

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }

    public double getLat() {
        return lat;
    }

    public void setLat(double lat) {
        this.lat = lat;
    }

    public double getLon() {
        return lon;
    }

    public void setLon(double lon) {
        this.lon = lon;
    }

    @Override
    public String toString() {
        return "Gps [index=" + index + ", lat=" + lat + ", lon=" + lon + "]";
    }
}
```

**(2) 创建`Douglas.java`文件**

`Douglas.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class Douglas {


    /**
     * @method [calculationDistance]
     * @param gps1 : 位置点1
     * @param gps2 : 位置点2
     * @description : 计算两点之间的距离
     */
    private double calculationDistance(Gps gps1, Gps gps2){
        double lat1 = gps1.getLat();
        double lat2 = gps2.getLat();
        double lng1 = gps1.getLon();
        double lng2 = gps2.getLon();
        double radLat1 = lat1 * Math.PI / 180.0;
        double radLat2 = lat2 * Math.PI / 180.0;
        double a = radLat1 - radLat2;
        double b = (lng1 * Math.PI / 180.0) - (lng2 * Math.PI / 180.0);
        double s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2)
                                           + Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
        return s * 6370996.81;
    }

    /**
     * @method [distToSegment]
     * @param start : 起始点
     * @param end : 结束点
     * @param center : 中心点
     * @return dist : 距离
     * @description : 计算两点之间的距离
     */
    private double distToSegment(Gps start,Gps end,Gps center) {
        double a = Math.abs(calculationDistance(start, end));
        double b = Math.abs(calculationDistance(start, center));
        double c = Math.abs(calculationDistance(end, center));
        double p = (a + b + c) / 2.0;
        double s = Math.sqrt(Math.abs(p * (p - a) * (p - b) * (p - c)));
        double dist = (double)(s * 2.0 / a);
        return dist;
    }

    /**
     * @method [compressLine]
     * @param traList : 过滤的轨迹数据
     * @param result : 过滤后的轨迹数据
     * @param start : 起始点
     * @param end : 结束点
     * @param dMax : 允许最大距离误差
     * @description : 递归方式压缩轨迹
     * */
    private List<Gps> compressLine(List<Gps> traList,List<Gps> result,int start,int end,int dMax){
        if(start < end) {
            double maxDist = 0;
            int currentIndex = 0;
            Gps startPoint = traList.get(start);
            Gps endPoint = traList.get(end);
            Gps center = null;
            ///遍历轨迹点
            for(int i = start + 1; i < end; i++) {
                /**中心点*/
                center = traList.get(i);
                /**计算阀值*/
                double currentDist = distToSegment(startPoint, endPoint, center);
                if (currentDist > maxDist) {
                    maxDist = currentDist;
                    currentIndex = i;
                }
            }
            if (maxDist >= dMax) {
                //将当前点加入到过滤数组中
                result.add(traList.get(currentIndex));
                //将原来的线段以当前点为中心拆成两段，分别进行递归处理
                compressLine(traList,result,start, currentIndex, dMax);
                compressLine(traList,result,currentIndex, end, dMax);
            }
        }
        return result;
    }

    /**
     * @method [Peucker]
     * @param traList : 过滤的轨迹数据
     * @param dMax : 允许最大距离误差
     * @description : 递归方式压缩轨迹
     * */
    public List<Gps> Peucker(List<Gps> traList,int dMax){
        List<Gps> result = null;
        if (traList != null && traList.size() > 2) {
            result = new ArrayList<Gps>();
            result = compressLine(traList, result,0, traList.size() - 1, dMax);
            result.add(traList.get(0));
            result.add(traList.get(traList.size() - 1));
            result.sort(new Comparator<Gps>() {
                @Override
                public int compare(Gps gps1, Gps gps2) {
                    if (gps1.getIndex() < gps2.getIndex()) {
                        return -1;
                    } 
                    if (gps1.getIndex() > gps2.getIndex()) {
                        return 1; 
                    }    
                    return 0;
                }
            });
        }
        return result;
    }
}
```

**(3) 创建`test.java`文件**

`test.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.List;

public class test {

    public static void main(String[] args) {
        String[] list = {"117.212448,39.133785", "117.212669,39.133667", "117.213165,39.133297", "117.213203,39.13327",
                         "117.213554,39.133099", "117.213669,39.13295", "117.213921,39.132462", "117.214088,39.132126", 
                         "117.214142,39.131962", "117.214188,39.13176", "117.214233,39.131397", "117.21418,39.13055", 
                         "117.214279,39.130459", "117.214539,39.130375", "117.214874,39.130188", "117.216881,39.128716", 
                         "117.217598,39.127995", "117.217972,39.12759", "117.218338,39.127178", "117.218407,39.127071", 
                         "117.218567,39.126911", "117.219704,39.125702", "117.219795,39.12561", "117.220284,39.125114", 
                         "117.220619,39.124802", "117.221046,39.124348", "117.221138,39.124245", "117.221268,39.124092", 
                         "117.222321,39.122955", "117.222824,39.122406", "117.222916,39.122311", "117.223663,39.121544", 
                         "117.2239,39.121452", "117.224113,39.12159", "117.224251,39.121677", "117.225136,39.122208", 
                         "117.225281,39.122292", "117.225319,39.122311", "117.226273,39.122875", "117.226685,39.123127",
                         "117.227371,39.12352", "117.227806,39.123779", "117.228477,39.124134", "117.228531,39.124161", 
                         "117.228531,39.124161", "117.228668,39.124187", "117.228897,39.124325", "117.229767,39.12479", 
                         "117.230927,39.12545", "117.231186,39.12561", "117.231659,39.125908", "117.231834,39.126026", 
                         "117.232018,39.126186", "117.232185,39.126362", "117.232353,39.126583", "117.232658,39.126972", 
                         "117.232658,39.126972", "117.233124,39.12748", "117.233253,39.127609", "117.233368,39.127689", 
                         "117.233513,39.127762", "117.233665,39.127823", "117.233734,39.127846", "117.233833,39.127865", 
                         "117.233994,39.127888", "117.234138,39.127892", "117.234329,39.127884", "117.234612,39.127838",
                         "117.234955,39.127754", "117.235252,39.12767", "117.236282,39.12738", "117.237137,39.127129", 
                         "117.237671,39.126961", "117.237953,39.126949", "117.238213,39.126865", "117.238472,39.126793",
                         "117.2397,39.126434", "117.242233,39.125698", "117.243538,39.12532", "117.243645,39.125298"};
        List<Gps> gpsList = new ArrayList<Gps>();
        double lat =0d;
        double lon = 0d;
        String[] str = null;
        /**遍历*/
        for(int i=0;i< list.length;i++) {
            str = list[i].split(",");
            lon = Double.parseDouble(str[0]);
            lat = Double.parseDouble(str[1]);
            /**追加数据*/
            gpsList.add(new Gps(i,lat,lon));
        }
        Douglas douglas = new Douglas();
        List<Gps> result = douglas.Peucker(gpsList, 10);
        System.out.println("总数：" + result.size());
        System.out.println(result);
    }
}
```

***

##### <font size="4" color="red">19. Java中Netty集成Websocket</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建test.java文件**

test.java文件中内容：

```java
public class test {
    public static void main(String[] args) {
        new NettWebSocket().WebSocketServer();
    }
}
```

**(3) 创建NettWebSocket.java文件**

NettWebSocket.java文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class NettWebSocket {


    public void WebSocketServer() {

        //线程数
        int boss_os_num = 8;
        //数据接收线程组
        EventLoopGroup bossLoop = new NioEventLoopGroup(boss_os_num);
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            ServerBootstrap serverBoot = new ServerBootstrap();
            //配置Tcp参数
            serverBoot.group(bossLoop,eventLoop)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 1024 * 1024)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new WebSocketChannel());
            ChannelFuture cf = serverBoot.bind(8080).sync();
            //等待线程池结束
            cf.channel().closeFuture().sync();
        }
        catch(Exception e) {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
        finally {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }  
    }
}
```

**(4) 创建WebSocketChannel.java文件**

WebSocketChannel.java文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.stream.ChunkedWriteHandler;

public class WebSocketChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline().addLast(new HttpServerCodec())
            .addLast(new ChunkedWriteHandler())
            .addLast(new HttpObjectAggregator(65536 * 10000))
            .addLast(new WebSocketHandler());
    }
}
```

**(5) 创建WebSocketHandler.java文件**

WebSocketHandler.java文件中内容：

```java
import io.netty.channel.Channel;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.DefaultFullHttpResponse;
import io.netty.handler.codec.http.FullHttpRequest;
import io.netty.handler.codec.http.HttpHeaderNames;
import io.netty.handler.codec.http.HttpResponseStatus;
import io.netty.handler.codec.http.HttpVersion;
import io.netty.handler.codec.http.websocketx.CloseWebSocketFrame;
import io.netty.handler.codec.http.websocketx.PingWebSocketFrame;
import io.netty.handler.codec.http.websocketx.PongWebSocketFrame;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.handler.codec.http.websocketx.WebSocketFrame;
import io.netty.handler.codec.http.websocketx.WebSocketServerHandshaker;
import io.netty.handler.codec.http.websocketx.WebSocketServerHandshakerFactory;
import io.netty.util.CharsetUtil;
import static io.netty.handler.codec.http.HttpHeaderNames.HOST;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;

public class WebSocketHandler extends SimpleChannelInboundHandler<Object> {

    //web握操作对象
    private WebSocketServerHandshaker handshaker = null;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object obj) throws Exception {
        //检测请求状态
        if (obj instanceof FullHttpRequest) {
            //http请求响应
            WebHttpRequest(ctx, ((FullHttpRequest) obj));
        } else if (obj instanceof WebSocketFrame) {
            WebSocketRequest(ctx, (WebSocketFrame) obj);
        }

    }

    /*
     * @method [WebHttpRequest]
     * @param [ChannelHandlerContext] ctx : 广播数据接收通道对象
     * @param [FullHttpRequest] req : http请求数据
     * @description : http请求响应
     * */
    private void WebHttpRequest(ChannelHandlerContext ctx,FullHttpRequest req) {
        if (!req.decoderResult().isSuccess() 
            || (!"websocket".equals(req.headers().get("Upgrade")))) {
            Htp_Res_Send(ctx,"访问失败");
            return;
        }
        //请求地址 
        String web_url = WebSocketUrl(req);
        WebSocketServerHandshakerFactory wsFactory = new 
            WebSocketServerHandshakerFactory(web_url, null, false);
        handshaker = wsFactory.newHandshaker(req);
        if (handshaker == null) {
            WebSocketServerHandshakerFactory.sendUnsupportedVersionResponse(ctx.channel());
        } else {
            //握手
            handshaker.handshake(ctx.channel(), req);
        }
    }


    private void WebSocketRequest(ChannelHandlerContext ctx, WebSocketFrame frame) {
   
        //地址标识
        Channel ch = ctx.channel();
        //连接地址
        String remote_addr = ch.remoteAddress().toString();
        //判断是否关闭链路的指令
        if (frame instanceof CloseWebSocketFrame) {
            handshaker.close(ctx.channel(), (CloseWebSocketFrame) frame
                             .retain());
        }
        //判断是否ping消息
        if (frame instanceof PingWebSocketFrame) {
            ctx.channel().write(
                new PongWebSocketFrame(frame.content().retain()));
            return;
        }
        //注册消息
        String web_req = ((TextWebSocketFrame) frame).text();
        System.out.println(web_req);
        //初始化消息
        TextWebSocketFrame txt_str = new TextWebSocketFrame(web_req);
        //消息推送
        ctx.writeAndFlush(txt_str);
    }

    /*
     * @method [WebSocketUrl]
     * @param [FullHttpRequest] req : http请求数据
     * @param [String] location : websocket请求地址
     * @description : http地址
     * */
    private String WebSocketUrl(FullHttpRequest req) {
        String location = req.headers().get(HOST).toString();
        return "ws://" + location;
    }


    public void Htp_Res_Send(ChannelHandlerContext htp_ctx,String cmd_str) {
        //检测对象
        if(htp_ctx != null) {
            //回复对象
            DefaultFullHttpResponse htp_res = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
            //错误对象
            ByteBuf buf = Unpooled.copiedBuffer(cmd_str,CharsetUtil.UTF_8);
            //允许跨域放
            htp_res.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_ORIGIN,"*");
            htp_res.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_CREDENTIALS,"true");
            htp_res.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_HEADERS,"cache-control,content-type,hash-referer,x-requested-with");
            //写明类型
            htp_res.headers().set(HttpHeaderNames.CONTENT_TYPE, "application/json; charset=UTF-8");
            //写入内容
            htp_res.content().writeBytes(buf);
            //回复内容
            htp_ctx.write(htp_res);
            //断开连接
            htp_ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
            //释放对象
            buf = null;
            htp_res = null;
            cmd_str = null;
        }
    }

    /*
     * @method [exceptionCaught]
     * @param [ChannelHandlerContext] ctx : 广播协议接入通道对象
     * @param [Throwable] cause : 广播异常信息
     * @description : 广播异常信息处理
     * */
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }

}
```

***

##### <font size="4" color="red">20. Java中Netty集成Http服务器</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建test.java文件**

```java
public class test {
    public static void main(String[] args) {
        new nettyHttp().HttpServer();
    }
}
```

**(3) 创建nettyHttp.java文件**

nettyHttp.java文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class nettyHttp {


    public void HttpServer() {

        //线程数
        int boss_os_num = 8;
        //数据接收线程组
        EventLoopGroup bossLoop = new NioEventLoopGroup(boss_os_num);
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            ServerBootstrap serverBoot = new ServerBootstrap();
            //配置Tcp参数
            serverBoot.group(bossLoop,eventLoop)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 1024 * 1024)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new HttpChannel());
            ChannelFuture cf = serverBoot.bind(8080).sync();
            //等待线程池结束
            cf.channel().closeFuture().sync();
        }
        catch(Exception e) {

            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
        finally {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }  
    }
}
```

**(4) 创建HttpChannel.java文件**

HttpChannel.java文件中内容：

```java
import java.util.concurrent.TimeUnit;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.stream.ChunkedWriteHandler;
import io.netty.handler.timeout.IdleStateHandler;

public class HttpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline().addLast(new IdleStateHandler(30,0,0,TimeUnit.SECONDS))
            .addLast(new HttpServerCodec())
            .addLast(new ChunkedWriteHandler())
            .addLast(new HttpObjectAggregator(65536 * 10000))
            .addLast(new HttpHandler());
    }
}
```

**(5) 创建HttpHandler.java文件**

HttpHandler.java文件中内容：

```java
import java.io.File;
import java.io.RandomAccessFile;
import java.net.URLDecoder;
import java.util.regex.Pattern;

import javax.activation.MimetypesFileTypeMap;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.DefaultFullHttpResponse;
import io.netty.handler.codec.http.DefaultHttpResponse;
import io.netty.handler.codec.http.FullHttpRequest;
import io.netty.handler.codec.http.HttpChunkedInput;
import io.netty.handler.codec.http.HttpContent;
import io.netty.handler.codec.http.HttpHeaderNames;
import io.netty.handler.codec.http.HttpHeaderValues;
import io.netty.handler.codec.http.HttpMethod;
import io.netty.handler.codec.http.HttpResponse;
import io.netty.handler.codec.http.HttpResponseStatus;
import io.netty.handler.codec.http.HttpUtil;
import io.netty.handler.codec.http.HttpVersion;
import io.netty.handler.codec.http.LastHttpContent;
import io.netty.handler.codec.http.multipart.DefaultHttpDataFactory;
import io.netty.handler.codec.http.multipart.FileUpload;
import io.netty.handler.codec.http.multipart.HttpDataFactory;
import io.netty.handler.codec.http.multipart.HttpPostRequestDecoder;
import io.netty.handler.codec.http.multipart.InterfaceHttpData;
import io.netty.handler.stream.ChunkedFile;
import io.netty.handler.timeout.IdleStateEvent;
import io.netty.util.CharsetUtil;
import io.netty.util.ReferenceCountUtil;
import io.netty.util.internal.SystemPropertyUtil;

public class HttpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object obj) throws Exception {
        //检测请求状态
        if (obj instanceof FullHttpRequest) {
            //http请求响应
            WebHttpRequest(ctx,  ((FullHttpRequest) obj));
        } 

    }

    /*
     * @method [userEventTriggered]
     * @param [ChannelHandlerContext] ctx : 广播协议接入通道对象
     * @param [Object] evt : 接收到的数据信息
     * @description : 超时计算
     * */ 
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) 
        throws Exception {
        //检测到期时间
        if (evt instanceof IdleStateEvent) { 
            //地址标识
            Channel ch = ctx.channel();
            //连接地址
            String remote_addr = ch.remoteAddress().toString();
            //超时操作
            Htp_Res_Send(ctx,"访问超时");
        } else {
            //监听消息
            super.userEventTriggered(ctx, evt);
        }
    }

    //post
    private final String post_url  = "/post";
    //上传
    private final String upload_url = "/upload";

    private void WebHttpRequest(ChannelHandlerContext ctx, FullHttpRequest req) {
        //验证请求
        if (req.decoderResult().isSuccess()) {
            //检测对象
            if(req.method().equals(HttpMethod.POST)) {
                //地址解析
                String req_url = req.uri().toString();
                //解析请求数据
                ByteBuf buf = req.content();
                //数据解析
                String req_str = buf.toString(CharsetUtil.UTF_8);
                //请求地址分组
                switch(req_url) {
                        //post访问
                    case post_url:
                        System.out.println(req_str);
                        Htp_Res_Send(ctx,"访问成功");
                        break;
                    case upload_url :
                        //异常捕获
                        try {
                            //文件解码
                            HttpDataFactory htp_fac = new DefaultHttpDataFactory(false);
                            //http请求解密
                            HttpPostRequestDecoder htp_dec = new HttpPostRequestDecoder(htp_fac, req);
                            //检测对象
                            if (htp_dec != null && req instanceof HttpContent) {
                                //读取文件
                                HttpContent htp_chunk = (HttpContent) req;
                                //读取内存
                                htp_dec.offer(htp_chunk);
                                //检测对象
                                if(htp_chunk instanceof LastHttpContent) {
                                    //遍历接口
                                    while(htp_dec.hasNext()) {
                                        //数据内容
                                        InterfaceHttpData htp_data = htp_dec.next();
                                        //文件上传对象
                                        FileUpload htp_file = (FileUpload) htp_data;
                                        //检测对象
                                        if(htp_file.isCompleted()) {
                                            //文件名称
                                            String file_name = htp_file.getFilename();
                                            //ota文件
                                            File file = new File("D:\\upload\\"+ file_name);
                                            //检测父文件夹
                                            if (!file.getParentFile().exists()) {
                                                //创建文件夹
                                                file.getParentFile().mkdirs();
                                            }
                                            //检测文件是否存在
                                            if (!file.exists()) {
                                                //创建文件
                                                file.createNewFile();
                                            }
                                            //重新替换
                                            htp_file.renameTo(file);
                                            //清除上传
                                            htp_dec.removeHttpDataFromClean(htp_file);
                                        }
                                        //释放对象
                                        htp_data.release();
                                        htp_data = null;
                                        htp_file = null;
                                    }
                                }
                            } 
                            //释放对象
                            htp_dec.destroy();
                            htp_fac.cleanAllHttpData();
                            htp_dec = null;
                            htp_fac = null;
                        }
                        catch(Exception e) {
                            Htp_Res_Send(ctx,"上传成功");
                        }
                        break;
                    default : 
                        Htp_Res_Send(ctx,"访问错误");
                        break;
                }
                //释放对象
                ReferenceCountUtil.release(buf);
                buf.clear();
            }
            //get请求
            if(req.method().equals(HttpMethod.GET)) {
                //异常捕获
                try {
                    //地址解析
                    String req_url = req.uri();
                    //解析文件路径
                    final String file_path = Request_Url(req_url);
                    //检测对象
                    if(file_path == null) {
                        Htp_Res_Send(ctx,"访问错误");
                    }
                    //初始化对象
                    File file_url = new File(file_path);
                    //检测对象
                    if(file_url.isHidden() || !file_url.exists()) {
                        Htp_Res_Send(ctx,"访问错误");
                    }
                    //检测对象
                    if (file_url.isDirectory()) {
                        Htp_Res_Send(ctx,"访问错误");
                    }
                    //随机读取文件
                    RandomAccessFile raf = new RandomAccessFile(file_url, "r");
                    //文件长度
                    long fileLength = raf.length();
                    //回复头
                    HttpResponse response = new DefaultHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
                    //标识长度
                    HttpUtil.setContentLength(response, fileLength);
                    //设置内容
                    setContentTypeHeader(response, file_url);
                    //保持长连接
                    if (HttpUtil.isKeepAlive(req)) {
                        //允许跨域放
                        response.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_ORIGIN,"*");
                        response.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_CREDENTIALS,"true");
                        response.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_HEADERS,"cache-control,content-type,hash-referer,x-requested-with");
                        //写明类型
                        response.headers().set(HttpHeaderNames.CONTENT_TYPE, "application/octet-stream; charset=UTF-8");
                        //设置长连接
                        response.headers().set(HttpHeaderNames.CONNECTION, HttpHeaderValues.KEEP_ALIVE);
                    }
                    //写入回复
                    ctx.write(response);
                    //通过Netty的ChunkedFile对象直接将文件写入发送到缓冲区中
                    ctx.write(new HttpChunkedInput(new ChunkedFile(raf, 0, fileLength, 8192)),ctx.newProgressivePromise());                     
                    //通道对象
                    ChannelFuture lastContentFuture = ctx.writeAndFlush(LastHttpContent.EMPTY_LAST_CONTENT);
                    //检测对象
                    if (!HttpUtil.isKeepAlive(req)) {
                        //关闭连接
                        lastContentFuture.addListener(ChannelFutureListener.CLOSE);
                    }
                }
                catch(Exception e) {
                    Htp_Res_Send(ctx,"访问错误");
                }
            }
        }

    }

    /**
    * @mthod [Request_Url]
    * @param [String] url : 路径地址
    * @description : 请求地址
    * */    
    private  String Request_Url(String url) {
        //转移路径
        try {
            url = URLDecoder.decode(url, "UTF-8");
        } catch (Exception e) {
            throw new Error(e);
        }
        //检测路径
        if (url.isEmpty() || url.charAt(0) != '/') {
            return null;
        }
        //文件路径分割
        url = url.replace('/', File.separatorChar);
        //文件分割
        if (url.contains(File.separator + '.') ||
            url.contains('.' + File.separator) ||
            url.charAt(0) == '.' || url.charAt(url.length() - 1) == '.' ||
            insecure_url.matcher(url).matches()) {
            return null;
        }
        //转换地址
        return SystemPropertyUtil.get("user.dir") + File.separator + url;
    }

    //安全地址正则表达式
    private Pattern insecure_url = Pattern.compile(".*[<>&\"].*");

    /**
    * @mthod [setContentTypeHeader]
    * @param [HttpResponse] response : 回复对象
    * @param [File] file : 文件对象
    * @description : 请求地址
    * */  
    private void setContentTypeHeader(HttpResponse response, File file) {
        //文件map对象
        MimetypesFileTypeMap mimeTypesMap = new MimetypesFileTypeMap();
        //设置通知对象
        response.headers().set(HttpHeaderNames.CONTENT_TYPE, mimeTypesMap.getContentType(file.getPath()));
    }

    /*
     * @method [Htp_Res_Send]
     * @param [ChannelHandlerContext] htp_ctx : 通道对象
     * @param [String] htp_str : ota消息内容
     * @description : 消息回复
     * */
    public void Htp_Res_Send(ChannelHandlerContext htp_ctx,String htp_str) {
        //检测对象
        if(htp_ctx != null) {
            //回复对象
            DefaultFullHttpResponse htp_res = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
            //错误对象
            ByteBuf ota_buf = Unpooled.copiedBuffer(htp_str,CharsetUtil.UTF_8);
            //允许跨域放
            htp_res.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_ORIGIN,"*");
            htp_res.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_CREDENTIALS,"true");
            htp_res.headers().set(HttpHeaderNames.ACCESS_CONTROL_ALLOW_HEADERS,"cache-control,content-type,hash-referer,x-requested-with");
            //写明类型
            htp_res.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/json; charset=UTF-8");
            //写入内容
            htp_res.content().writeBytes(ota_buf);
            //回复内容
            htp_ctx.write(htp_res);
            //断开连接
            htp_ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
            //释放对象
            ota_buf.clear();
            htp_res = null;
            ota_buf = null;
        }
    }

    /*
     * @method [exceptionCaught]
     * @param [ChannelHandlerContext] ctx : 广播协议接入通道对象
     * @param [Throwable] cause : 广播异常信息
     * @description : 广播异常信息处理
     * */
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }

}
```

***

##### <font size="4" color="red">21. Java中Netty集成Tcp服务器端(避免断包)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建test.java文件**

test.java文件中内容：

```java
public class test {
    public static void main(String[] args) {
        new nettyTcp().TcpServer();
    }
}
```

**(3) 创建nettyTcp.java文件**

nettyTcp.java文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class nettyTcp {


    public void TcpServer() {

        //线程数
        int boss_os_num = 8;
        //数据接收线程组
        EventLoopGroup bossLoop = new NioEventLoopGroup(boss_os_num);
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
        //配置netty中ServerBootstrap对象
        ServerBootstrap serverBoot = new ServerBootstrap();
        //配置Tcp参数
        serverBoot.group(bossLoop,eventLoop)
                  .channel(NioServerSocketChannel.class)
                  .option(ChannelOption.SO_BACKLOG, 1024 * 1024)
                  .childOption(ChannelOption.SO_KEEPALIVE, true)
                                  .childHandler(new TcpChannel());
        ChannelFuture cf = serverBoot.bind(8080).sync();
          //等待线程池结束
          cf.channel().closeFuture().sync();
        }
        catch(Exception e) {

            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
        finally {
            //释放数据接收线程组
            bossLoop.shutdownGracefully();
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }  
    }
}
```

**(4) 创建`TcpChannel.java`文件**

`TcpChannel.java`文件中内容：

```java
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.serialization.ClassResolvers;
import io.netty.handler.codec.serialization.ObjectDecoder;
import io.netty.handler.codec.serialization.ObjectEncoder;

public class TcpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline().addLast(new ObjectEncoder())
            .addLast(new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())))
            .addLast(new TcpHandler());
        //通道关闭事件
        ChannelFuture scf = sch.closeFuture();
        //添加关闭事件
        scf.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture cf) throws Exception {
                //通道关闭事件
                Channel ch = cf.channel(); 
                //连接地址
                String remote_addr = ch.remoteAddress().toString();

            }

        });
    }
}
```

**(5) 创建`TcpHandler.java`文件**

`TcpHandler.java`文件中内容：

```java
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class TcpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        //地址标识
        Channel ch = ctx.channel();
        //连接地址
        String remote_addr = ch.remoteAddress().toString();
        //转换字节编码
        String ser_str = (String) msg;
        //回复广播接口
        ctx.writeAndFlush(ser_str);
    }

    /*
     * @method [exceptionCaught]
     * @param [ChannelHandlerContext] ctx : 广播协议接入通道对象
     * @param [Throwable] cause : 广播异常信息
     * @description : 广播异常信息处理
     * */
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }

}
```

***

##### <font size="4" color="red">22. Java中Netty集成Tcp客户端(避免断包)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

**(2) 创建test.java文件**

test.java文件中内容：

```java
public class test {
    public static void main(String[] args) {
        new nettyTcp().TcpClient();
    }
}
```

**(3) 创建nettyTcp.java文件**

nettyTcp.java文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

public class nettyTcp {


    public void TcpClient() {
        //线程数
        int event_os_num = 32;
        //Tcp管理线程组
        EventLoopGroup eventLoop = new NioEventLoopGroup(event_os_num);
        try {
            //配置netty中ServerBootstrap对象
            Bootstrap ClientBoot = new Bootstrap();
            //配置Tcp参数
            ClientBoot.group(eventLoop)
                .channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .handler(new TcpChannel());
            ChannelFuture cf = ClientBoot.connect("120.24.248.220",8006).sync();
            //管道对象
            Channel    ch = cf.channel();
            //接入广播节点
            ch.writeAndFlush("{\"status\":101}");

        }
        catch(Exception e) {
            //释放Tcp线程组
            eventLoop.shutdownGracefully();
        }
    }
}
```

**(4) 创建TcpChannel.java文件**

TcpChannel.java文件中内容：

```java
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.serialization.ClassResolvers;
import io.netty.handler.codec.serialization.ObjectDecoder;
import io.netty.handler.codec.serialization.ObjectEncoder;

public class TcpChannel extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel sch) throws Exception {
        //管理数据接收管道
        sch.pipeline().addLast(new ObjectEncoder())
            .addLast(new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())))
            .addLast(new TcpHandler());
        //通道关闭事件
        ChannelFuture scf = sch.closeFuture();
        //添加关闭事件
        scf.addListener(new ChannelFutureListener() {
            public void operationComplete(ChannelFuture cf) throws Exception {
                //通道关闭事件
                Channel ch = cf.channel(); 
                //连接地址
                String remote_addr = ch.remoteAddress().toString();

            }

        });
    }
}
```

**(5) 创建TcpHandler.java文件**

TcpHandler.java文件中内容：

```java
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class TcpHandler extends SimpleChannelInboundHandler<Object> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        //地址标识
        Channel ch = ctx.channel();
        //连接地址
        String remote_addr = ch.remoteAddress().toString();
        //转换字节编码
        String ser_str = (String) msg;
        System.out.println(ser_str);
        //回复广播接口
        ctx.writeAndFlush(ser_str);
    }

    /*
     * @method [exceptionCaught]
     * @param [ChannelHandlerContext] ctx : 广播协议接入通道对象
     * @param [Throwable] cause : 广播异常信息
     * @description : 广播异常信息处理
     * */
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        //释放对象
        ctx.close();
        ctx = null;
    }

}
```

***

##### <font size="4" color="red">23. Java中监听Redis过期时间</font>

**(1) 依赖包**

```xml
<!--redis连接-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

**(2) 创建test.java文件**

test.java文件中内容：

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class test {

    //只有修改配置文件redis.conf中的：notify-keyspace-events Ex，默认为notify-keyspace-events "" 

    //修改好配置文件后，redis会对设置了expire的数据进行监听，当数据过期时便会将其从redis中删除：
    @SuppressWarnings("resource")
    public static void main(String[] args) {

        //连接池配置
        JedisPoolConfig config = new JedisPoolConfig(); 
        //设置等待时间
        config.setMaxWaitMillis(5000);
        //获取连接时检测有效性
        config.setTestOnBorrow(false); 
        config.setTestOnReturn(true);
        //空闲时检查连接有效性
        config.setTestWhileIdle(true);
        //释放连接扫描间隔
        config.setTimeBetweenEvictionRunsMillis(5000);
        //表示每次释放连接的最大数目
        config.setNumTestsPerEvictionRun(5000);
        //这一项只有在timeBetweenEvictionRunsMillis大于0时才有意义
        config.setMinEvictableIdleTimeMillis(5000);
        //连接耗尽是否阻塞
        config.setBlockWhenExhausted(false);
        JedisPool pool = new JedisPool(config, "127.0.0.1",8002);

        Jedis jedis = pool.getResource();
        jedis.psubscribe(new KeyExpiredListener(), "__key*__:*");

    }
}
```

**(3) 创建KeyExpiredListener.java文件**

KeyExpiredListener.java文件中内容：

```java
import redis.clients.jedis.JedisPubSub;

public class KeyExpiredListener extends JedisPubSub {

    @Override
    public void onPSubscribe(String pattern, int subscribedChannels) {
        System.out.println("onPSubscribe "
                           + pattern + " " + subscribedChannels);
    }

    @Override
    public void onPMessage(String pattern, String channel, String message) {

        System.out.println("onPMessage pattern "
                           + pattern + " " + channel + " " + message);
    }

}
```

> **注：**修改redis配置文件redis.conf
> 
> ```
> notify-keyspace-events Ex
> notify-keyspace-events ""
> ```
> 
> 修改完成后重启redis此时设置expire过期时间，当数据过期时触发

***

##### <font size="4" color="red">24. Java中使用Forest接口访问</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>com.dtflys.forest</groupId>
    <artifactId>spring-boot-starter-forest</artifactId>
    <version>1.3.8</version>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.3.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```

**(2) 创建MyClient.java文件**

MyClient.java文件中内容：

```java
import com.dtflys.forest.annotation.DataParam;
import com.dtflys.forest.annotation.Request;
public interface MyClient {
    @Request(
        url = "http://www.carhere.net/",
        type = "get",
        headers = {"Accept:text/plan"}
    )
    String send(@DataParam("username") String username);
}
```

**(3) 创建test.java文件**

test.java文件中内容：

```java
import com.dtflys.forest.config.ForestConfiguration;
import com.dtflys.forest.ssl.SSLUtils;
public class test {

    public static void main(String[] args) throws Exception {
        ForestConfiguration configuration = ForestConfiguration.configuration();
        configuration.setBackendName("okhttp3");
        // 连接池最大连接数，默认值为500
        configuration.setMaxConnections(123);
        // 每个路由的最大连接数，默认值为500
        configuration.setMaxRouteConnections(222);
        // 请求超时时间，单位为毫秒, 默认值为3000
        configuration.setTimeout(3000);
        // 连接超时时间，单位为毫秒, 默认值为2000
        configuration.setConnectTimeout(2000);
        // 请求失败后重试次数，默认为0次不重试
        configuration.setRetryCount(3);
        // 单向验证的HTTPS的默认SSL协议，默认为SSLv3
        configuration.setSslProtocol(SSLUtils.SSLv3);
        // 打开或关闭日志，默认为true
        configuration.setLogEnabled(false);
        //初始化客户端
        MyClient myClient = configuration.createInstance(MyClient.class);
        String result = myClient.send("test");
        System.out.println(result);
    } 
}
```

***

##### <font size="4" color="red">25. Java中使用自定义异常重写FillInStackTrace</font>

**(1) 创建ApiException.java文件**

ApiException.java文件中内容：

```java
public class ApiException extends Exception  {

    private static final long serialVersionUID = 1L;


    public ApiException(String message) {
        super(message);
    }

    /*
     * 重写fillInStackTrace方法会使得这个自定义的异常不会收集线程的整个异常栈信息，会大大
     * 提高减少异常开销。
     */
    @Override
    public synchronized Throwable fillInStackTrace() {
        return this;
    }
}
```

**(2) 创建test.java文件**

test.java文件中内容：

```java
public class test {

    public static void main(String[] args) {
        try {
            throw new ApiException("由于MyException重写了fillInStackTrace方法，那么它不会收集线程运行栈信息。");
        } catch (ApiException e) {
            e.printStackTrace(); 
        }
    } 
}
```

***

##### <font size="4" color="red">26. Java中使用Tessercat图像识别</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>net.sourceforge.tess4j</groupId>
    <artifactId>tess4j</artifactId>
    <version>4.4.0</version>
</dependency>
```

**(2) 创建test.java文件**

test.java文件中内容：

```java
import java.io.File;
import net.sourceforge.tess4j.ITesseract;
import net.sourceforge.tess4j.Tesseract;
import net.sourceforge.tess4j.TesseractException;

public class test {

    public static void main(String[] args) throws Exception {
        // 识别图片的路径（修改为自己的图片路径）
        String path = "test.png";
        // 语言库位置（修改为跟自己语言库文件夹的路径）
        String lagnguagePath = "D:\\Program Files\\Tesseract-OCR\\tessdata";
        File file = new File(path);
        ITesseract instance = new Tesseract();
        //设置训练库的位置
        instance.setDatapath(lagnguagePath);

        //chi_sim ：简体中文， eng    根据需求选择语言库
        instance.setLanguage("chi_sim");
        String result = null;
        try {
            long startTime = System.currentTimeMillis();
            result =  instance.doOCR(file);
            long endTime = System.currentTimeMillis();
            System.out.println("Time is：" + (endTime - startTime) + " 毫秒");
        } catch (TesseractException e) {
            e.printStackTrace();
        }
        System.out.println("result: ");
        System.out.println(result);
    } 

}
```

> **注：**需安装`Tessercat`并设置成环境变量

****

##### <font size="4" color="red">27. Java中公平分配Hash一致算法</font>

```java
import java.util.UUID;

public class test {

    public static int FnvHash(String key) {
        final int p = 16777619;
        long hash = (int) 2166136261L;
        for (int i = 0, n = key.length(); i < n; i++) {
            hash = (hash ^ key.charAt(i)) * p;
        }
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;
        return ((int) hash & 0x7FFFFFFF) ;
    }

    public static void main( String[] args ) {
        //a,b,c,d,分别记录四组分到的imei个数
        int a = 0;
        int b = 0;
        int c = 0;
        int d = 0;
        for (int i = 0; i < 10000; i++) {
            //模拟15位的imei码
            String str = UUID.randomUUID().toString().replaceAll("-","");
            //模4 将FnvHash算法得到的固定结果分成四组
            int hash = FnvHash(str) % 4;
            switch (hash){
                case 0:a++;break;
                case 1:b++;break;
                case 2:c++;break;
                case 3:d++;break;
            }
        }
        System.out.println("a：" + a);
        System.out.println("b：" + b);
        System.out.println("c：" + c);
        System.out.println("d：" + d);
        System.out.println("a+b+c+d：" + (a + b + c + d));
    }
}
```

****

##### <font size="4" color="red">28. Java中负载均衡算法</font>

**1.随机算法**

        通过系统随机函数，根据后台服务器的server的地址随机选取其中一台服务器进行访问，根据概率论的相关知识，随着调用量的增加，最终的访问趋于平均，就是达到了均衡的目的。

```java
import java.util.ArrayList;
import java.util.Map;
import java.util.Set;
import java.util.Random;
import java.util.concurrent.ConcurrentHashMap;

public class test {

    static Map<String,Integer> ipMap=new ConcurrentHashMap<String,Integer>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);
    }

    public String Random() {
        Set<String> ipSet=ipMap.keySet();
        //定义一个list放所有server
        ArrayList<String> ipArrayList=new ArrayList<String>();
        ipArrayList.addAll(ipSet);
        //循环随机数
        Random random=new Random();
        //随机数在list数量中取（1-list.size）
        int pos=random.nextInt(ipArrayList.size());
        String serverNameReturn= ipArrayList.get(pos);
        return  serverNameReturn;
    }

    public static void main(String[] args) {
        test testRandom=new test();
        for (int i =0;i<10;i++){
            String server=testRandom.Random();
            System.out.println(server);
        }
    }
}
```

**2.加权随机算法**

        加权随机算法就是在上面的随机算法的基础上做的优化，比如一些性能好的Server多承担一些，请求根据权重分发到各个服务器。

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.Random;
import java.util.concurrent.ConcurrentHashMap;

public class test {

    static Map<String,Integer> ipMap=new ConcurrentHashMap<String,Integer>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);
    }
    public String weightRandom() {
        Set<String> ipSet = ipMap.keySet();
        Iterator<String> ipIterator = ipSet.iterator();
        //定义一个list放所有server
        ArrayList<String> ipArrayList = new ArrayList<String>();
        //循环set，根据set中的可以去得知map中的value，给list中添加对应数字的server数量
        while (ipIterator.hasNext()) {
            String serverName = ipIterator.next();
            Integer weight = ipMap.get(serverName);
            for (int i = 0; i < weight; i++) {
                ipArrayList.add(serverName);
            }
        }
        //循环随机数
        Random random = new Random();
        //随机数在list数量中取（1-list.size）
        int pos = random.nextInt(ipArrayList.size());
        String serverNameReturn = ipArrayList.get(pos);
        return serverNameReturn;
    }

    public static void main(String[] args) {
        test testRandom=new test();
        for (int i =0;i<10;i++){
            String server=testRandom.weightRandom();
            System.out.println(server);
        }
    }
}
```

**3.轮询算法**

        轮询算法顾名思义，就是按照顺序轮流访问后台服务。

```java
import java.util.ArrayList;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class test {

    static Map<String,Integer> ipMap=new ConcurrentHashMap<String,Integer>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);
    }

    Integer  pos = 0;

    public String RoundRobin(){
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);
        //2.取出来key,放到set中
        Set<String> ipset=ipServerMap.keySet();
        //3.set放到list，要循环list取出
        ArrayList<String> iplist=new ArrayList<String>();
        iplist.addAll(ipset);
        String serverName=null;
        //4.定义一个循环的值，如果大于set就从0开始
        synchronized(pos){
            if (pos>=ipset.size()){
                pos=0;
            }
            serverName=iplist.get(pos);
            //轮询+1
            pos ++;
        }
        return serverName;
    }

    public static void main(String[] args) {
        test testRandom=new test();
        for (int i =0;i<12;i++){
            String server=testRandom.RoundRobin();
            System.out.println(server);
        }
    }
}
```

**4.加权轮询算法**

        加权随机一样，加权轮询，就是在轮询的基础上加上权重，将服务器性能好的，权重高一些。

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class test {

    static Map<String,Integer> ipMap=new ConcurrentHashMap<String,Integer>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);
    }

    Integer pos=0;
    public String weightRoundRobin(){
        Map<String,Integer> ipServerMap=new ConcurrentHashMap<>();
        ipServerMap.putAll(ipMap);
        Set<String> ipSet=ipServerMap.keySet();
        Iterator<String> ipIterator=ipSet.iterator();
        //定义一个list放所有server
        ArrayList<String> ipArrayList=new ArrayList<String>();
        //循环set，根据set中的可以去得知map中的value，给list中添加对应数字的server数量
        while (ipIterator.hasNext()){
            String serverName=ipIterator.next();
            Integer weight=ipServerMap.get(serverName);
            for (int i = 0;i < weight ;i++){
                ipArrayList.add(serverName);
            }
        }
        String serverName=null;
        if (pos>=ipArrayList.size()){
            pos=0;
        }
        serverName=ipArrayList.get(pos);
        //轮询+1
        pos ++;
        return  serverName;
    }

    public static void main(String[] args) {
        test testRandom=new test();
        for (int i =0;i<12;i++){
            String server=testRandom.weightRoundRobin();
            System.out.println(server);
        }
    }
}
```

**5.`Ip-Hash`算法**

        根据`hash`算法，将请求大致均分的分配到各个服务器上。

```java
import java.util.ArrayList;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class test {

    static Map<String,Integer> ipMap=new ConcurrentHashMap<String,Integer>();
    static {
        ipMap.put("192.168.13.1",1);
        ipMap.put("192.168.13.2",2);
        ipMap.put("192.168.13.3",4);
    }

    public String ipHash(String clientIP) {
        //2.取出来key,放到set中
        Set<String> ipset = ipMap.keySet();
        //3.set放到list，要循环list取出
        ArrayList<String> iplist = new ArrayList<String>();
        iplist.addAll(ipset);
        //对ip的hashcode值取余数，每次都一样的
        int hashCode = clientIP.hashCode();
        int serverListsize = iplist.size();
        int pos = hashCode % serverListsize;
        return iplist.get(pos);

    }

    public static void main(String[] args) {
        test testIpHash=new test();
        for (int i =0;i<12;i++){
            System.out.println(testIpHash.ipHash("192.168.21.2"));
            System.out.println(testIpHash.ipHash("192.168.21.3"));
            System.out.println(testIpHash.ipHash("192.168.21.1"));
        }
    }
}
```

**6.最小连接数算法**

```java
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class test {

    static Map<String,Integer> ipMap=new ConcurrentHashMap<String,Integer>();
    static {
        ipMap.put("192.168.13.1",0);
        ipMap.put("192.168.13.2",0);
        ipMap.put("192.168.13.3",0);
    }

    //从list中选取接受请求数最少的服务并返回
    public String leastConnection() {
        Iterator<String> ipListIterator = ipMap.keySet().iterator();
        String serverName = null;
        int times = 0;//访问次数
        while (ipListIterator.hasNext()) {
            String tmpServerName = ipListIterator.next();
            int requestTimes = ipMap.get(tmpServerName);
            //第一次需要赋值
            if (times == 0) {
                serverName = tmpServerName;
                times = requestTimes;
            } else {
                //找到最小次数
                if (times > requestTimes) {
                    serverName = tmpServerName;
                    times = requestTimes;
                }
            }
        }
        ipMap.put(serverName, ++times);//访问后+1
        System.out.println("获取到的地址是：" + serverName + ", 访问次数：" + times);
        return serverName;
    }

    public static void main(String[] args) {
        test testLeastConnection =new test();
        for (int i =0;i<10;i++){
            testLeastConnection.leastConnection();
        }
    }
}
```

***

##### <font size="4" color="red">29. Java中视频添加水印</font>

```java
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
public class test {

    public static void main(String[] args) throws Exception {
        List<String> command = new ArrayList<>();
        command.add("F:\\ffmpeg\\bin\\ffmpeg.exe");
        command.add("-y");
        command.add("-i");
        command.add("E:\\xx.mp4");
        command.add("-vf");
        command.add("\"movie=xx.png");
        command.add("[logo];[in][logo]");
        command.add("overlay=x=5;y=5");
        command.add("[out]\"");
        command.add("E:\\xx.mp4");
        ProcessBuilder builder = new ProcessBuilder(command);
        Process process = builder.start();
        InputStream in = process.getErrorStream();
        byte[] re = new byte[1024];
        while(in.read(re) != -1){
            System.out.println(new String(re));
        }
        in.close();
        if(process.isAlive()){
            process.waitFor();
        }
    }
}
```

***

##### <font size="4" color="red">30. Java中Hbase创建表</font>

**依赖包：**

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.ColumnFamilyDescriptorBuilder.ModifyableColumnFamilyDescription;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.TableDescriptorBuilder.ModifyableTableDescriptor;

public class test {

    public static void main(String[] args) throws Exception {
        Confuration cnf = HBaseConfiguration.create();
        cnf.set("hbase.zookeeper.quorum","<zookeeper的ip地址>");
        cnf.set("hbase.zookeeper.property.clientPort","2181");
        cnf.set("zookeeper.znode.parent",'/hbase/master');
        Connection connection = ConnectionFactory.createConnection(cnf);
        //表名
        String tableName = "user";
        //列名
        String columnName = "info";
        Admin admin = connection.getAdmin();
        TableName table = TableName.valueOf(tableName);
        ModifyableTableDescriptor descriptor = new ModifyableTableDescriptor(table);
        ModifyableColumnFamilyDescriptor family = new ModifyableTableDescriptor(columnName.getBytes());
        descriptor.setColumnFamily(family);
        admin.createTable(description);
        admin.close();
        connection.close();
    }
}
```

****

##### <font size="4" color="red">31. Java中Hbase插入表数据</font>

**依赖包：**

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

```java
import org.apache.hadoop.cnf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.habse.util.Bytes;
public class test {

    public static void main(String[] args) throws Exception {
        Confuration cnf = HBaseConfiguration.create();
        cnf.set("hbase.zookeeper.quorum","<zookeeper的ip地址>");
        cnf.set("hbase.zookeeper.property.clientPort","2181");
        cnf.set("zookeeper.znode.parent",'/hbase/master');
        Connection connection = ConnectionFactory.createConnection(cnf);
        //表名
        String tableName = "user";
        //列族名
        String columnName = "info";
        String rowKey = "102";
        //列名
        String column = "name";
        //值内容
        String value = "test";
        Admin admin = connection.getAdmin();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put p = new Put(Bytes.toBytes(rowKey));
        p.addColumn(Bytes.toBytes(columnName),Bytes.toBytes(column),Bytes.toByte(value));
        table.put(p);
        admin.close();
        connection.close();
    }
}
```

***

##### <font size="4" color="red">32. Java中Hbase查询表数据</font>

**依赖包：**

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

```java
import org.apahce.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;
public class test {

    public static void main(String[] args) throws Exception {
        Confuration cnf = HBaseConfiguration.create();
        cnf.set("hbase.zookeeper.quorum","<zookeeper的ip地址>");
        cnf.set("hbase.zookeeper.property.clientPort","2181");
        cnf.set("zookeeper.znode.parent",'/hbase/master');
        Connection connection = ConnectionFactory.createConnection(cnf);
        //表名
        String tableName = "user";
        //列族名
        String columnName = "info";
        String rowKey = "102";
        //列名
        String column = "name";
        Admin admin = connection.getAdmin();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(rowKey.getBytes());
        get.addColumn(columnName.getBytes(),column.getBytes());
        Result res = table.get(get);
        String val = Bytes.toString(res.getValue(columnName.getBytes(),column.getBytes()));
        admin.close();
        connection.close();
    }
}
```

***

##### <font size="4" color="red">33. Java中Hbase删除数据</font>

**依赖包：**

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

```java
import org.apache.hadoop.cnf.Configuration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Table;
public class test {

    public static void main(String[] args) throws Exception {
        Confuration cnf = HBaseConfiguration.create();
        cnf.set("hbase.zookeeper.quorum","<zookeeper的ip地址>");
        cnf.set("hbase.zookeeper.property.clientPort","2181");
        cnf.set("zookeeper.znode.parent",'/hbase/master');
        Connection connection = ConnectionFactory.createConnection(cnf);
        //表名
        String tableName = "user";
        //列族名
        String columnName = "info";
        String rowKey = "102";
        //列名
        String column = "name";
        Admin admin = connection.getAdmin();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(rowKey.getBytes());
        delete.addColumn(columnName.getBytes(),column.getBytes());
        table.delete(delete);
        admin.close();
        connection.close();
    }
}
```

> **注：**更新key值使用put，rowKey，columnName不需要变更，更新value值即可

***

##### <font size="4" color="red">34. Java中Hbase删除表</font>

**依赖包：**

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Table;

public class test {

    public static void main(String[] args) throws Exception {
        Confuration cnf = HBaseConfiguration.create();
        cnf.set("hbase.zookeeper.quorum","<zookeeper的ip地址>");
        cnf.set("hbase.zookeeper.property.clientPort","2181");
        cnf.set("zookeeper.znode.parent",'/hbase/master');
        Connection connection = ConnectionFactory.createConnection(cnf);
        //表名
        String tableName = "user";
        Admin admin = connection.getAdmin();
        Table table = connection.getTable(TableName.valueOf(tableName));
        admin.disableTable(table);
        admin.deleteTable(table);
        admin.close();
        connection.close();
    }
}
```

***

##### <font size="4" color="red">35. Java中Hbase全表扫描</font>

**依赖包：**

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;
public class test {

    public static void main(String[] args) throws Exception {
        Confuration cnf = HBaseConfiguration.create();
        cnf.set("hbase.zookeeper.quorum","<zookeeper的ip地址>");
        cnf.set("hbase.zookeeper.property.clientPort","2181");
        cnf.set("zookeeper.znode.parent",'/hbase/master');
        Connection connection = ConnectionFactory.createConnection(cnf);
        //表名
        String tableName = "user";
        //列族名
        String columnName = "info";
        //列名
        String column = "name";
        Admin admin = connection.getAdmin();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        ResultScanner scanner = table.getScanner(scan);
        for(Result result : scanner){
            System.out.println(Bytes.toString(result.getValue(Bytes.toBytes(columnName),Bytes.toBytes(column))));
        }
        admin.close();
        connection.close();
    }
}
```

***

##### <font size="4" color="red">36. Java中word添加水印</font>

(1) 下载Spire.Doc.Free并引入

```java
import java.awt.Color;
import com.spire.doc.Document;
import com.spire.doc.FileFormat;
import com.spire.doc.PictureWatermark;
import com.spire.doc.TextWatermark;
import com.spire.doc.documents.WatermarkLayout;
public class test {

    public static void main(String[] args) {
        Document doc = new Document();
        doc.loadFromFile("D:\\xx.doc");
        TextWatermark textWatermark = new TextWatermark();
        //水印内容
        textWatermark.setText("水印内容");
        textWatermark.setFontName("宋体");
        textWatermark.setFontSize(30f);
        textWatermark.setColor(Color.RED);
        textWatermark.setLayout(WatermarkLayout.Diagonal);
        doc.setWatermark(textWatermark);
        dco.saveToFile("E:\\xx.doc",FileFormat.Docx_2013);
        //图片水印
        Document imgdoc = new Document();
        imgdoc.loadFromFile("E:\\xx.doc");
        PictureWatermark imageWatermark = new PictureWatermark();
        imageWatermark.setPicture("E:\\xx.jpg");
        imageWatermark.isWashout(false);
        imgdoc.setWatermark(imageWatermark);
        imgdoc.saveToFile("E:\\xx.doc",FileFormat.Docx_2013);
    }
}
```

***

##### <font size="4" color="red">37. Java中根据doc模版生成指定内容</font>

**依赖包：**

```xml
<dependency>
   <groupId>freemarker</groupId>
   <artifactId>freemarker</artifactId>
   <version>2.3.9</version>
</dependency>
```

**(1) 创建MDoc.java文件**

MDoc.java文件中内容：

```java
import java.io.BufferedWriter;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.io.UnsupportedEncodingException;
import java.io.Writer;
import java.util.Map;
import freemarker.template.Configuration;
import java.template.Template;
public class MDoc {

    private Configuration configuration = null;
    public MDoc() {

        configuration = new Configuration();
        configuration.setDefaultEncoding("utf-8");
    }

    public createDoc(Map<String,Object> dataMap,String fileName) throws Exception {
        //配置信息
        configuration.setClassForTemplateLoading(this.getClass(),"/");
        Template t = null;
        try{
            //加载模版
            t = configuration.getTemplate("test.ftl")
        }
        catch(Exception e){
            e.printStackTrace();
        }
        //输出文档名称
        File outFile = new File(fileName);
        Writer out = null;
        FileOutputStream fos = null;
        try{
            fos = new FileOutputStream(outFile);
            OutputStreamWriter oWriter = new OutputStreamWriter(fos,"utf-8");
            out = new BufferedWriter(oWriter);
        }
        catch(Exception e){
            e.printStackTrace();
        }
        try{
            t.process(dataMap,out);
            out.close();
            fos.close();
        }
        catch(Exception e){
            e.printStackTrace();
        }
    }  
}
```

(2) 创建test.java文件

test.java文件中内容：

```java
import java.util.HashMap;
import java.util.Map;
public class test{

    public static void main(String[] args) throws Exception {
        Map<String,Object> dataMap = new HashMap<String,Object>();
        dataMap.put("name","字符串");
        MDoc mdoc = new MDoc();
        mdoc.createDoc(dataMap,"E:\\xx.doc");
    }
}
```

(3) 将test.doc模版另存为test.xml文件，然后在test.xml文件中找到需要填充的地方编辑为${name}后保存成功后，改为test.ftl并在启动目录中创建resources文件夹，将test.ftl文件放入

***

##### <font size="4" color="red">38. Java中Pdf合并</font>

**依赖包：**

```xml
<dependency>
   <groupId>com.itextpdf</groupId>
   <artifactId>itext-asian</artifactId>
   <version>5.2.0</version>
</dependency>
<dependency>
   <groupId>com.lowgie</groupId>
   <artifactId>itext</artifactId>
   <version>2.1.7</version>
</dependency>
```

```java
import com.lowagie.text.Document;
import com.lowagie.text.pdf.PdfCopy;
import com.lowagie.text.pdf.PdfImportedPage;
import com.lowagie.text.pdf.PdfReader;

public class test {

    public static void mergePdfFiles(String[] files,String newfile) {
        Document document = null;
        try{
            document = new Document(new PdfReader(files[0]).getPageSize());
            PdfCopy copy = new PdfCopy(document,new FileOutputStream(newfile));
            document.open();
            for(int i=0;i<files.length;i++){
                PdfReader reader = new PdfReader(files[i]);
                int n = reader.getNumberOfPages();
                for(int j=1;j<=n;j++){
                    document.newPage();
                    PdfImportedPage page = copy.getImportedPage(reader,j);
                    copy.addPage(page);
                }
            }
        }
        catch(Exception e){
            e.printStackTrace();
        }
        finally{
            document.close();
        }
    }
    public static void main(String[] args) throws Exception {
        String[] files = {"E:\\xx.pdf","E:\\xx.pdf"};
        String savePath = "E:\\xx.pdf";//合并后生成的文件
        mergePdfFiles(files,savePath);
    }
}
```

***

##### <font size="4" color="red">40. Java中Coap服务器搭建</font>

**依赖包：**

```xml
<dependency>
   <groupId>org.eclipse.californium</groupId>
   <artifactId>californium-core</artifactId>
   <version>2.0.0-M15</version>
</dependency>
<dependency>
   <groupId>org.eclipse.californium</groupId>
   <artifactId>element-connector</artifactId>
   <version>2.0.0-M15</version>
</dependency>
<dependency>
    <groupId>org.eclipse.californium</groupId>
    <artifactId>scandium</artifactId>
    <version>2.0.0-M15</version>
</dependency>
```

**(服务器端)**

```java
import org.eclipse.californium.core.CoapResource;
import org.eclipse.californium.core.CoapServer;
import org.eclipse.californium.core.coap.CoAP.ResponseCode;
import org.eclipse.californium.core.server.resources.CoapExchange;
public class test {

    public static void main(String[] args) throws Exception {
        //默认端口5683
        CoapServer  server = new CoapServer(8050);
        //创建一个资源为hello格式为\hello
        server.add(new CoapResource("hello")){
            @Override
            public void handleGET(CoapExchange exchange) {
                //重写处理Get请求方法
                exchange.respond(ResponseCode.CONTENT,"字符串");
            }
        });

        server.add(new CoapResource("time")){
            @Override
            public void handleGET(CoapExchange exchange) {
                exchange.respond(ResponseCode.CONTENT,"字符串");
            }
        });
        //post请求
        server.add(new CoapResource("test",true)) {
            @Override
            public void handlePOST(CoapExchange exchange){
                String payload = exchange.getRequestText();
                byte[] payloadBytes = exchange.getRequestPayload();
                System.out.println("收到的数据：" + new String(payloadBytes));
                exchange.respond("字符串");
            }
            @Override
            public void handleGET(CoapExchange exchange) {
                System.out.println("端口：" + exchange.getSourcePort());
                System.out.println("编码：" + exchange.getRequestCode());
                exchange.respond(ResponseCode.CONTENT,"字符串");
            }
        });
        server.start();
    }
}
```

**(客户端)**

```java
import org.eclipse.californium.core.CoapClient;
import org.eclipse.californium.core.CoapResponse;
import org.eclipse.californium.core.coap.MediaTypeRegistry;
import org.eclipse.californium.core.Utils;
public class test {

    public static void main(String[] args) throws Exception {
        URI uri = new URI("coap://xxip地址:8050/hello");
        CoapClient client = new CoapClient(uri);
        StringBuilder sb = new StringBuilder();
        sb.append("字符串");
        //get请求
        // CoapResponse response = client.get();
        CoapResponse response = client.post(sb.toString(),MediaTypeRegistry.TEXT_DLAIN);
        if(respone != null){
            System.out.println(response.getPayload());
            System.out.println(response.getOptions());
            System.out.println(response.getResponseText());
            System.out.println(Utils.prettyPrint(response));
        }
    }
}
```

***

##### <font size="4" color="red">41. Java中操作Nsq</font>

**依赖包：**

```xml
<dependency>
   <groupId>com.sproutsocial</groupId>
   <artifactId>nsq-j</artifactId>
   <version>0.9.4</version>
</dependency>
<dependency>
   <groupId>com.google.code.gson</groupId>
   <artifactId>gson</artifactId>
   <version>2.8.5</version>
</dependency>
```

**(生产者)**

```java
import com.sproutsocial.nsq.Publisher;
public class test {

    public static void main(String[] args) {
        Publisher publisher = new Publisher("xxIp地址:4150");
        byte[] data = ("消息内容").getBytes();
        publisher.Publish("topic",data);
    }
}
```

**(消费者)**

```java
public class test implements MeesageHandler {

    @Override
    public void accept(Message paramMessage){
        //接收内容
        System.out.println(new Stirng(paramMessage.getData()));
        paramMessage.finish();
    }

    public static void main(String[] args){
        Subscriber subscriber = new Subscriber("xxIp地址");
        Test test = new Test();
        subscriber.subscribe("topic","flag",test);
    }
}
```

***

##### <font size="4" color="red">42. Java中生成二维码</font>

**1.带图片二维码生成**

**(1) 依赖包**

```xml
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>3.3.0</version>
</dependency>
<dependency>
   <groupId>com.google.zxing</groupId>
   <artifactId>javase</artifactId>
   <version>3.3.0</version>
</dependency>
```

**(2) 创建ImageSource.java文件**

ImageSource.java文件中内容：

```java
import java.awt.Graphics2D;
import java.awt.geom.AffineTransform;
import java.awt.image.BufferedImage;

import com.google.zxing.LuminanceSource;

public class ImageSource extends LuminanceSource{


    private final BufferedImage image;
    private final int left;
    private final int top;

    public ImageSource(BufferedImage image) {
        this(image, 0, 0, image.getWidth(), image.getHeight());
    }

    public ImageSource(BufferedImage image, int left, int top, int width, int height) {
        super(width, height);

        int sourceWidth = image.getWidth();
        int sourceHeight = image.getHeight();
        if (left + width > sourceWidth || top + height > sourceHeight) {
            throw new IllegalArgumentException("复制失败");
        }

        for (int y = top; y < top + height; y++) {
            for (int x = left; x < left + width; x++) {
                if ((image.getRGB(x, y) & 0xFF000000) == 0) {
                    //设置成白色
                    image.setRGB(x, y, 0xFFFFFFFF);
                }
            }
        }

        this.image = new BufferedImage(sourceWidth, sourceHeight, BufferedImage.TYPE_BYTE_GRAY);
        this.image.getGraphics().drawImage(image, 0, 0, null);
        this.left = left;
        this.top = top;
    }


    @Override
    public byte[] getRow(int y, byte[] row) {
        //从底层平台的位图提取一行（only one row）的亮度数据值
        if (y < 0 || y >= getHeight()) {
            throw new IllegalArgumentException("需要的行超出: " + y);
        }
        int width = getWidth();
        if (row == null || row.length < width) {
            row = new byte[width];
        }
        image.getRaster().getDataElements(left, top + y, width, 1, row);
        return row;
    }

    @Override
    public byte[] getMatrix() {
        ///从底层平台的位图提取亮度数据值
        int width = getWidth();
        int height = getHeight();
        int area = width * height;
        byte[] matrix = new byte[area];
        image.getRaster().getDataElements(left, top, width, height, matrix);
        return matrix;
    }

    @Override
    public boolean isCropSupported() {//是否支持裁剪
        return true;
    }

    /**
     * 返回一个新的对象与裁剪的图像数据。实现可以保存对原始数据的引用，而不是复制。
     */
    @Override
    public LuminanceSource crop(int left, int top, int width, int height) {
        return new BufferedImageLuminanceSource(image, this.left + left, this.top + top, width, height);
    }

    @Override
    public boolean isRotateSupported() {//是否支持旋转
        return true;
    }

    @Override
    public LuminanceSource rotateCounterClockwise() {//逆时针旋转图像数据的90度，返回一个新的对象。
        int sourceWidth = image.getWidth();
        int sourceHeight = image.getHeight();
        AffineTransform transform = new AffineTransform(0.0, -1.0, 1.0, 0.0, 0.0, sourceWidth);
        BufferedImage rotatedImage = new BufferedImage(sourceHeight, sourceWidth, BufferedImage.TYPE_BYTE_GRAY);
        Graphics2D g = rotatedImage.createGraphics();
        g.drawImage(image, transform, null);
        g.dispose();
        int width = getWidth();
        return new BufferedImageLuminanceSource(rotatedImage, top, sourceWidth - (left + width), getHeight(), width);
    }

}
```

**(3) 创建QRCodeUtil.java文件**

QRCodeUtil.java文件中内容：

```java
import java.awt.BasicStroke;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.Shape;
import java.awt.geom.RoundRectangle2D;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.OutputStream;
import java.util.Hashtable;
import javax.imageio.ImageIO;
import com.google.zxing.BarcodeFormat;
import com.google.zxing.BinaryBitmap;
import com.google.zxing.DecodeHintType;
import com.google.zxing.EncodeHintType;
import com.google.zxing.MultiFormatReader;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.Result;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.common.HybridBinarizer;
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel;

public class QRCodeUtil {

    private static final String CHARSET = "utf-8";
    private static final String FORMAT_NAME = "JPG";
    //二维码尺寸
    private static final int QRCODE_SIZE = 300;
    //LOGO宽度
    private static final int WIDTH = 60;
    //LOGO高度
    private static final int HEIGHT = 60;


    /**
     * 生成二维码
     */
    private static BufferedImage createImage(String content, String imgPath, boolean needCompress) throws Exception {
        @SuppressWarnings("rawtypes")
        Hashtable<EncodeHintType, Comparable> hints = new Hashtable<EncodeHintType, Comparable>();
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H);
        hints.put(EncodeHintType.CHARACTER_SET, CHARSET);
        hints.put(EncodeHintType.MARGIN, 1);
        BitMatrix bitMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, QRCODE_SIZE, QRCODE_SIZE,
                                                             hints);
        int width = bitMatrix.getWidth();
        int height = bitMatrix.getHeight();
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < width; x++) {
            for (int y = 0; y < height; y++) {
                image.setRGB(x, y, bitMatrix.get(x, y) ? 0xFF000000 : 0xFFFFFFFF);
            }
        }
        if (imgPath == null || "".equals(imgPath)) {
            return image;
        }
        // 插入图片
        QRCodeUtil.insertImage(image, imgPath, needCompress);
        return image;
    }

    /**
     * 在生成的二维码中插入图片
     */
    private static void insertImage(BufferedImage source, String imgPath, boolean needCompress) throws Exception {
        File file1 = new File(imgPath);
        if (!file1.exists()) {
            System.err.println("" + imgPath + "   该文件不存在！");
            return;
        }
        Image src = ImageIO.read(new File(imgPath));
        int width = src.getWidth(null);
        int height = src.getHeight(null);
        if (needCompress) { // 压缩LOGO
            if (width > WIDTH) {
                width = WIDTH;
            }
            if (height > HEIGHT) {
                height = HEIGHT;
            }
            Image image = src.getScaledInstance(width, height, Image.SCALE_SMOOTH);
            BufferedImage tag = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
            Graphics g = tag.getGraphics();
            g.drawImage(image, 0, 0, null); // 绘制缩小后的图
            g.dispose();
            src = image;
        }
        // 插入LOGO
        Graphics2D graph = source.createGraphics();
        int x = (QRCODE_SIZE - width) / 2;
        int y = (QRCODE_SIZE - height) / 2;
        graph.drawImage(src, x, y, width, height, null);
        Shape shape = new RoundRectangle2D.Float(x, y, width, width, 6, 6);
        graph.setStroke(new BasicStroke(3f));
        graph.draw(shape);
        graph.dispose();
    }

    /**
     * 生成带logo二维码，并保存到磁盘
     */
    public static void encode(String content, String imgPath, String destPath, boolean needCompress,String file) throws Exception {
        BufferedImage image = QRCodeUtil.createImage(content, imgPath, needCompress);
        mkdirs(destPath);

        ImageIO.write(image, FORMAT_NAME, new File(destPath + "/" + file));
    }

    public static void mkdirs(String destPath) {
        File file = new File(destPath);
        // 当文件夹不存在时，mkdirs会自动创建多层目录，区别于mkdir。(mkdir如果父目录不存在则会抛出异常)
        if (!file.exists() && !file.isDirectory()) {
            file.mkdirs();
        }
    }

    public static void encode(String content, String imgPath, String destPath,String file) throws Exception {
        QRCodeUtil.encode(content, imgPath, destPath, false ,file);
    }

    public static void encode(String content, String destPath, boolean needCompress,String file) throws Exception {
        QRCodeUtil.encode(content, null, destPath, needCompress,file);
    }

    public static void encode(String content, String destPath,String file) throws Exception {
        QRCodeUtil.encode(content, null, destPath, false,file);
    }

    public static void encode(String content, String imgPath, OutputStream output, boolean needCompress)
        throws Exception {
        BufferedImage image = QRCodeUtil.createImage(content, imgPath, needCompress);
        ImageIO.write(image, FORMAT_NAME, output);
    }

    public static void encode(String content, OutputStream output) throws Exception {
        QRCodeUtil.encode(content, null, output, false);
    }


    /**
     * 从二维码中，解析数据
     */
    public static String decode(File file) throws Exception {
        BufferedImage image;
        image = ImageIO.read(file);
        if (image == null) {
            return null;
        }
        BufferedImageLuminanceSource source = new BufferedImageLuminanceSource(image);
        BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
        Result result;
        Hashtable<DecodeHintType, String> hints = new Hashtable<DecodeHintType, String>();
        hints.put(DecodeHintType.CHARACTER_SET, CHARSET);
        result = new MultiFormatReader().decode(bitmap, hints);
        String resultStr = result.getText();
        return resultStr;
    }

    public static String decode(String path) throws Exception {
        return QRCodeUtil.decode(new File(path));
    }
}
```

**(4) 创建test.java文件**

test.java文件中内容：

```java
public class test {

    public static void main(String[] args) throws Exception {
        String textt = "www.baidu.com";//二维码的内容
        String logo = "E:\\storage\\logo.jpg";//logo的路径
        //生成二维码
        QRCodeUtil.encode(textt,logo,"E:\\",true,"test.jpg");
    }
}
```

**2.生成普通二维码**

```java
import com.google.zxing.BarcodeFormat;
import com.google.zxing.WriterException;
import com.google.zxing.client.j2se.MatrixToImageWriter;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.qrcode.QRCodeWriter;

public class test {

    private static void generateQRCodeImage(String text, int width, int height, String filePath) throws WriterException, IOException  {
        QRCodeWriter qrCodeWriter = new QRCodeWriter();
        BitMatrix bitMatrix = qrCodeWriter.encode(text, BarcodeFormat.QR_CODE, width, height);
        Path path = FileSystems.getDefault().getPath(filePath);
        MatrixToImageWriter.writeToPath(bitMatrix, "PNG", path);
    }

    public static void main(String[] args) throws Exception {
        //生成二维码
        generateQRCodeImage("This is my first QR Code", 350, 350, "E:\\test.png");
    }
}
```

***

##### <font size="4" color="red">43. Java中生成条形码</font>

**方法一：**

**(1) 依赖包**

```xml
<dependency>
    <groupId>net.sf.barcode4j</groupId>
    <artifactId>barcode4j-light</artifactId>
    <version>2.0</version>
</dependency>
```

**(2) 创建BarcodeUtil.java文件**

BarcodeUtil.java文件中内容：

```java
import java.awt.image.BufferedImage;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.OutputStream;
import org.apache.commons.lang.StringUtils;
import org.krysalis.barcode4j.impl.code39.Code39Bean;
import org.krysails.barcode4j.output.bitmap.BitmapCanvasProvider;
import org.krysails.barcode4j.tools.UnitConv;
public class BarcodeUtil {

    public static File generateFile(String msg,String path){
        File file = new File(path);
        try{
            generate(msg,new FileOutputStream(file));
        }
        catch(FileNotFoundException e){
            throw new RuntimeException(e);
        }
        return file;
    }

    //生成字节
    public static byte[] generate(String msg){
        ByteArrayOutputStream ous = new ByteArrayOutputStream();
        generate(msg,ous);
        return ous.toByteArray();
    }

    public static void generate(String msg,OutputStream ous){
        if(StringUtils.isEmpty(msg) || ous == null){
            return;
        }
        Code39Bean bean = new Code39Bean();
        //精细度
        final int dpi = 150;
        //module宽度
        final double moduleWidth = UnitConv.in2mm(1.0f/dpi);
        //配置对象
        bean.setModuleWidth(moduleWidth);
        bean.setWideFactor(3);
        bean.doQuietZone(false);
        String format = "image/png";
        try{
            //输出到流
            BitmapCanvasProvider canvas = new BitmapCanvasProvider(ous,format,dpi,BufferedImage.TYPE_BYTE_BINARY,false,0);
            //生成条形码
            bean.generateBarCode(canvas,msg);
            //结束绘制
            canvas.finish();
        }
        catch(IOException e){
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args){
        generateFile("xx","E:\\xx.png");
    }
}
```

**方法二：**

**(1) 依赖包**

```xml
<dependency>
   <groupId>com.google.zxing</groupId>
   <artifactId>javase</artifactId>
   <version>3.3.3</version>
</dependency>
```

```java
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.FileOutputStream;
import java.util.HashMap;
import java.util.Map;
import javax.imageio.ImageIO;
import com.google.zxing.BarcodeFormat;
import com.google.zxing.BinaryBitmap;
import com.google.zxing.DecodeHintType;
import com.google.zxing.LuminanceSource;
import com.google.zxing.MultiFormatReader;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.Result;
import com.google.zxing.WriterException;
import com.google.zxing.client.jzse.BufferedImageLuminanceSource;
import com.google.zxing.client.jzse.MatrixToImageWriter;
import com.google.zxing.common.bitMartix;
import com.google.zxing.common.HybridBinarizer;
public class test {
    //生成二维码、条形码
    public static void generateCode(File file,String code,int width,int height) {
        //定义位图
        BitMatrix matrix = null;
        try{
            //使用code_128格式进行编码
            MutliFormatWriter writer = new MutliFormatWriter();
            matrix = writer.encode(code,BarcodeFormat.CODE_128,width,height,null);
        }
        catch(WriterException e){
            e.printStackTrace();
        }
        //将位图保存成图片
        try(FileOutputStream outStream = new FileOutputStream(file)){
            ImageIO.write(MatrixToImageWriter.toBufferedImage(matrix),"png",outStream);
            outStream.flush();
        }
        catch(Exception e){
            e.printStackTrace();
        }
    }

    public static void readCode(File file){
        try{
            BufferedImage image = ImageIO.read(file);
            if(image == null){
                return;
            }
            LuminanceSource source = new BufferedImageLuminanceSource(image);
            BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
            Map<DecodeHintType,Object> hints = new HashMap<>();
            hints.put(DecodeHintType.CHARACTER_SET,"GBK");
            hints.put(DecodeHintType.PURE_BARCODE,Boolean.TRUE);
            hints.put(DecodeHintType.TRY_HARDER,Boolean.TRUE);
            Result result = new MultiFormatReader().decode(bitmap,hints);
            System.out.println(result.getText());
        }
        catch(Exception e){
            e.printStaceTrace();
        }
    }

    public static void main(String[] args) {
        //生成条形码
        generateCode(new File("D:\\xx.png"),"123",500,250);
        //识别二维码
        readCode(new File("xx.png"));
    }
}
```

***

##### <font size="4" color="red">44. Java中识别二维码</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>3.3.3</version>
</dependency>
```

```java
import com.google.zxing.*;
import com.google.zxing.client.jzse.BufferedImageLuminanceSource;
import com.google.zxing.client.jzse.MatrixToImageWriter;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.common.HybirdBinarizer;
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.nio.file.path;
import java.util.HashMap;
import java.util.Map;
public class test {

    public static void readQrCode(File file){
        MultiFormatReader reader = new MultiFormatReader();
        try{
            BufferedImage image = ImageIO.read(file);
            BinaryBitmap binaryBitmap = new BinaryBitmap(new HybridBinarizer(new BufferedImageLuminanceSource(image)));
            Map<DecodeHintType,Object> hints = new HashMap<>();
            //设置编码
            hints.put(DecodeHintType.CHARACTER_SET,"utf-8");
            Result result = reader.decode(binaryBitmap,hints);
            System.out.println(result.toString());
            System.out.println(result.getText());
        }
        catch(Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args){
        readQrCode(new File("xx.png"));
    }
}
```

***

##### <font size="4" color="red">45. Java中netty搭建TCP服务器</font>

**1. (服务器端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Socket.java文件**

Socket.java文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class Socket {

    public void bind(int port) throws Exception {
        /*服务线程组用于网络事件处理一个用户服务器断接收另一个socketChannel处理*/
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workGroup = new NioEventLoopGroup();
        try {
            /*Nio服务器端辅助启动类*/
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workGroup)
                            /*类似Nio中serverSocketChannel*/
                           .channel(NioServerSocketChannel.class)
                           //配置Tcp参数
                           .option(ChannelOption.SO_BACKLOG,1024)
                           /*配置延时*/
                           .childOption(ChannelOption.SO_KEEPALIVE,true)
                /*绑定IO处理类*/
                .childHandler(new ChildChannelHandler());
            /*服务启动后，绑定监听端口，同步等待成功，主要用于异步处理回调*/
            ChannelFuture f= serverBootstrap.bind(port).sync();
            /*等待端口关闭*/
            f.channel().closeFuture().sync();
        }
        finally{
            /*退出线程资源*/
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        new Socket().binf(8010);        
    }
}
```

**(3) 创建ChildChannelHandler.java文件**

ChildChannelHandler.java文件中内容：

```java
import io.netty.channel.ChannelFuture;
import io.nety.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;

public class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

    protected void initChannel(SocketChannel ch){
          //下发消息
        ch.pipeline().addLast(new ServerHandler());
        ChannelFuture f = ch.closeFuture();
        f.addListeners(new ChannelFutureListener(){

            public void operationComplete(ChannelFuture cf){
                /*Socket关闭操作*/
                Channel ch = cf.channel();
                String remoteAddress = ch.remoteAddress().toString();
                System.out.println(remoteAddress);
            }
        });
    }
}
```

**(4) 创建ServerHandler.java文件**

ServerHandler.java文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.ChannelHandlerContext;

public class ServerHandler extends SimpleChannelInboundHandler<ByteBuf> {

    protected void channelRead0(ChannelHandlerContext ctx ,ByteBuf msg) throws Exception {
        /*
          ctx.write("内容");
          ctx.flush();
        */
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req,"UTF-8");
        /*获取远程客户端地址*/
        ctx.channel().writeAndFlush("内容");
        String remoteAddress = ctx.channel().remoteAddress();
        System.out.println(remoteAddress);
        //转十六进制
        String str = ByteBufUtil.hexDump(buf).toLowerCase();
        System.out.println(str);
    }


    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx = null;
    }
}
```

> **注：**
> 
> 要使用发送自定义字符串使用以下方式
> 
> ```java
> String str = "字符串";
> ByteBuf resp = Unpooled.copiedBuffer(str.getBytes());
> ctx.writeAndFlush(resp);
> ```

**2. (客户端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Client.java文件**

Client.java文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.Socket.nio.NioSocketChannel;

public class Client {

    public void connect(int port,String host) throws Exception {
        /*配置线程组*/
        EventLoopGroup group = new EventLoopGroup();
        try{
            Bootstrap b = new Bootstrap();
            b.group(group).channel(NioSocketChannel.class)
                          .option(ChannelOption.TCP_NODELAY,true)
                          .handler(new childChannelHandler());
            ChannelFuture f = b.connect(host,port).sync();
            channel ch = f.channel().closeFuture().sync().Channel();
            ch.writeAndFlush(Unpooled.copiedBuffer("xx",CharsetUtil.UTF_8));
        }
        finally{
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new Client().connect(8023,"xxIP地址");
    }
}
```

**(3) 创建childChannelHandler.java文件**

childChannelHandler.java文件中内容：

```java
import java.util.concurrent.TimeOnit;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.socket.SocketChannel;
import io.netty.handler,timeout.IdleStateHandler;
public class childChannelHandler extends ChannelInitializer<SocketChannel>{

    protected void initChannel(SocketChannel ch){
        /*添加心跳检测*/
        ch.pipeline().add(new IdleStateHandler(15,0,0,TimeUnit.SECONDS))
        /*心跳处理*/
        ch.pipeline().addLast(new HeartBeatServerHandler());
        /*连接下发消息*/
        ch.pipeline().addLast(new ClientHandler());
        ChanelFuture f = ch.closeFuture();
        f.addListeners(new ChannelFutureListener(){

            public void operationComplete(ChannelFuture cf){
                /*Socket关闭操作*/
                Channel ch = cf.channel();
                String remoteAddress = ch.remoteAddress().toString();
                System.out.println(remoteAddress);
            }
       });
    }
}
```

**(4) 创建serverHandler.java文件**

serverHandler.java文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class ClientHandler extends SimpleChannelInboundHandler<ByteBuf>{

    protected void channelRead0(ChannelHandlerContext ctx ,ByteBuf msg) throws Exception {
        /*
          ctx.write("内容");
          ctx.flush();
        */
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req,'UTF-8');
        /*获取远程客户端地址*/
        ctx.channel().writeAndFlush("内容");
        String remoteAddress = ctx.channel().remoteAddress();
        System.out.println(remoteAddress);
    }

     public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx.allocc();
    }
}
```

**(5) 创建HeartBeatServerHandler.java文件**

HeartBeatServerHandler.java文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.handler.timeout.IdleStateEvent;

public class HeartBeatServerHandler extends ChannelInboundHandlerAdapter {

    public void useEventTriggered(ChannelHandlerContext ctx,Object evt) throws Exception{

        if(evt instanceof IdelStateEvent){
            ctx.channel().close();
        }
    }

     public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx.alloc();
    }
}
```

> **注：**心跳检测可以加入netty中TCP和UDP机制中

***

##### <font size="4" color="red">46. Java中netty搭建UDP服务器</font>

**1. (服务器端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Server.java文件**

Server.java文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import java.net.InetSocketAddress;
import io.netty.channel.Channel;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.DatagramPacket;
import io.netty.channel.socket.nio.NioDatagramChannel;

public class Server {

    public void bind(int port) throws Exception {
        EventLoopGroup workGroup = new NioEventLoopGroup();
        try{
            Bootstrap serverbootstrap = new Bootstrap();
            serverbootstrap.group(workgroup)
                           .channel(NioDatagramChannel.class)
                           .option(ChannelOption.SO_BROADCAST,true)
                           //设置udp读取缓冲区
                           .option(ChannelOption.SO_REVBUF,1024 * 1024)
                           //设置udp缓冲区
                           .option(ChannelOption.SO_SNDBUF,1024 *1024)
                           .handler(new UdpChannelHandler());
            /*绑定监听端口*/
            ChannelFuture f = serverbootstrap.bind(port).sync();
            f.channel().closeFuture().sync().await();
        }
        finally{
            workgroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new Server().bind(8320);
    }
}
```

**(3) 创建UdpChannelHandler.java文件**

UdpChannelHandler.java文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.nio.NioDatagramChannel;

public class UdpChannelHandler extends ChannelInitializer<NioDatagramChannel> {

    protected void initChannel(NioDatagramChannel ch) throws Exception {
       ch.pipeline().addLast(new ServerHandler());        
    }
}
```

**(4) 创建ServerHandler.java文件**

ServerHandler.java文件中内容：

```java
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.util.CharsetUtil;

public class ServerHandler extends SimpleChannelInboundHandler<DatagramPacket>{

    protected void channelRead0(ChannelHandlerContext ctx,DatagramPacket msg) throws Exception {
        //文件内容
        ByteBuf buf = dp.copy().content();
        //数据起始位
        int _protocol = buf.getByte(0);
        //释放对象
        ReferenceCountUtil.release(buf);
        ctx.writeAndFlush(new DatagramPacket(Unpooled.copiedBuffer("字符串"),CharsetUtil.UTF_8),msg.sender());
    }
}  
```

**2. (客户端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建UdpClient.java文件**

UdpClient.java文件中内容：

```java
import java.net.InetSocketAddress;
import io.netty.bootstrap.Bootstrap;
import io.netty.util.CharsetUtil;
import io.netty.buffer.Unpooled;
import io.netty.channel.Channel;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;

public class UdpClient {

    public void bind() throws Exception {

        EventLoopGroup wrokGroup = new NioEventLoopGroup();
        try{
            Bootstrap serverbootstrap = new Bootstrap();
            serverbootstrap.group(workGroup)
                           .channel(NioDatagramChannel.class)
                           .option(ChannelOption.SO_BROADCAST,true)
                           .handler(new UdpChannelHandler());
            Channel ch = serverbootstrap.bind(0).sysnc().channel();
            ch.writeAndFlush(new DatagramPacket(Unpooled.copiedBuffer("字符串",CharsetUtil.UTF_8)),new InetSocketAddress("xxIP地址",8320)).sync();
            ch.closeFuture().sync().await();
        }
        finally{
            workGroup.shutdownGracefully();
        }
    }

     public static void main(String[] args) throws Exception {
        new UdpClient().bind();
    }
}
```

**(2) 创建UdpChannelHandler.java文件**

UdpChannelHandler.java文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.socket.DatagramPacket;
import io.netty.util.CharsetUtil;

public class UdpChannelHandler extends SimpleChannelInboundHandler<DatagramPacket> {

    protected void channelRead0(ChannelHandlerContext ctx,DatagramPacket msg){
        ctx.writeAndFlush(new DatagramPacket(Unpooled.copiedBuffer("字符串"),CharsetUtil.UTF_8),msg.sender());
    }
}
```

***

##### <font size="4" color="red">47. Java中netty搭建文本服务器(普通实现)</font>

**1. (服务器端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Socket.java文件**

Socket.java文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class Socket {

    public void bind(int port) throws Exception {
        /*服务线程组用于网络事件处理一个用户服务器断接收另一个socketChannel处理*/
        //线程数
        int boss_os_num = 8;
        int event_os_num = 32;
        EventLoopGroup bossGroup = new NioEventLoopGroup(boss_os_num);
        EventLoopGroup workGroup = new NioEventLoopGroup(event_os_num);
        try {
            /*Nio服务器端辅助启动类*/
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workGroup)
                            /*类似Nio中serverSocketChannel*/
                           .channel(NioServerSocketChannel.class)
                           //配置Tcp参数
                           .option(ChannelOption.SO_BACKLOG,1024 * 1024)
                           /*配置延时*/
                           .childOption(ChannelOption.SO_KEEPALIVE,true)
                /*绑定IO处理类*/
                .childHandler(new ChildChannelHandler());
            /*服务启动后，绑定监听端口，同步等待成功，主要用于异步处理回调*/
            ChannelFuture f= serverBootstrap.bind(port).sync();
            /*等待端口关闭*/
            f.channel().closeFuture().sync();
        }
        finally{
            /*退出线程资源*/
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        new Socket().binf(8010);        
    }
}
```

**(3) 创建ChildChannelHandler.java文件**

ChildChannelHandler.java文件中内容：

```java
import io.netty.channel.ChannelFuture;
import io.nety.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.ChannelFuture;
import io.netty.handler.codec.serialization.ClassResolvers;
import io.netty.handler.codec.serialization.ObjectDecoder;
import io.netty.handler.codec.serialization.ObjectEncoder;

public class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

    protected void initChannel(SocketChannel sch){
        sch.pipeline().addLast(new ObjectEncoder())
                      .addLast(new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())))
                      .addLast(new ServerHandler());
        ChannelFuture scf = sch.closeFuture();
        scf.addListener(new ChannelFutureListener(){

            public void operationComplete(ChannelFuture cf) throws Exception {
                Channel ch = cf.channel();
                String remote_addr = ch.remoteAddress().toString();
                System.out.println(remote_addr);
            }
        });
    }
}
```

**(4) 创建ServerHandler.java文件**

ServerHandler.java文件中内容：

```java
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.ChannelHandlerContext;

public class ServerHandler extends SimpleChannelInboundHandler<Object> {

    protected void channelRead0(ChannelHandlerContext ctx ,Object msg) throws Exception {
        String str = (String) msg;
        System.out.println(str);
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx = null;
    }
}
```

**2. (客户端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Client.java文件**

Client.java文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.Socket.nio.NioSocketChannel;

public class Client {

    public void connect(int port,String host) throws Exception {
        /*配置线程组*/
        EventLoopGroup group = new EventLoopGroup();
        try{
            Bootstrap b = new Bootstrap();
            b.group(group).channel(NioSocketChannel.class)
                         .option(ChannelOption.TCP_KEEPALIVE,true)
                          .option(ChannelOption.TCP_NODELY,true)
                          .handler(new childChannelHandler());
            ChannelFuture f = b.connect(host,port).sync();
            channel ch = f.channel().closeFuture().sync().Channel();
            ch.writeAndFlush("字符串");
        }
        finally{
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new Client().connect(8023,"xxIP地址");
    }
}
```

**(3) 创建childChannelHandler.java文件**

childChannelHandler.java文件中内容：

```java
import java.util.concurrent.TimeOnit;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.socket.SocketChannel;
import io.netty.handler.imeout.IdleStateHandler;

public class childChannelHandler extends ChannelInitializer<SocketChannel>{

    protected void initChannel(SocketChannel ch){
        /*添加心跳检测*/
        ch.pipeline().add(new IdleStateHandler(15,0,0,TimeUnit.SECONDS))
        /*连接下发消息*/
        ch.pipeline().addLast(new ObjectEncoder())
                     .addLast(new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())))
                     .addLast(new clientHandler());
        ChanelFuture f = ch.closeFuture();
        f.addListeners(new ChannelFutureListener(){

            public void operationComplete(ChannelFuture cf){
                /*Socket关闭操作*/
                Channel ch = cf.channel();
                if(ch.remoteAddress != null){
                    String remoteAddress = ch.remoteAddress().toString();
                    System.out.println(remoteAddress);
                }
            }
       });
    }
}
```

**(4) 创建clientHandler.java文件**

clientHandler.java文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.EventLoop;
import io.netty.channel.handler.timeout.IdleStateEvent;

public class ClientHandler extends SimpleChannelInboundHandler<Object>{

    protected void channelRead0(ChannelHandlerContext ctx ,Object msg) throws Exception {
        String str = (String) msg;
        /*获取远程客户端地址*/
        ctx.writeAndFlush(str);
        String remoteAddress = ctx.channel().remoteAddress();
        System.out.println(remoteAddress);
    }

    //掉线重连
    public void  channelInactive(ChannelHandlerContext ctx) throw Exception {
        final EventLoop eventLoop = ctx.channel.eventLoop();
        eventLoop.schedule(new Runnable(){

            public void run(){
                eventLoop.shutdownGracefully();
                try{
                    //重连操作
                }
                catch(Exception e){

                }
            }
        },1L,TimeUnit.SECONDS);
        super.channelInactive(ctx);
    }

    //心跳检查
    public void userEventTriggered(ChannelHandlerContext ctx,Object evt) throws Exception {
        if(evt instanceof IdleStateEvent){
            //发送心跳数据
        }
        else{
            super.userEventTriggered(ctx,evt);
        }
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx = null;
    }
}
```

> **注：**心跳检测可以加入netty中TCP和UDP机制中

***

##### <font size="4" color="red">48. Java中netty搭建文本服务器(连接池)</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Socket.java文件**

Socket.java文件中内容：

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class Socket {

    public void bind(int port) throws Exception {
        /*服务线程组用于网络事件处理一个用户服务器断接收另一个socketChannel处理*/
        //线程数
        int boss_os_num = 8;
        int event_os_num = 32;
        EventLoopGroup bossGroup = new NioEventLoopGroup(boss_os_num);
        EventLoopGroup workGroup = new NioEventLoopGroup(event_os_num);
        try {
            /*Nio服务器端辅助启动类*/
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workGroup)
                            /*类似Nio中serverSocketChannel*/
                           .channel(NioServerSocketChannel.class)
                           //配置Tcp参数
                           .option(ChannelOption.SO_BACKLOG,1024 * 1024)
                           /*配置延时*/
                           .childOption(ChannelOption.SO_KEEPALIVE,true)
                           /*绑定IO处理类*/
                           .childHandler(new ChildChannelHandler());
            /*服务启动后，绑定监听端口，同步等待成功，主要用于异步处理回调*/
            ChannelFuture f= serverBootstrap.bind(port).sync();
            /*等待端口关闭*/
            f.channel().closeFuture().sync();
        }
        finally{
            /*退出线程资源*/
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        new Socket().binf(8010);        
    }
}
```

**(3) 创建ChildChannelHandler.java文件**

ChildChannelHandler.java文件中内容：

```java
import io.netty.channel.ChannelFuture;
import io.nety.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.ChannelFuture;
import io.netty.handler.codec.serialization.ClassResolvers;
import io.netty.handler.codec.serialization.ObjectDecoder;
import io.netty.handler.codec.serialization.ObjectEncoder;

public class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

    protected void initChannel(SocketChannel sch){
        sch.pipeline().addLast(new ObjectEncoder())
                      .addLast(new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())))
                      .addLast(new ServerHandler());
        ChannelFuture scf = sch.closeFuture();
        scf.addListener(new ChannelFutureListener(){

            public void operationComplete(ChannelFuture cf) throws Exception {
                Channel ch = cf.channel();
                String remote_addr = ch.remoteAddress().toString();
                System.out.println(remote_addr);
            }
        });
    }
}
```

**(4) 创建ServerHandler.java文件**

ServerHandler.java文件中内容：

```java
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.ChannelHandlerContext;

public class ServerHandler extends SimpleChannelInboundHandler<Object> {

    protected void channelRead0(ChannelHandlerContext ctx ,Object msg) throws Exception {
        String str = (String) msg;
        System.out.println(str);
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx = null;
    }
}
```

**2. (客户端)**

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建Client.java文件**

Client.java文件中内容：

```java
import io.netty.channel.Channel;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.pool.AbstractChannelPoolMap;
import io.netty.channel.pool.ChannelPoolHandler;
import io.netty.channel.pool.FixedChannelPool;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.serialization.ClassResolvers;
import io.netty.handler.codec.serialization.ObjectDecoder;
import io.netty.handler.codec.serialization.ObjectEncoder;
import java.net.InetAddress;
import io.netty.bootstrap.Bootstrap;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.FutureListener;

public class Client {

    //连接池对象
    public static ChannelPoolMap<String, FixedChannelPool> tra_pool = null;

    public void connect(int port,String host) throws Exception {
        /*配置线程组*/
        EventLoopGroup group = new EventLoopGroup();
        try{
            //Tcp管理线程组
            NioEventLoopGroup eventLoop = new NioEventLoopGroup();
            //配置netty中ServerBootstrap对象
            Bootstrap clientBoot = new Bootstrap();
            //配置Tcp参数
            clientBoot.group(eventLoop)
                      .channel(NioSocketChannel.class)
                      .option(ChannelOption.TCP_NODELAY, true)
                      .option(ChannelOption.SO_KEEPALIVE , true)
                      .remoteAddress(host,port);
            //检测对象
            if(Tracker.ota_tra_pool == null) {
                //初始化连接池
                tra_pool = new AbstractChannelPoolMap<String, FixedChannelPool>(){
                    @Override
                    protected FixedChannelPool newPool(String key) {
                        //连接池操作类
                        ChannelPoolHandler handler = new ChannelPoolHandler() {
                            @Override
                            public void channelReleased(Channel ch) throws Exception {
                            }
                            @Override
                            public void channelAcquired(Channel ch) throws Exception {

                            }
                            @Override
                            public void channelCreated(Channel ch) throws Exception {
                                ch.pipeline()
                                  .addLast(new ObjectEncoder())
                                  .addLast(new ObjectDecoder(ClassResolvers.cacheDisabled(this.getClass().getClassLoader())))
                                  .addLast(new clientHandler());
                            }
                        };
                        //单个连接池大小
                        return new FixedChannelPool(clientBoot, handler, 50);
                    }
                };
            }
            //释放对象
            _host = null;
            _port = null;    

        }
        catch(Exception e) {

        }
    }

    public static void main(String[] args) throws Exception {
        new Client().connect(8023,"xxIP地址");
        //获取tcp线程池
        FixedChannelPool ota_pool = Tracker.ota_tra_pool.get("xxIP地址");
        //获取连接池客户端
        Future<Channel> future = ota_pool.acquire();
        //添加监听事件
        future.addListener(new FutureListener<Channel>() {
            @Override
            public void operationComplete(Future<Channel> future) throws Exception {
                //给服务端发送数据
                Channel channel = future.getNow();
                //转发数据
                channel.writeAndFlush("字符串");
                //回收线程
                ota_pool.release(channel);
            }
        });
    }
}
```

**(4) 创建clientHandler.java文件**

clientHandler.java文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.EventLoop;
import io.netty.channel.handler.timeout.IdleStateEvent;

public class ClientHandler extends SimpleChannelInboundHandler<Object>{

    protected void channelRead0(ChannelHandlerContext ctx ,Object msg) throws Exception {
        String str = (String) msg;
        /*获取远程客户端地址*/
        ctx.writeAndFlush(str);
        String remoteAddress = ctx.channel().remoteAddress();
        System.out.println(remoteAddress);
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
        ctx = null;
    }
}
```

> **注：**心跳检测可以加入netty中TCP和UDP机制中

***

##### <font size="4" color="red">49. Java中Mqtt使用</font>

**依赖包**

```xml
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.0</version>
</dependency>
```

**1.生产者**

**(1) 创建Producer.java文件**

Producer.java文件中内容：

```java
import org.eclipse.paho.client.mqttv3.MqttAsyncClient;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.persist.MqttDefaultFilePersistence;
import org.eclipse.paho.client.mqttv3.persist.MqttMemoryPersistence;

public class Producer {

    public static void main(String[] args){
        //订阅id
        String topic = "topic";
        String content = "消息内容";
        int qos = 1;
        //多个地址用逗号隔开tcp://xx:1883,xx:1884,xx:1885
        String broker = "tcp://xIP地址:1937";
        String username = "xx用户名";
        String password = "xx密码";
        String clientId = "id标识";
        try{
            //内存存储
            MemoryPersistence  persitence = new MemoryPersistence();
            //或
            //MqttDefaultFilePersistence persitence = new MqttDefaultFilePersistence();
            MqttClient sampleClient = new MqttClient(broker,clientId,persitence);
            //创建连接参数
            MqttConnectOptions connOps = new MqttConnectOptions();
            //重新启动重连时记住状态
            connOps.setCleanSession(false);
            //设置用户名
            connOps.setUserName(username);
            //设置密码
            connOps.setPassword(password.toCharArray());
            //建立连接
            sampleClient.connect(connOps);
            //创建消息
            MqttMessage message = new MqttMessage(content.getBytes());
            //设置消息服务质量
            message.setQos(qos);
            //发布消息
            sampleClient.publish(topic,message);
            //断开连接
            sampleClient.disconnect();
            //关闭客户端
            sampleClient.close();
        }
        catch(Exception e){

        }
    }
}
```

**2.订阅端口**

**(1) 创建`mqttHandler.java`文件**

`mqttHandler.java`文件中内容：

```java
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttMessage;

public class mqttHandler {

    public void connectionLost(Throwable cause) {
        //重连操作
    }

    public void messageArrived(String topic, MqttMessage mqtt_msg) {
        //接收mqtt订阅消息
        String message = new String(mqtt_msg.getPayload());
        System.out.println(topic);
        System.out.println(message.getQos());
        System.out.println(new String(message.getPayload()));
    }

    public void deliveryComplete(IMqttDeliveryToken token) {

    }
}
```

**(2) 创建`Consumer.java`文件**

Consumer.java文件中内容：

```java
import eclipse.paho.client.mqttv3.MqttAsyncClient;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.persist.MqttMemoryPersistence;

public class Consumer {

    String host = "tcp:xxIP地址//:1937";
    String topic = "topic值";
    String qos = 1;
    String username = "xx用户名";
    String password = "xx密码";
    String clientId = "id标识";
    try{
        //内存存储
        MemoryPersistence  mqttMemory = new MemoryPersistence();
        //默认为以内存保存
        MqttClient mqttClient = new MqttClient(host,cliendId,mqttMemory);
        //MQTT的连接设置
        MqttConnectOptions options = new MqttConnectOptions();
        //这里设置为true表示每次连接到服务器都以新的身份连接
        options.setCleanSession(false);
        //设置连接的用户名
        options.setUserName(username);
        //设置连接的密码
        options.setPassword(password.toCharArray());
        //设置超时时间 单位为秒
        options.setConnectionTimeout(10000);
        //设置会话心跳时间 单位为秒 服务器会每隔1.5*20秒的时间向客户端发送个消息判断客户端是否在线，但这个方法并没有重连的机制
        options.setKeepAliveInterval(20);
        //设置回调函数
        mqttClient.setCallback(new mqttHandler());
        //建立mqtt连接
        mqttClient.connect(options);
        //订阅消息
        mqttClient.subscribe(topic, qos);
    }
    catch(Exception e){

    } 
}
```

> **注：**
> 
> `MqttClient`：同步调用客户端，使用阻塞的方式与mqtt服务器沟通
> 
> `MqttAsyncClient`：异步调用客户端，使用非阻塞方式与mqtt服务器通信

****

##### <font size="4" color="red">50. Java中netty搭建websocket服务器端(wss协议)</font>

**(1) jdk中使用keytool工具生成签名**

```shell
keytool -gentkey -keysize 2048 -validity 365 -keypass netty123 -storepass netty123 -keystore wss.jks
```

**(2) netty搭建Websocket服务器端handler类**

```java
public static SSLContext createSSLContext(String type,String path,String password) throws Exception {
    keyStore ks = keyStore.getInstance("JKS");
    InputStream ksInpustStream = new FileInputStream(path);
    ks.load(ksInputStream,password.toCharArray());
    keyManagerFactory kmf = keyManagerFactory.getInstance(keyManagerFactory.getDefaultAlgorithm());
    kmf.init(ks,password.toCharArray());
    //安全套接字协议
    SSLContext sslContext = SSLContext.getInstance("TLS");
    sslContext.init(kmf.getKeyManager(),null,null);
    return sslContext;
}

public void initChannel(SocketChannel ch) throws Exception {
    SSLContext sslContext = createSSLContext("JKS","wss.jks","netty123");
    SSLEngine engine = sslContext.createSSLEngine();
    engine.setUseClientMode(false);
    ch.pipline().addLast(new SslHeader(engine));
    ch.pipline().addLast(new IdleStateHandler(5,0,0,TineUint.SECONDS));
    ch.pipeline().addLast(new HttpServerCodec());
    ch.pipeline().addLast(new HttpObjectAggregator(65536));
    ch.pipeline().addLast(new ChunkedWriteHandler());
    ch.pipeline().addLast(new AcceptorIdleStateTrigger());
    ch.pipeline().addLast(new websocketHandler());
}
```

****

##### <font size="4" color="red">51. Java中Netty客户端连接池</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(2) 创建MyNettyPool.java文件**

MyNettyPool.java文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.pool.AbstractChannelPoolMap;
import io.netty.channel.pool.ChannelPoolHandler;
import io.netty.channel.pool.ChannelPool;
import io.netty.channel.socket.nio.FixedChannelPool;
import io.netty.channel.pool.socket.nio.NioSocketChannel;
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.Delimiters;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.FutureListener;

public class MyNettyPool {

    public static ChannelPoolMap<String,FixedChannelPool> poolMap = null;
    private static final Bootstrap = new Bootstrap();
    static {
        bootstrap.group(new NioEventLoopGroup())
                 .channel(NioSocketChannel.class)
                 .remoteAddress("localhost",8080);
    }

    public MyNettyPool(){
        init();
    }

    public void init(){
        poolMap = new AbstractChannelPoolMap<String,FixedChannelPool>(){

            @Override
            protected FixedChannelPool newPool(String key) {
                ChannelPoolHandler handler = new ChannelPoolHandler(){

                    @Override
                    public void channelReleased(Channel ch) throws Exception {
                        //刷新数据
                    }
                    //添加handler
                    @Override
                    public void channelCreated(Channel ch) throws Exception {
                        ch.pipeline().addLast(new StringEncoder())
                                     .addLast(new StringDecoder())
                                     .addLast(new NettyClientHandler());
                    }

                    @Override
                    public void channelAcquiredd(Channel ch) throws Exception {
                        //获取连接池channel
                    }
                };
                //设置连接池大小
                return new FixedChannelPool(bootstrap,handler,50)
            }
        };
    }
    //发送消息
    public void send(String msg){
        //从连接池获取连接，远程连接域名作为key值
        FixedChannelPool pool = poolMap.get("localhost");
        //没有申请或网络断开返回null
        Future<Channel> future = pool.acquire();
        future.addListener(new FutureListener<Channel>(){
            @Override
            public void operationComplete(Future<Channel> future) throws Exception {
                //发送数据
                Channel channel = future.getNow();
                channel.writeAndFlush(msg);
                //回收线程池
                pool.release(channel);
            }
        });
    }
}
```

**(3) 创建NettyClientHandler.java文件**

NettyClientHandler.java文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufUtil;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
public class NettyClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx,ByteBuf buf) throws Exception {
        String hext_str = ByteBufUtil.hexDump(buf);
        //接收服务器端消息
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //完成消息接收操作
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        ctx.close();
        ctx = null;
    }
}
```

> **注：**
> 
> (1) 连接池大小需设定在合理范围，否则会导致服务器`cpu`损耗增加
> 
> (2)`SimpleChannelPool`实现`ChannelPool`接口，简单的连接池实现
> 
> (3)`FixedChannelPool`继承`SimpleChannelPool`，有大小限制的连接池实现
> 
> (4) `ChannelPoolMap`管理`host`与连接池映射的接口
> 
> (5) `AbstractChannelPoolMap`抽象类，实现`ChannelPoolMap`接口

****

##### <font size="4" color="red">52. Java中Netty客户端连接</font>

**(1) 依赖包**

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.45.Final</version>
</dependency>
```

**(1) 创建TcpClient.java文件**

TcpClient.java文件中内容：

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
public class TcpClient {

    public void clientStart(){
        try{
            String host = "127.0.0.1";
            int port = 8006;
            //tcp管理线程
            NioEventLoopGroup eventLoop = new NioEventLoopGroup();
            //配置netty中serverBoot对象
            Bootstrap clientBoot = new Bootstrap();
            //配置tcp参数
            clientBoot.group(eventLoop)
                      .channel(NioSocketChannel.class)
                      .option(ChannelOption.TCP_NODELAY,true)
                      .option(ChannelOption.SO_KEEPALIVE,true)
                      .handler(new ChannelInitializer<SocketChannel>{
                          @Override
                          protected void initChannel(SocketChannel ch) throws Exception {
                              ChannelPipeline p = ch.pipeline();
                              ChannelFuture scf = ch.closeFuture();
                              p.addLast(new TcpHandler());
                              //添加关闭事件
                              scf.addListener(new ChannelFutureListener(){
                                  @Override
                                  public void operationComplete(ChannelFuture cf) throws Exception {
                                      Channel ch = cf.channel();
                                      //断开连接操作区域
                                  }
                              });
                          }
                      });
            //连接服务器端
           ChannelFuture cf = clientBoot.connect(host,port).sync();
            /*for(init i=0;i<2;i++){
                //建立连接
                ChannelFuture sync = clientBoot.connect(host,port);
                //异步阻塞
                /*sync.channel().closeFuture().addListener((r)->{
                    System.out.println("shutdown");
                    eventLoop.shutdownGracefully();
                });*/
            }*/
        }
        catch(Exception e){
            e.printStackTrace();
        }
    }
}
```

**(2) 创建TcpHandler.java文件**

TcpHandler.java文件中内容：

```java
import io.netty.buffer.Bytebuf;
import io.netty.buffer.ByteBufUtil;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
public class TcpHandler extends SimpleChannelInboundHandler<Bytebuf> {
    //字符串缓冲区
    private StringBuilder buf_hex = null;

    @Override
    protected void channelRead0(ChannelHandler ctx,Bytebuf  buf) throws Exception {
        String hex_str = ByteBufUtil.hexDump(buf).toLowerCase();
        if(buf_hex == null) {
            buf_hex = new StringBuilder();
        }
        buf_hex.append(hex_str);
        buf.clear();
        buf = null;
    }
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        if(buf_hex != null){
            String hex_str = buf_hex.toString();
            System.out.println(hex_str);
        }
        buf_hex.delete(0,buf_hex.length());
        buf_hex = null;
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception {
        ctx.close();
        ctx = null;
    }
}
```

> **注：**
> 
> (1) 依赖出发时建立连接
> 
> ```java
> ChannelFuture cf = clientBoot.connect("127.0.0.1",8008).sync();
> ```

***

##### <font size="4" color="red">53. Java中图片合成</font>

```java
import java.awt.Color;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.RenderingHints;
import java.awt.geom.AffineTransform;
import java.awt.image.AffineTransformOp;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;

public class test {

    //调整图片大小
    public final static void changeSize(String srcImageFile, String descImageFile, int width, int height, boolean isPadding) {
        try {
            // 缩放比例
            double ratio = 0.0;
            File file = new File(srcImageFile);
            BufferedImage bufferedImage = ImageIO.read(file);
            Image image = bufferedImage.getScaledInstance(width, height, bufferedImage.SCALE_SMOOTH);
            // 计算缩放比例
            if (bufferedImage.getHeight() > bufferedImage.getWidth()) {
                ratio = (new Integer(height)).doubleValue() / bufferedImage.getHeight();
            } else {
                ratio = (new Integer(width)).doubleValue() / bufferedImage.getWidth();
            }
            AffineTransformOp op = new AffineTransformOp(AffineTransform.getScaleInstance(ratio, ratio), null);
            image = op.filter(bufferedImage, null);

            // 是否需要补白
            if (isPadding) {
                BufferedImage tempBufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
                Graphics2D graphics2d = tempBufferedImage.createGraphics();
                graphics2d.setColor(Color.white);
                graphics2d.fillRect(0, 0, width, height);
                if (width == image.getWidth(null)) {
                    graphics2d.drawImage(image, 0, (height - image.getHeight(null)) / 2, image.getWidth(null), image.getHeight(null), Color.white, null);
                } else {
                    graphics2d.drawImage(image, (width - image.getWidth(null)) / 2, 0, image.getWidth(null), image.getHeight(null), Color.white, null);
                }
                graphics2d.dispose();
                image = tempBufferedImage;
            }
            ImageIO.write((BufferedImage) image, "png", new File(descImageFile));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //图片缩放
    public static void mergeImage(String backImage, String srcImage, String descImage) {
        try {
            int offset = 10;
            BufferedImage backBufferedImage = ImageIO.read(new File(backImage));
            BufferedImage srcBufferedImage = ImageIO.read(new File(srcImage));
            // 输出图片宽度
            int width = backBufferedImage.getWidth()/2; //+ offset;
            // 输出图片高度
            int height = backBufferedImage.getWidth()/2; //+ offset;
            BufferedImage descBufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_ARGB);
            Graphics2D graphics2d = (Graphics2D) descBufferedImage.getGraphics();
            graphics2d.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
            // 往画布上添加图片,并设置边距
            graphics2d.drawImage(backBufferedImage, null, 0, 0);
            graphics2d.drawImage(srcBufferedImage, 0,0, width, height, null, null);
            graphics2d.dispose();
            // 输出新图片
            ImageIO.write(descBufferedImage, "png", new File(descImage));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        mergeImage("xx.png","xx.png","D:\\test.png");
    }
}
```

****

##### <font size="4" color="red">54. Srs(Simple Rtmp Server)中配置Flv视频录制</font>

(1) 下载`srs`源代码并解压

```
源码下载地址：https://github.com/ossrs/srs/releases/
```

```
二进制文件下载地址：http://www.ossrs.net/srs.release/releases/download.html
```

(2) 进入`srs`解压目录`/xx/trunk`执行

```shell
./configure --prefix=/xx/srs安装路径 --full
```

(3) 编译

```shell
make 
```

(4) 安装

```shell
make install
```

(5) 修改`srs`安装目录`/xx/conf/srs.conf`文件

`srs.conf`文件中内容：

```nginx
listen              1935;
max_connections     1000;
srs_log_tank        file;
srs_log_file        ./objs/srs.log;
daemon              on;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}
stats {
    network         0;
    disk            sda sdb xvda xvdb;
}
vhost __defaultVhost__ {
    dvr {
        enabled             on;
        dvr_path            ./objs/nginx/html/[app]/[stream].flv;
        # 录制视频的路径以及文件名称生成的格式
        dvr_plan            segment;
        dvr_duration        30;
        dvr_wait_keyframe   on;
    }
    http_hooks {
        enabled         on;
        #配置一个rest服务，进行保存文件信息的收集，post方式
        on_dvr          http://192.168.0.4:8310/index; 
    }
}
```

或

```nginx
listen              1935;
max_connections     1000;
srs_log_tank        file;
srs_log_file        ./objs/srs.log;
daemon              on;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}
stats {
    network         0;
    disk            sda sdb xvda xvdb;
}
vhost __defaultVhost__ {
    dvr {
        enabled             on;
        #all表示录制所有视频流，也可以按频道录制
        dvr_apply       all;
        dvr_path        ./objs/nginx/html/[app]/[stream].[timestamp].flv;
        # 录制视频的路径以及文件名称生成的格式
        dvr_plan            segment;
        dvr_duration        30;
        dvr_wait_keyframe   on;
        #时间戳抖动算法。full使用完全的时间戳矫正；
        #zero只是保证从0开始；off不矫正时间戳。
        time_jitter             full;
    }
    http_hooks {
        enabled         on;
        on_dvr          http://192.168.0.4:8310/index; 
    }
}
```

(6) 进入`srs`安装目录`/etc/init.d`文件夹中并启动`srs`服务

```shell
./srs start
```

(7) 创建`Springboot`项目，并配置依赖包

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

(8) 创建`controller`包，并创建`controller.java`文件

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @RequestMapping(value="/index",method = RequestMethod.POST)
    public String Index(@RequestBody String dvrMsg) {
        try {
            System.out.println(dvrMsg);
            return "成功";
        }
        catch(Exception e) {
            return "失败";
        }
    }
}
```

(9) 主启动目录中内容

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**`on_dvr`回调函数返回参数
> 
> ```json
> {"action":"on_dvr","client_id":509,"ip":"192.168.0.4","vhost":"__defaultVhost__","app":"live","stream":"stream","param":"","cwd":"/disk/D/Srs","file":"./objs/nginx/html/live/stream.flv"}
> ```

(10) 查看端口监听状态

```shell
netstat -antp | grep 1935
```

(11) 使用`ffmpeg`推流

```shell
ffmpeg -re -i xx.mp4 -vcodec libx264 -acodec copy -strict -2 -f flv rtmp://xxIP地址:1935/live/stream
```

> **注：**在`srs`安装根目录`/objs/nginx/html/live/xx.flv`生成

(12) 播放`flv`视频文件

```
地址：http://192.168.0.28:8080/live/stream.flv
```

***

##### <font size="4" color="red">55. Srs(Simple Rtmp Server)配置推流权限(Java语言)</font>

(1) 下载`srs`源代码并解压

```
源码下载地址：https://github.com/ossrs/srs/releases/
```

```
二进制文件下载地址：http://www.ossrs.net/srs.release/releases/download.html
```

(2) 进入`srs`解压目录`/xx/trunk`执行

```shell
./configure --prefix=/xx/srs安装路径 --full
```

(3) 编译

```shell
make 
```

(4) 安装

```shell
make install
```

(5) 修改`srs`安装目录`/xx/conf/srs.conf`文件

`srs.conf`文件中内容：

```nginx
listen            1935;
max_connections   1000;
srs_log_tank      file;
srs_log_file      ./objs/srs.log;
http_api {
    enabled    on;
    listen    1985;
}
http_server {
    enabled    on;
    listen    8080;
    dir    ./objs/nginx/html;
}
stats {
    network    0;
    disk  sda sdb xvda xvdb;
}

vhost __defaultVhost__ {
    min_latency    on;
    mr {
        enabled    off;
    }
    mw_latency    100;
    gop_cache    off;
    queue_length    10;
    tcp_nodelay    on;
    http_hooks {
        enabled  on;
        on_connect  http://192.168.0.4:8310/connect;
        on_close    http://192.168.0.4:8310/close;
        on_publish   http://192.168.0.4:8310/publish;
        on_unpublish  http://192.168.0.4:8310/unpublish;
        on_play  http://192.168.0.4:8310/play;
        on_stop  http://192.168.0.4:8310/stop;
        on_dvr  http://192.168.0.4:8310/dvr;
        on_hls_notify http://192.168.0.4:8310/notify;
    }
}
```

> **注：**`Srs`要求`Http`服务器返回状态码`200`并且`response`内容为整数错误码(0表示成功)，其他错误码会断开客户端连接。

(6) 进入`srs`安装目录`/etc/init.d`文件夹中并启动`srs`服务

```shell
./srs start
```

(7) 创建`Springboot`项目，并配置依赖包

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

(8) 创建`controller`包，并创建`controller.java`文件

`controller.java`文件中内容：

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @RequestMapping(value="/connect",method = RequestMethod.POST)
    public int Connect(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        //0表示成功1表示失败
        return 0;
    }

    @RequestMapping(value="/close",method = RequestMethod.POST)
    public ResponseEntity<Object> Close(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);

    }

    @RequestMapping(value="/publish",method = RequestMethod.POST)
    public ResponseEntity<Object> Publish(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);

    }

    @RequestMapping(value="/unpublish",method = RequestMethod.POST)
    public ResponseEntity<Object> Unpublish(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);

    }

    @RequestMapping(value="/play",method = RequestMethod.POST)
    public ResponseEntity<Object> Play(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);

    }

    @RequestMapping(value="/stop",method = RequestMethod.POST)
    public ResponseEntity<Object> Stop(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);
    }

    @RequestMapping(value="/dvr",method = RequestMethod.POST)
    public ResponseEntity<Object> Dvr(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);
    }

    @RequestMapping(value="/notify",method = RequestMethod.POST)
    public ResponseEntity<Object> Notify(@RequestBody String dvrMsg) {
        System.out.println(dvrMsg);
        return new ResponseEntity<Object>(0,HttpStatus.OK);
    }
}
```

(9) 主启动目录中内容

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**`on_dvr`回调函数返回参数
> 
> ```json
> {"action":"on_dvr","client_id":509,"ip":"192.168.0.4","vhost":"__defaultVhost__","app":"live","stream":"stream","param":"","cwd":"/disk/D/Srs","file":"./objs/nginx/html/live/stream.flv"}
> ```

(10) 查看端口监听状态

```shell
netstat -antp | grep 1935
```

(11) 使用`ffmpeg`推流

```shell
ffmpeg -re -i xx.mp4 -vcodec libx264 -acodec copy -strict -2 -f flv rtmp://xxIP地址:1935/live/stream
```

***

##### <font size="4" color="red">56. Srs(Simple Rtmp Server)中回调函数</font>

1.回调函数

| 回调函数           | 描述                       |
| -------------- | ------------------------ |
| `on_connect`   | 当客户端连接到指定的`vhost`和`app`时 |
| `on_close`     | 当客户端关闭连接，或者`Srs主动关闭连接时`  |
| `on_publish`   | 当客户端发布流时                 |
| `on_unpublish` | 当客户端停止发布流时               |
| `on_play`      | 当客户端开始播放流时               |
| `on_stp`       | 当客户端停止播放时                |
| `on_dvr`       | 当`dvr`录制关闭一个`flv`文件时     |

2.回调函数返回参数

(1) `on_connect`函数

```json
{"action": "on_connect","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app/": "live","tcUrl": "rtmp://video.test.com/live?key=d2fa801d08e3f90ed1e1670e6e52651a","pageUrl": "http://www.test.com/live.html"}
```

(2) `on_close`函数

```json
{"action": "on_close","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","send_bytes": 10240, "recv_bytes": 10240}
```

(3) `on_publish`函数

```json
{"action": "on_publish","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","tcUrl" => "rtmp://video.test.com/live?token=xxx&salt=yyy","stream": "livestream", "param":"?token=xxx&salt=yyy"}
```

(4) `on_unpublish`函数

```json
{"action": "on_unpublish","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy"}
```

(5) `on_play`函数

```json
{"action": "on_play","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy","pageUrl": "http://www.test.com/live.html"}
```

(6) `on_stop`函数

```json
{"action": "on_stop","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy"}
```

(7) `on_dvr`函数

```json
{"action": "on_dvr","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy","cwd": "/usr/local/srs","file": "./objs/nginx/html/live/livestream.1420254068776.flv"}
```

***
