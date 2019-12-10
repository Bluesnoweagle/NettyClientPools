# NettyClientPools
基于apche-common-pool2的Netty客户端连接池，支持同步，异步，超时设置，断线重连，登录认证等功能

之前公司内部都是使用自己的消息中间件的jar包，客户端是每次使用都重新创建，使用完就关闭。
这在大并发的情况下太消耗消息服务器的资源，并且创建连接也需要消耗不少的时间，毕竟三次握手，而且再加上消息服务器的登录认证，基本要消耗几十ms的时间来建立连接。
所以利用这次新项目的时期，基于netty4.26与apach-common-pool2实现了一套新的客户端程序。

## 主要功能点：
* 客户端连接池，支持健康检查，动态创建和销毁
* TCP登录认证
* 心跳机制
* 同步、异步发送接收
* 超时机制（发送消息后等待N秒后超时）

## 主要实现原理
### Netty客户端实现
先实现一个客户端，再进行池化，由于使用连接池进行
#### Bootstrap初始处理链
每一个Channel的编解码器，登录认证，处理handler，超时，心跳，异常处理，断线重连等都是一种处理handler，都可以放在handler的传播链中。这些传播链中的元素都需要在ChannelInitializer中设置，每启动一个channel时都会调用其中的initChannel方法，因此可以将需要与Channel绑定的属性或者方法在此设置。

### 客户端连接池
Netty本身自带连接池:ChannelPool（连接池）与ChannelPoolMap（多个不同地址的连接池），但没有没有健康检查机制、无法剔除(evict)指定Channel、无法动态控制连接数等。
所以不建议使用。我选用的是apche-common-pool2作为连接池，相关资料可以自行百度。
#### 创建Netty的启动器Bootstrap
由于Chanel由连接池来管理而且是客户端，所以我们的Channel的loop的线程池可以只设置为1.
#### 创建一个BasePooledObjectFactory对象——public class MsgsrvChannelPoolFactory extends BasePooledObjectFactory&lt;Channel&gt;
其中包含create（）:创建一个Channel对象，当池中对象不足时会调用该方法来创建一个对象，这里主要是调用bootstrap对象来发起一个连接，
注意bootstrap不用每次创建，而提供创建对象，而create方法只需要调用connect返回一个ChannelFuture。
这里就是netty的Channel,相关代码
 ``` python
 a = range(2000)
 for i in a:
    if i % 2 == 0:
        continue
    print(i + 1)
 ```
