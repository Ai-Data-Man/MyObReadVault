---
{"title":"消息中间件MQ总结","url":"https://www.wiz.cn/xapp","clipped_at":"2022-08-02 06:56:32","author":"我自己","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/消息中间件MQ总结/","dgPassFrontmatter":true}
---

# 消息中间件MQ总结

## MQ的本质


## 透过模型看MQ的应用场景(先有应用场景，后有解决方案，而模型恰恰就是解决方案的抽象，因此从这个角度看MQ)

* 从RPC通信到MQ通信
  
  ![图片](https://mmbiz.qpic.cn/mmbiz_png/AaabKZjib2kYtDDgk83QJEQc4mzdZXMoTBgOCf9taFQ44e2dhuIIX0xiaZY7CnxarrMFtDbAMmEw6eO3LvZNZlDQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
* MQ的中间转储本质决定了它的应用场景（解耦、异步、限流这三个最根本，其他的场景为衍生）：
  
  * **解耦**（系统或者微服务）
    
    * 上游（生者者）和下游（消费者）物理和逻辑上都无需直接耦合，各自只要和MQ单向直接耦合即可
  * **异步**
    
    * 因为MQ中间转储，生产者放完消息就可以继续走了，不用等消费者回应，节省时间
  * **削峰/限流**
    
    * MQ中间转储，可以buff住超峰消息，起到限流保护作用
      
      * ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVOgaAgt8teMfdwibooQOyQyxTUkpZg9woVWH3r8fgdeTvIibdWAEAu6Xw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
  *  延迟通知
  * 最终一致性保证
  * 顺序消息
  * 流失处理
* 举例：
  
  * 举例1上游不必关心下游执行结果：以用户下单为例，请求会先通过上游的订单系统，然后分别调用下游子系统：支付系统、库存系统、积分系统 和 物流系统。如果调用的任何一个子系统出现异常比如宕机，整个请求都会异常，上下游逻辑+物理逻辑依赖严重。还有，每增加一个下游子系统，都需要更改上游订单系统代码。最后，因为同步调用，流程执行的时间增加了
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVVHfo3x761bMOclj84IIhibTaWHNdibBp7MLgqOiaBn3prCI7a8UrrVRPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
    
    使用mq之后，订单系统作为消息生产者，保证它自己没有异常即可，不会受支付等业务子系统的异常影响，且各个消费者业务子系统之间也互不影响，也就是上下游逻辑+物理解耦，除了与MQ有物理连接，模块之间都不相互依赖。新增一个下游消息关注方，上游不需要修改任何代码，符合架构设计的依赖倒置原则。消息异步，执行时间更短
    
    ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVsXA4ecrCXyOiaLjGBDW4QTjQUmiaE824xAJLKYwu0ia2NxvS5VB4hBOUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
  * 举例2 数据驱动的任务依赖：有些任务之间有一定的依赖关系，比如：task3需要使用task2的输出作为输入，task2需要使用task1的输出作为输入。这样的话，tast1, task2, task3之间就有任务依赖关系，必须task1先执行，再task2执行，再task3执行。对于这类需求，常见的实现方式是，使用cron人工排执行时间表：
    ![](/img/user/阅读库藏/assets/ecf380a8-00be-42db-884e-64711c75bfd8.png)
    
    结果有以下坏处：
    
    1. 如果有一个任务执行时间超过了预留buffer的时间，将会得到错误的结果；
    2. 总任务的执行时间很长，总是要预留很多buffer，如果前置任务提前完成，后置任务不会提前开始；
    3. 如果一个任务被多个任务依赖，这个任务将会称为关键路径，排班表很难体现依赖关系，容易出错；
    4. 如果有一个任务的执行时间要调整，将会有多个任务的执行时间要调整。
    
    使用MQ之后：
    
    ![](/img/user/阅读库藏/assets/8de6a95a-113f-41ad-a84d-39b3f089f821.png)
    
    1. 不需要预留buffer，上游任务执行完，下游任务总会在第一时间被执行;
    2. 依赖多个任务，被多个任务依赖都很好处理，只需要订阅相关消息即可;
    3. 有任务执行时间变化，下游任务都不需要调整执行时间
  * 举例3上游关注执行结果，但执行时间很长：比如微信支付，跨公网调用微信的接口，执行时间会比较长，但调用方又非常关注执行结果，此时一般回调网关+MQ方案来解耦：
    
    ![](/img/user/阅读库藏/assets/3d407786-8637-4a49-8416-539e31d341f4.png)
* 举例4请求高峰：比如系统上秒杀功能，用户突增，一时间所有的请求都到数据库，可能会导致数据库无法承受这么大的请求峰值，响应变慢或者直接挂掉
  
  ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbV8uWc4QPsFQljyWiaibHicIsM2icZQxtoG1uYTWa9x6yDPHnr3z9PwWYuAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
  
  使用mq后， 如出现`请求峰值`的情况，因消费者会按照自己的能力来消费消息，多的请求不处理，保留在mq的队列中，不会对系统的稳定性造成影响

# 如何设计MQ
* 从基础功能考虑：较为简单
	* 三个功能：发消息、存消息、消费消息（支持发布订阅）
{ #rko9w5}

* 从非功能性需求考虑：复杂，很关键
{ #oi2ex3}

  
  * 高并发场景下收发消息的性能
    
    * 存储高性能
    * 网路及IO高性能
  * 高可用高可靠
    
    * 消息服务的高可用
    * 存储高可用
    * 消息可靠性
  *  扩展性
    
    * 消息服务水平扩展
    * 消息存储水平扩展
  * 元数据管理
    
    * 消费关系管理
  * 数据一致性问题
*    整体设计思路：
  
  ![图片](https://mmbiz.qpic.cn/mmbiz_png/AaabKZjib2kYtDDgk83QJEQc4mzdZXMoTPZfxteQ4f8zSYfy8DSDMWV93YOHBxJyoN7Vq1Do1wfc2JxlunQZxNw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
{ #csagm5}

  
  * Broker（服务端）：MQ 中最核心的部分，是 MQ 的服务端，核心逻辑几乎全在这里，它为生产者和消费者提供 RPC 接口，负责消息的存储、备份和删除，以及消费关系的维护等。
  * Producer（生产者）：MQ 的客户端之一，调用 Broker 提供的 RPC 接口发送消息。
  * Consumer（消费者）：MQ 的另外一个客户端，调用 Broker 提供的 RPC 接口接收消息，同时完成消费确认
* 设计细化
  
  * 难点1RPC通信：
{ #f9avq9}

    
    * 成熟框架： Dubbo或Thrift
    * 底层自研：
      
      * Netty做底层通信
      * Zookeeper、Euraka 等来做注册中心
      * 自定义通信协议如Kafka或套标准MQ协议如AMQP
  * 难点2高可用可靠
    
    * Broker服务高可用可靠
      
      * 通过服务自动注册与发现、负载均衡、超时重试机制、发送和消费消息时的 ack 机制确保Broker集群部署
    * 存储高可用可靠
{ #6sjon3}

      
      * 参考Kafaka分区+多副本模式，但要注意和Kafaka一样要考虑到数据复制和一致性方案，并实现自动故障转移
      * 使用主流的有高可用方案的DB/分布式文件系统/KV系统
      * 要求不高的话可考虑直接内存或分布式缓存
    * 消息可靠性
{ #5izcl5}

      
      * 消息必达
        
        为了保证消息必达：
        
        * 要在架构中增加*ACK* (*Ack*nowledge character）确认字符机制，以表示发来的数据已确认接收无误，然后基于ACK，来做消息重传。
        * 消息落地要保证(比如消息持久化)
        
        ![](/img/user/阅读库藏/assets/31b436e3-c618-40c7-be6a-68a91bda77fa.png)
        
        ![](/img/user/阅读库藏/assets/a7f458db-c8de-4133-9019-34f9231ae786.png)
  * 难点3存储设计
    
    * 存储高性能
      
      * 目前主流的方案是：追加写日志文件（数据部分） + 索引文件的方式（很多主流的开源 MQ 都是这种方式），索引设计上可以考虑稠密索引或者稀疏索引，查找消息可以利用跳转表、二分查找等，还可以通过操作系统的页缓存、零拷贝等技术来提升磁盘文件的读写性能。
      * 不追求很高的性能，也可以考虑现成的分布式文件系统、KV 存储或者数据库方案。
  * 难点4消费关系管理
    
    * 由于 Broker 是集群部署的，所以消费关系通常维护在公共存储上如 Zookeeper、Apollo 等配置中心来管理以及进行变更通知
  * 难点4高性能设计
    
    * 其他性能优化
      
      * 比如 Reactor 网络 IO 模型、业务线程池的设计、生产端的批量发送、Broker 端的异步刷盘、消费端的批量拉取等等

## 引入MQ的好坏处

* 好处即应用场景，坏处如下：
  
  * 多了一个MQ组件，系统更复杂，意味着要考虑的问题更多，如下：
    
    * 需要额外部署mq服务器
    
    - mq有一定学习成本，使用有点复杂
    - 使用不好出现的问题，不易排查，解决方案也较复杂
      
      * 重复消息问题(重复消息不做正确的处理，会对业务造成很大的影响)
        
        * 场景：
          
          * 生产者产生重复消息
          * kafka/rocketmq的offset回调
          * 消费者确认失败或超时（消息可靠性设计，此时MQ重传）
          * 业务系统主动重试（消息可靠性设计）
        * 举例：如会员系统多开通了一个月的会员
          
          ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbV4yHj1U4qdXZ4z8b3FAaNE9nicgoqweibV5feiaV5ASIoqzwOXEVIfibKMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
          
          * 解决方案：
            
            * 前提：
              
              * 在消息生产时，MQ 内部针对每条生产者发送的消息生成一个 inner-msg-id(MQ生成，对消息的唯一标识)，作为去重的依据（消息投递失败并重传），避免重复的消息进入队列；  
              * 在消息消费时，要求消息体中也必须要有一个 msgid（对于同一业务全局唯一，如支付 ID、订单 ID、帖子 ID 等）作为去重的依据，避免同一条消息被重复消费。
            * MQ使用冥等设计
            * 消费者业务处理使用冥等设计
            
            ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVNdSXkDuxQmvhw0mibq32ZKK9w5GIRs6UzjZG6XEWNjakrV6MibpbZ5aQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1=238x360.4125061035156)
      * 数据一致性问题 
        
        * 场景：
          
          * 消费者业务处理异常
        * 举例 ：一个完整的业务流程是，下单成功之后，送100个积分。下单写库了，但消息消费者在送积分的时候失败，就会造成`数据不一致`，即该业务流程的部分数据写库了，另外一部分没有写库。如果下单和送积分在同一个事务中，要么同时成功，要么同时失败，是不会出现数据一致性问题的。但由于跨系统调用，为性能考虑，一般不会用强一致性的方案，而改成达成最终一致性即可
          
          ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVoXsqEv0SlKZQ39CyfZPm3nR7jDXPpM5XicS3cic5Gp1QnebYMhxxIzlg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
          
          * 解决方法：
            
            * 方法一同步重试
              
              * 消费消息时处理失败，立刻重试3到5次，如还失败则写入重试表
              * 不适合消息量较大场景，因为影响处理速度，可能造成消息堆积
            * 方法二异步重试
              
              * 消费处理失败时写入重试表。设置job定时处理重试任务
              * 适合消息量较大业务场景，但一致要延后
            * 消费端重发消息到topic，变相重试
              
              * 适合消息顺序要求不高的场景
      * 消息丢失问题（最终的结果会导致消费者无法正确的处理消息，而导致数据不一致的情况）
        
        * 场景：
          
          * 生产者发消息时的网路原因
          * mq服务器持久化时，磁盘异常
          * kafka/rocketmq的offset被回调时，略过了消息
          * 消费者已经ack确认了，刚读取消息还没处理完服务就重启了
        * 解决方法：
          
          增加一张`消息发送表`，当生产者发完消息之后，会往该表中写入一条数据，状态status标记为待确认。如果消费者读取消息之后，调用生产者的api更新该消息的status为已确认。有个job，每隔一段时间检查一次消息发送表，如果5分钟（时间可以视实际情况来定）后还有状态是待确认的消息，则认为该消息已经丢失了，重新发条消息
          
          ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVTbsSeOVfbXvUEicpunXwB7ZbbXQcDeH00QFU8Mr3YoBktE9cJiclria2g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
      * 消息顺序问题
        
        * 场景：
          
          * kafka同一个`partition`中能保证顺序，不同partition无法保证顺序
          * `rabbitmq`的同一个`queue`能够保证顺序，但是如果多个消费者同一个`queue`也会有顺序问题
          * 消费者多线程消费，无法保证顺序
          * 消费者处理多条消息中的一条出错，顺序会打乱
          * 生产者发送到mq中的路由规则，与消费者不同
        * 举例：订单有下单、支付、完成、退货等状态，果订单数据作为消息体，就涉及顺序问题。如消费者收到同一个订单的两条消息，但如果第一条消息的状态是支付，第二条消息的状态是下单就会有问题，没有下单就先支付了？![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVQMDicyRxXg6fj3vA3Zp1gXIsD4AJn7H0iaDjWAu6tlFny5qoepOfn9Sw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
        * 解决方法：
          
          * 若只是查看最终状态的需求，其实消费端调整查询逻辑即可，顺序什么的无所谓
          * 但要真的要有顺序，以kafka和上述举例来说，可以把订单号路由到不同的`partition`，同一个订单号的消息，每次到发到同一个`partition`。
            
            ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVAH7cH61JTicqsbgDGLfYetTib5V4e4soyluibJXvqNxUhAc39cvGSVnww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
      * 消息堆积（消息消费的速度小于生产的速度。这样会直接导致消息堆积问题，从而影响业务功能）
        
        * 场景：
          
          * 采用很慢的批处理
        * 举例：以下单开通会员为例，如果消息出现堆积，会导致用户下单之后，很久之后才能变成会员，这种情况肯定会引起大量用户投诉
          
          ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVjgfYHmM1uFfrJQF9X8bcuts0FwRkcqvs6u9WaicofzSWiayYJpPKMIag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
          
          * 解决方法（提升消费者的处理能力）：
            
            * 如果不用考虑消息顺序，可以消费者多线程并发消费消息，进行业务逻辑处理
            * 如果要保证顺序，可以读取消息之后，将消息按照一定的规则分发到多个队列中，然后在队列中用单线程处理
              
              ![图片](https://mmbiz.qpic.cn/mmbiz_png/uL371281oDEEegWyONc7CZpoOuRp6GbVMicvG4qMqVCejmf37bHOiau7fNUeoTwzB25gNibmXPvlYC1NFnOJOibQQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
  * 消息传递路径更长，延时会增加
  * 消息可靠性和重复性互为矛盾，消息不丢不重难以同时保证
  * 上游无法实时知道下游的执行结果
* 基于引入MQ的好坏处，可以得出：
  
  * 什么时候不使用MQ？无应用场景或上游实时关注执行结果
  * 什么时候使用MQ？有应用场景，并且上游不实时关注执行结果

## AMQP

* 是什么
  
  * AMQP(Advanced Message Queuing Protocol)高级消息队列协议
  * 协议包括：
    
    - 系统模型，定义了AMQP中各个**实体**的名称，交互流程
    - 通讯规范，定义了实体之间交互的数据包格式、**实体**之间通讯的具体命令
    - 序列化标准，定义了通讯数据的序列化和反序列化格式
  * 协议不限制产品、开发语言
* AMQP系统模型![](/img/user/阅读库藏/assets/eecd9ff4-f721-4842-9326-3c8284b7055a.png)以下是系统模型中涉及到的实体：
  
  * Message消息
    
    * 消息是不具名的通信数据，它由消息头和消息体组成
    * 消息体不透明，消息头则由一些可选属性组成，包括Routing Key路由键、priority消息优先权、delivery-mode投递方式（指出该消息可能需要持久性存储）等。
  * publisher发布者（生产者producer）
    
    * 消息的生产者，也是一个向交换器发布消息的客户端应用程序
    * 其将消息发布到MQ，大概流程：
      
      * 生产者与Broker建立Connection连接
      * 生产者和Broker建立Channel信道
      * 生产者通过通道消息发送给Broker
  * Consumer消费者
    
    * 消息的消费者，表示一个从消息队列中取得消息的客户端应用程序
    * 接收消息，大概流程：
      
      * 消费者和Broker建立Connection连接
      * 消费者和Broker建立Channel信道
      * 消费者监听指定的Queue队列
      * 当有消息到达Queue时Broker消息队列服务器默认将消息推送给消费者。
      * 消费者接收到消息。
  * Connection连接
    
    * 生产者跟消费者必须跟Broker建立一个连接，这个是TCP长连接
  * Channel信道
    
    * Channel信道是在真实的Connection连接内建立的虚拟连接。AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接
    * 相互隔离，每个Channel有自己的编号id
    * 每一个 channel 只
      能运行在一个独立的操作系统线程上，故投递到特定 channel 上的 message 是有顺序的，多线程之间共享一个 socket。
    * channel 号为 0 的 channel 用于处理所有
      对于当前 connection 全局有效的帧，而 1-65535 号 channel 用于处理和特定 channel 相关的帧
    * 原生API重要的编程接口，调用的都是Channel接口上的方法
  * Broker消息队列服务器
    
    * MQ服务器
  * Virtual Host(VHost)虚拟主机
    
    * 本质上就是一个 mini 版的MQ服务器，每一个VHost各自拥有自己独立的Queue队列、Exchange交换器、Binding绑定关系等相关对象和权限机制
    * VHost是AMQP概念的基础，必须在连接时指定。AMOP的MQ应该初始的拥有默认VHost，以提供MQ服务，比如RabbitMQ默认的VHost是/。
    * 同一个硬件上需要多个MQ服务，只需创建Vhost，以提高资源利用率。不过这样会增加运维成本，必须分配资源部署MQ，保证MQ时刻正常运行
  * Queue
    
    * VHost上用来存储消息直到发送给消费者的对象，它是消息的容器
    * 队列是生产者跟消费者之间的纽带，生产者发送消息到达队列，消费者从队列消费消息
    * 消息一直在队列里面，等待消费者连接到这个队列将其取走
  * Routing Key路由键（路由关键字）
    
    * 由生产者指定的消息的逻辑分类关键字，传给Exchange，由Exchage交换机按分类路由投递到Queue
    * Routing Key路由键是消息的一部分
  * Binding绑定关系
    
    * 关系描述了绑定到Exchange交换机上的Queue队列与其“消息接收意向”（Binding key）的映射
    * 关系维护在Exchange交换机上
  * Exchang交换机
    
    * 根据Binding绑定关系和Routing Key路由键以及Exchang模式所限定的规则，将接受到的生产者发送消息路由给VHost中的Queue队列

    
