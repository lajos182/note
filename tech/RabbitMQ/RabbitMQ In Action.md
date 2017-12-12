# RabbitMQ In Action

生产者发布消息,而消费者订阅他们`感兴趣`的消息,如何区分不同的消息,特定的消息如何发送到特定的消费者.

要维护所有的消费者信息.消息到来时,筛选出符合规则的消费者,把消息发送给他们.

AMQP 从任何发布者到任何消费者,只通过一条软件总线实时动态链接起来

这根总线对任何设备可插拔

## 消息通信

生产者创建的消息包含两部分:payload和label.

payload是想要传送的数据,可以是任何内容mp4或者json等等.label是描述payload,用来决定谁将获得消息的拷贝.

消费者连接到代理服务器,并且订阅到队列上.

### 信道

应用程序通过TCP与RabbitMQ建立连接.一旦TCP连接打开,应用程序就可以在TCP中建立信道.他是TCP连接中的虚拟连接.

建立和销毁TCP连接耗费大量资源-->所以多个线程共享一个TCP连接.

### EXCHANGE BIND  QUEUE

生产者->消息-->交换器-->queue

消费者必须发送确认,RabbitMQ才会把消息删除掉,如果消费者在内超时时间范围内,一直没有发送确认,那么RabbitMQ也会重新发送该消息

如果消费者一直没有发送确认,那么RabbitMQ不会再向该消费者发送任何消息.

如果处理消息非常耗时,那么可以延迟确认消息,直到消息处理完成,这样可以防止RabbitMQ不停的发送新的消息过来

如何拒绝消息:

1. 断开连接.RabbitMQ会自动重新发送消息给其他消费者
2. basic.reject.默认参数requeue为True,如果requeue为False,那么RabbitMQ会丢弃掉该消息,不在发送给任何消费者,而是记录为dead letter

producer和consumer都应该创建queue,如果启动了生产者,但是还没有声明queue,那么生产者发送的消息将会全部丢失.

服务器会根据路由键将消息从交换器路由到队列,但是他是如何投递到多个队列的情况的呢,协议中定义的不同类型交换器发挥了作用.一共4中类型:direct,fanout,topic和headers.header允许匹配消息的header而非路由键.headers性能很差.

direct 消息只会发送到一个消费者

fanout 会给每一个消费者发送消息

topic exchange会根据路由pattern来讲消息发送给符合pattern的队列,队列需要绑定pattern.`#`匹配任意字符,`*`会将`.`当做分隔符

### 消息持久化

队列和交换器的durable属性设置为True时,重启rabbitmq他们不会消失,*但是消息没有被持久化*

消息持久化需要把投递模式改为2,而且持久化消息必须被发送到持久化交换器,和持久化队列中.

把信号设置成事务模式后,第一条消息发送成功时信道才会继续执行完其他命令,如果发送失败,其他命令将不执行.**事务对性能影响极大**

**发送方确认模式**(代替事务):信道设置成*confirm*模式,所有的消息都会被分配一个唯一的ID.当消息被投递到所有匹配的队列后,信道会发送一个发送方确认模式给生产者应用程序(包含唯一的ID)

star_consumign()是一个无尽的阻塞while循环

## 运行和管理

`rabbitmq-server -detached`

### 关闭和重启应用程序

`rabbitmqctl stop`会把应用程序和节点都关闭掉

`rabbitmqctl stop_app`只会关闭应用程序

