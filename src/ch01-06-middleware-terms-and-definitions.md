# Middleware Terms and Definitions

下面列出了一些与中间件有关的术语及其与这些模式有关的定义.

## Event handling

事件处理是在指定的接收者("Handler")处接收可识别的事件. 事件处理涉及的关键流程包括:

- 确定一个事件应该转发到哪里.
- 执行该转发操作.
- 接收转发的事件.
- 采取某种适当的响应措施, 例如写入日志,发送错误或恢复程序, 或者发送额外的消息.

请记住,事件处理最终可能会将事件转发给一个事件消费者.

该功能与中间件的常见用途可以扩展到包括流行的 "发布/订阅" 或 "pub/sub" 功能.在发布/订阅场景中, 中间件将请求或消息从活跃的数据事件发布者那里路由到活跃的数据事件订阅者.然后,这些有活动监听器的消费者可以在事件发布时检索它们.

在使用中间件的 Salesforce 集成中, 事件处理的控制由中间件层承担; 它收集所有相关事件(同步或异步)并管理分发到所有端点, 包括 Salesforce.

另外, 这个功能也可以通过使用带有平台事件的 event bus 在 Salesforce 企业消息平台上实现.

请查看: [http://searchsoa.techtarget.com/definition/event-handler](http://searchsoa.techtarget.com/definition/event-handler).

## Protocol conversion	

协议转换 "_通常是一种软件应用程序,将一个设备的标准或专有协议转换为适合另一个设备的协议,以实现互操作性_.

_在中间件的背景下,与特定目标系统的连接可能受到协议的限制.在这种情况下,消息格式需要转换为目标系统的格式,或者封装在目标系统的格式中,在那里可以提取 payload.这也被称为隧道(tunneling)_".<sup>1<sup> 

Salesforce 不支持本机协议转换, 所以假定任何此类要求都由中间件层或终端来满足.

请查看: [https://www.techopedia.com/definition/30653/protocol-conversion](https://www.techopedia.com/definition/30653/protocol-conversion)

## Translation and transformation

转化是将一种数据格式映射到另一种数据格式的能力, 以确保被整合的各种系统之间的相互操作性. 通常情况下, 这需要在转换过程中对信息进行重新格式化, 以符合发送方或接收方的要求. 在更复杂的情况下, 一个应用程序可以用它自己的本地格式发送消息, 而其他两个或更多的应用程序可能分别以它们自己的本地格式收到该消息的副本.

中间件转换和转换工具通常包括为遗留或其他非标准端点创建服务的能力; 这允许这些端点看起来是服务可分配的.

对于 Salesforce 的集成, 我们认为中间件层或终端都能满足任何此类要求. 数据的转换可以在 Apex 中进行编码, 但出于维护和性能方面的考虑, 我们不建议这样做.

请查看: [http://en.wikipedia.org/wiki/Message-oriented_middleware](http://en.wikipedia.org/wiki/Message-oriented_middleware)

## Queuing and buffering	

排队和缓冲通常依赖于异步的消息传递, 而不是请求-响应架构. 在异步系统中, 当目标程序繁忙或连接受到影响时, 消息队列提供临时存储. 此外, 大多数异步中间件系统提供持久性存储来备份消息队列.

异步消息进程的主要好处是, 如果接收方的应用程序由于任何原因发生故障, 发送方可以继续不受影响; 发送的消息只是积累在消息队列中, 以便在接收方重新启动时进行处理.

Salesforce 仅以基于工作流的出站消息传递形式提供显式排队功能. 要为其他集成场景(包括编排,流程编排,服务质量等)提供真正的消息队列, 需要一个中间件解决方案.

请查看: [http://en.wikipedia.org/wiki/Message-oriented_middleware](http://en.wikipedia.org/wiki/Message-oriented_middleware)

## Synchronous transport protocols	

同步传输协议指的是支持以下活动的协议: "_调用者中的单个线程发送请求消息, 阻塞以等待回复消息, 然后处理回复...., 等待回复的请求线程意味着只有一个未完成的请求, 或者这个请求的回复通道对这个线程是私有的_ ". <sup>2<sup> 

## Asynchronous transport protocols	

异步传输协议是指支持活动的协议, 其中 "_调用者中的一个线程发送请求消息并为回复设置回调. 一个单独的线程侦听回复消息. 当回复消息到达时, 回复线程调用适当的回调, 重新建立调用者的上下文并处理回复. 这种方法使多个未完成的请求能够共享一个回复线程._" <sup>3<sup> 

## Mediation routing	

调解路由是指从组件到组件的复杂的消息 "流" 的规范. 例如, 许多基于中间件的解决方案依赖于一个消息队列系统. 虽然有些实现允许路由逻辑由消息传递层本身提供, 但其他实现则依赖于客户端应用程序提供路由信息,或允许两种模式的混合.在这种复杂的情况下,(中间件方面的)调解简化了开发,集成和验证.

"_具体来说, 调解员协调一组对象, 这样它们[不]需要知道如何相互协调...., 然后, 每个消费者可以专注于处理特定种类的消息, 而协调者[调解员]可以确保正确的消息到达正确的消费者手中._"<sup>4<sup> 

## Process choreography and service orchestration	

流程编排和服务编配都是"服务组合"的形式, 其中协调任意数量的端点和功能.

编排和服务协调之间的区别在于:

- 编排可以被定义为"_由一群没有中央权威的互动个体实体产生的行为._"<sup>5<sup> 
- 协调可以被定义为"_由中央指挥者协调执行独立任务的各个实体的行为而产生的行为._"<sup>6<sup> 

此外, "_编排显示了每个服务的完整行为,而编排结合了每个服务的接口行为描述._" <sup>7<sup> 

业务流程编排的部分内容可以在 Salesforce 工作流中或使用 Apex 构建. 由于 Salesforce 的超时限制和治理者限制(特别是在需要真正的事务处理的解决方案中), 我们建议所有复杂的编排都在中间件层实现.

## Transactionality (encryption, signing, reliable delivery, transaction management)

事务性可以定义为支持包含针对每个所需资源的所有必要操作的全局事务的能力. 事务性意味着支持所有四个 ACID(原子性,一致性,隔离性,持久性)属性, 其中原子性保证工作单元(事务)的结果要么全有要么全无.

虽然 Salesforce 本身是事务性的 ,但它无法参与分布式事务或在 Salesforce 外部发起的事务. 因此, 假设对于需要复杂的多系统事务的解决方案, 事务性(以及相关的回滚/补偿机制)在中间件层实现.

请查看: [http://en.wikipedia.org/wiki/Distributed_transaction](http://en.wikipedia.org/wiki/Distributed_transaction)

## Routing

路由可以定义为指定从组件到组件的复杂消息流. 在现代基于服务的解决方案中, 此类消息流可以基于许多标准,包括数据头(header), 内容类型, 规则和优先级.

对于Salesforce的集成, 我们认为中间件层或终端都能满足任何此类要求. 消息路由可以在 Apex 中进行编码, 但出于维护和性能方面的考虑, 我们不建议这样做.

## Extract, transform, and load

提取,转换和加载 (ETL) 是指涉及以下过程的过程:

- 从源系统中提取数据. 这通常涉及来自几个源系统的数据, 以及关系型和非关系型的结构.
- 转换数据以满足运营需求, 其中可能包括数据质量级别. 转换阶段通常对从源系统中提取的数据应用一系列规则或函数, 以导出要加载到最终目标环境中的数据.
- 将数据加载到目标系统中. 目标系统可能与数据库, 操作数据存储, 数据集市(data mart), 数据仓库(data warehouse)或其他操作系统有很大不同.

虽然并非绝对必要,但大多数成熟的 ETL 工具都提供变更数据捕获的功能. 此功能是该工具识别源系统中自上次提取后发生更改的记录的地方, 从而减少了记录处理量.

Salesforce现在还支持 "Change Data Capture", 即发布代表 Salesforce 记录变更的变更事件. 通过 Change Data Capture, 客户或外部系统会收到Salesforce记录的近乎实时的变更. 这使得客户或外部系统能够同步外部数据存储中的相应记录.

请查看: [https://en.wikipedia.org/wiki/Extract,_transform,_load](https://en.wikipedia.org/wiki/Extract,_transform,_load) and [Change Data Capture 开发者指南](https://developer.salesforce.com/docs/atlas.en-us.242.0.change_data_capture.meta/change_data_capture/cdc_intro.htm).

> **小知识**:
> 
> **Data mart** 和 **Data warehouse** 都是用于存储和管理企业数据的解决方案, 它们之间的区别在于规模,范围和目的.
>
> Data mart 是一个专门为单个业务部门或特定用户群体设计的小型数据仓库. 它通常包含与该部门或用户群体相关的数据, 并且可以由该部门或用户群体独立地管理和使用. Data mart 通常是针对特定问题或业务需求构建的, 因此它们具有较小的规模和范围, 并且可以更快速地实现.
>
>相比之下, Data warehouse 是一个集中式的,大型的,跨部门的数据存储解决方案. 它涵盖了整个组织的数据, 并且旨在帮助用户从不同的角度分析数据来支持业务决策. 数据仓库需要进行复杂的数据集成和转换, 以确保所有数据都可以在一起使用, 这通常需要更长的时间和更多的资源.
>
>总之, data mart 更小,更局限, 而 data warehouse 更大,更全面. 选择哪种解决方案取决于企业的特定需求和预算.

## Long polling	

长轮询,也称为 Comet 编程, 模拟从服务器到客户端的信息推送. 与普通轮询类似,客户端连接并向服务器请求信息. 但是, 如果信息不可用,服务器不会发送空响应, 而是保留请求并等待直到信息可用(事件发生). 然后服务器向客户端发送一个完整的响应. 然后客户立即重新请求信息. 客户端持续保持与服务器的连接, 因此它总是在等待接收响应. 如果服务器超时, 客户端会再次连接并重新开始.

Salesforce Streaming API使用 Bayeux 协议和 CometD 进行长时轮询.

- Bayeux是一个传输异步信息的协议, 主要通过HTTP传输.
- CometD是一个可扩展的基于HTTP的事件路由总线, 使用被称为 Comet 的 AJAX 推送技术模式. 它实现了 Bayeux 协议.


<sup>1<sup> Gregor Hohpe, and Bobby Woolf, Enterprise Integration Patterns (Boston: Addison-Wesley Professional, 2003).

<sup>2<sup> Gregor Hohpe, and Bobby Woolf, Enterprise Integration Patterns (Boston: Addison-Wesley Professional, 2003).

<sup>3<sup> Ibid.

<sup>4<sup> Ibid.

<sup>5<sup> "Choreography and Orchestration: A Software Perspective," e-Zest, last accessed April 11, 2019, [http://www.e-zest.net/blog/choreography-and-orchestration-a-software-perspective/](http://www.e-zest.net/blog/choreography-and-orchestration-a-software-perspective/).

<sup>6<sup> Ibid.

<sup>7<sup> "Orchestration vs. Choreography," Stack Overflow, last accessed April 11, 2019, [http://stackoverflow.com/questions/4127241/orchestration-vs-choreography](http://stackoverflow.com/questions/4127241/orchestration-vs-choreography).