# 远程系统调用-即发即弃

## Context

您使用 Salesforce 来跟踪潜在客户,管理您的销售渠道,创建业务机会. 但是,Salesforce 系统不包含或处理订单. 在 Salesforce 中获取到订单详细信息后, 将在远程系统中创建订单, 该系统管理订单直至流程结束.

当您在项目中实施这种模式时,Salesforce 调用远程系统来创建订单, 但并不等待调用的成功完成. 远程系统可以选择在一个单独的事务中用新的订单号和状态更新 Salesforce 数据.

## Problem

当 Salesforce 中发生事件时,如何在远程系统中启动一个流程, 并将所需信息传递给该流程, 而无需等待远程系统的响应?

## Forces

在应用基于这一模式的解决方案时, 要考虑以下几个方面:

- 调用远程系统是否需要 Salesforce 等待响应才能继续处理?调用远程系统是同步还是异步的?
- 如果对远程系统的调用是同步的, 那么响应是否需要作为调用的一部分在 Salesforce 中进行处理?
- 传递的信息量是否很小?
- 集成是基于特定事件的发生的吗? 如 Salesforce 用户界面上的按钮点击,还是基于 DML 的事件?
- 从 Salesforce 到远程系统的消息传递是否有保证?
- 远程系统能否参与以契约(contract)为先的集成,其中 Salesforce 指定集成规范(contract)? 在某些解决方案变体(例如,出站消息), Salesforce 指定远程系统实现指定的规范.
- Endpoint 或企业服务总线(ESB)是否支持长轮询?
- 声明性配置的方法是否优于自定义 Apex 开发? 在这种集成模式下, 像平台事件这样的解决方案优于 Apex 调用.

> 这里的 **Contract** 可以理解为一个规范或协议, 用于描述两个或多个系统之间的交互行为. 在这种情况下, Salesforce 定义了一个规范, 要求远程系统按照规范进行实现.
>
> 关于**长轮询(Long Polling)**, 它是一种 Web 应用的实时通信技术, 允许客户端发送异步请求并保持打开连接, 在有新数据可用时即时接收响应. 而 ESB 是一种支持不同应用和服务集成的中间件, 用于管理不同服务之间的通信.

## Solution

下面几个部分包含了这个集成问题的解决方案:

### 方案一: 流程驱动的平台事件(Process-driven platform events) (适配度: Best)

在 Salesforce 中实现平台事件不需要定制化. 推荐的解决方案是在插入或更新事件中调用远程系统.

平台事件是一个事件消息(或通知), 以便您的应用程序在发送和接收消息后可以采取进一步操作, 平台事件简化了在不编写复杂逻辑的情况下通信变化并响应的过程. 一个或多个订阅者可以侦听同一事件并执行操作.

例如: 软件系统可以发送包含打印机墨盒信息的事件. 订阅者可以订阅事件, 来监控打印机墨水量, 并下订单更换墨水量低的墨盒.

外部应用程序可以通过 CometD 订阅一个 Channel 来监听事件消息. 平台应用程序, 如 Visualforce 页面和 Lightning 组件, 也可以用 CometD 订阅事件消息.

### 方案二: 基于自定义驱动的平台事件(Customization-driven platform events) (适配度: Good)

类似于方案一, 但是此方案是是由Apex Trigger 或 Class 创建的, 你可以通过使用 Apex 或 API 来发布和消费平台事件.

平台事件通过 Apex Trigger 与 Salesforce 平台集成. Apex Trigger 是 Salesforce 平台上监听事件消息的事件消费者.

当外部应用程序使用API或 Salesforce应用程序使用 Apex 发布事件消息时, 就会触发该事件的触发器. 触发器运行响应事件通知的动作.

### 方案二: 基于工作流驱动的出站消息传递(Workflow-driven outbound messaging) (适配度: Good)

在 Salesforce 中实现出站消息不需要定制化. 针对这种类型的集成, 建议的解决方案是在插入或更新事件中调用远程过程. Salesforce 提供了基于工作流的出站消息功能, 允许在 Salesforce 中的插入或更新操作触发时向远程系统发送 SOAP 消息. 这些消息是异步发送的, 与 Salesforce 用户界面无关.

出站消息被发送到特定的远程端点. 远程服务必须能够参与以契约为先的集成, 其中 Salesforce 提供契约规范.

收到消息后, 如果远程服务没有用肯定的确认信息回应, Salesforce 将重试发送消息, 提供一种保证交付的方式.当使用中间件时, 这个解决方案成为 _第一英里(first-mile)_ 交付保证.

> 在这里, **first-mile** 指的是从消息发送方到中间件的传输阶段, 也就是消息传输的起始阶段. 因此, **first-mile guarantee of delivery** 是指在消息从发送方到达中间件的过程中, 确保消息的可靠交付.

### 方案三: 出站消息和回调(Outbound messaging and callbacks) (适配度: Good)

回调提供了一种方法来减轻消息顺序错乱的影响.此外,它们还处理这些场景:

- **幂等性:** 如果没有及时收到确认, 则出站消息执行重试. 可以向目标系统发送多条消息. 使用回调确保检索到的数据是在特定时间点, 而不是在发送消息时.
- **获取更多数据:** 单个出站消息只能发送单个对象的数据. 可以使用回调从其他相关记录检索数据, 例如与父对象相关联的相关列表.

出站消息提供了一个唯一的 SessionId, 您可以使用它作为认证令牌来使用 SOAP API 或 REST API 进行回调的认证和授权. 执行回调的系统不需要单独向 Salesforce 进行身份验证. 然后可以使用任一 API 的标准方法来执行所需的业务功能.

这个变体的典型用途是 Salesforce 向远程系统发送出站消息以创建记录的场景. 回调会使用在远程系统中创建的记录的唯一键更新原始的 Salesforce 记录.

### 方案四: 自定义 Lightning 组件或 Visualforce 页面,使用 Apex SOAP 或 HTTP 异步调用(Custom Lightning component or Visualforce page that initiates an Apex SOAP or HTTP asynchronous call-out)  (适配度: Suboptimal)

这种解决方案通常用于基于用户界面的场景,但需要定制.此外,解决方案必须在代码中处理消息的可靠传递.

类似于 _远程流程调用的--请求和回复_ 模式的解决方案, 指定使用 Visualforce 页面或 Lightning 组件, 以及 Apex call-out. 不同的是, 在这种模式中, Salesforce 不会等待请求完成后再将控制权交给用户.

在收到消息后, 远程系统会做出响应并表示收到消息, 然后异步处理该消息. 远程系统在开始处理消息之前将控制权交还给 Salesforce; 因此, Salesforce 无需等待处理完成.

### 方案五: 从 Salesforce 数据变化后触发触发器再执行一个 Apex SOAP 或 HTTP 异步调用(Trigger that's invoked from Salesforce data changes performs an Apex SOAP or HTTP asynchronous cal-lout) (适配度: Suboptimal)

你可以使用 Apex 触发器基于记录数据的变化来执行自动化.

使用Apex触发器可以将Apex代理类作为DML操作的结果执行. 但是, 从触发器上下文中发起的所有调用都必须异步执行.

### 方案六: 使用 Batch Job 执行 Apex SOAP 或 HTTP 异步 call-out(Batch Apex job that performs an Apex SOAP or HTTP asynchronous call-out) (适配度: Suboptimal)

可以在 Batch Job 中调用远程系统. 该解决方案允许批量执行远程进程, 并在 Salesforce 中处理来自远程系统的响应. 然而, 在给定的批处理上下文中, 调用数量存在限制. 有关更多信息, 请查看 [Salesforce Developer Limits and Allocations Quick Reference](https://developer.salesforce.com/docs/atlas.en-us.242.0.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_overview.htm)


## Sketch

以下图表说明了Salesforce向远程系统发起调用的情况, 其中对记录进行创建或更新操作会触发调用.

<p align="center">
    <img src="img/pattern-callout-to-fire-forget-flow.png?raw=true">
</p>

具体步骤: 

1. 一个远程系统订阅了 Salesforce 平台事件.
2. 更新或插入发生在 Salesforce 的一组特定记录上.
3. 当满足一组条件时, Salesforce流程将被触发.
4. 这个过程创建一个平台事件.
5. 远程监听器接收事件消息, 并将消息放置在本地队列中.
6. 排队的应用程序将消息转发给远程应用程序进行处理.

在远程系统必须对 Salesforce 执行操作的情况下, 您可以实现一个可选的回调操作.

## Results

与此模式相关的解决方案的应用可以实现:

- 用户界面发起的远程进程调用, 其事务结果可以显示给最终用户.
- DML事件启动的远程进程调用, 其中事务的结果可以由调用进程处理.

### Calling Mechanisms

调用机制取决于为实现这一模式而选择的解决方案.

| **调用机制**                                   | **说明**                                                                                                   |   |
|---------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|---|
| Process Builder                                         | 这适用于由流程驱动和定制驱动的解决方案使用. 事件触发 Salesforce 流程,然后该流程可以发布平台事件以供远程系统订阅. |   |
| Lightning component or Visualforce and Apex controllers | 这适用于使用 Apex callout 异步调用一个远程进程.                                                                  |   |
| Workflow rules                                          | 仅用于出站消息解决方案.创建和更新DML事件触发Salesforce工作流规则,然后可以将消息发送到远程系统.                 |   |
| Apex triggers                                           | 用于触发器驱动的平台事件和从 DML 触发的事件中使用 Apex callout来调用远程进程.                                    |   |
| Apex batch classes                                      | 用于在批处理模式下调用远程程序.                                                                                   |   |

### Error Handling and Recovery

处理错误和恢复策略必须作为整体解决方案的一部分进行考虑.最佳方法取决于您选择的解决方案.

#### 方案一: Apex callouts	

- **错误处理**: 远程系统交接最终进程的调用,因此调用仅处理远程服务的初始调用中的异常.例如,如果没有收到来自远程调用的肯定确认,则会触发超时事件.当初始调用被交接进行异步处理时,远程系统必须处理随后出现的错误.
- **恢复**: 在这种情况下, 恢复更加复杂. 如果需求要求高质量的完成, 则必须创建自定义重试机制.

#### 方案二: Outbound messaging	

- **错误处理**: 因为此模式是异步的, 所以远程系统负责错误处理. 对于出站消息, 如果在超时期内未收到肯定的确认,则 Salesforce 发起重试操作, 时间最长为24小时.

错误处理必须在远程服务中执行,因为该消息以 *fire-and-forget* 的方式有效地移交给远程系统.

- **恢复**: 因为此模式是异步的,系统必须根据服务质量要求启动重试.对于出站消息,如果在超时时间内未从出站监听器收到肯定的确认,Salesforce 将启动重试,最长达24小时.重试间隔随时间呈指数增长, 从15秒间隔开始, 到60分钟间隔结束. 超时时间可以通过向 Salesforce 支持部门请求延长至七天, 但自动重试仅限于24小时. 所有24小时后失败的消息都将放置在队列中, 管理员必须监视此队列以查找超过24小时交付期限的任何消息, 并在必要时手动重试.


#### 方案三: Platform Events	

- **错误处理**: 错误处理必须由远程服务执行,因为该事件实际上已移交给远程系统进行进一步处理. 由于此模式是异步的, 因此远程系统处理消息排队,和错误处理等. 此外, 在数据库事务中无法处理平台事件. 因此, 发布的平台事件无法在事务内回滚.

- **恢复**: 由于此模式是异步的, 因此远程系统必须根据服务的服务质量要求发起重试. 与每个事件相关联的 replay ID 是原子性的, 并随着每个已发布事件的增加而增加. 可以使用此 ID (例如,基于最后成功捕获的事件)来 replay 特定事件的流. 大容量平台事件消息存储时间为 72 小时(三天). 当使用 CometD 客户端订阅一个频道时, 可以检索过去的事件消息.

### Idempotent Design Considerations(幂等性设计的注意事项)

平台事件只会被发布到总线上一次. Salesforce 端不会进行重试. 请求重播事件取决于 ESB. 平台事件的 replay ID 保持不变, ESB 可以根据 replay ID 尝试重复消息.

幂等性对于出站消息传递很重要, 因为它是异步的, 并且在未收到肯定确认时会启动重试. 因此, 远程服务必须能够以幂等方式处理来自 Salesforce 的消息.

出站消息会为每个消息发送一个唯一的 ID, 任何重试都会保持这个 ID 不变. 远程系统可以基于这个唯一的 ID 跟踪重复的消息. 同时也会发送正在更新的每条记录的唯一记录 ID, 可以用来防止重复记录创建.

*远程过程调用-请求和回复* 模式中的 [idempotent design considerations](ch01-01-request-and-Reply.html#idempotent-design-considerations) 也适用于这个模式.

### Security Considerations

任何对远程系统的调用必须保持请求的保密性,完整性和可用性. 根据你选择的解决方案, 适配不同的安全考虑因素.

#### 方案一: Apex callouts

对远程系统的调用必须保持请求的保密性,完整性和可用性.以下是在这种模式下使用 Apex SOAP 和 HTTP 调用的具体安全考虑:

- 单向SSL默认启用, 但支持使用自签名和CA签名证书来实现双向SSL, 以维护客户端和服务器的真实性.
- Salesforce 目前并不支持 WS-Security.
- 必要时,考虑使用 [Apex Crypto](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_restful_crypto.htm) 类方法中的单向哈希或数字签名,以确保请求的完整性.
- 必须通过实施适当的防火墙机制来保护远程系统.

> **Web Service 安全性(WS-Security)** 描述了对 SOAP 消息传递的改进, 即通过消息完整性,消息机密性以及单一消息认证提供保护功能.  WS-Security 机制可用于采纳各种安全模型和加密技术.
>
> WS-Security 是一个消息级别的标准, 主要用于通过 XML 数字签名保护 SOAP 消息, 通过 XML 加密保护机密性以及通过安全性令牌保护凭证传播. Web Service 安全性规范定义了保护消息完整性和机密性的功能,并且提供了使安全相关声明与消息关联的机制.
>
> WS-Security 为安全性令牌与消息的关联提供了一个通用机制. WS-Security 不需要任何特定类型的安全性令牌. 它具有可扩展性, 例如,支持多种安全性令牌格式.

#### 方案二: Outbound Messaging

对于出站消息, 默认启用了单向SSL. 但是, 双向 SSL 可以与 Salesforce outbound messaging 证书一起使用.

以下是一些额外的安全考虑因素:

- 为远程集成服务器添加 Salesforce 服务器 IP 范围到白名单中.
- 通过实施适当的防火墙机制来保护远程系统.

#### 方案三: Platform Events

对于平台事件, 订阅的外部系统必须能够对 Salesforce streaming API 进行认证.

平台事件遵循在 Salesforce 组织中配置的现有安全模型. 要订阅事件, 用户需要对事件实体具有读取访问权限. 要发布事件, 用户需要在事件实体上具有创建权限.

## Sidebars

### Timeliness

对于即发即弃模式,及时性不是一个重要因素. 控制权立即或在成功将消息交付到远程系统后得到积极确认后即返回给客户端. 对于 Salesforce 出站消息, 确认必须在 24 小时内发生(可以延长到 7 天); 否则, 消息过期. 对于平台事件, Salesforce 将事件发送到事件总线, 而不等待订阅者的确认. 如果订阅者没有收到消息, 订阅者可以使用事件 reply ID 请求重播事件. 事件消息会存储 72 小时(三天). 要检索过去的事件消息, 请使用 CometD 客户端订阅频道.

### Data Volumes

数据量的考虑取决于您选择的解决方案.关于每个解决方案的限制,请参考 [Salesforce Limits Quick Reference Guide](https://developer.salesforce.com/docs/atlas.en-us.242.0.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_overview.htm).

### Endpoint Capability and Standards Support

对 Endpoint 的能力和标准支持取决于你选择的解决方案.

#### 方案一: Apex SOAP callouts

**考虑因素:**

该端点必须能够通过 HTTP 处理 Web 服务调用. Salesforce 必须能够通过公共互联网访问该端点.

这个解决方案要求远程系统与 Salesforce 所支持的标准兼容. 在撰写本文时,Salesforce 为 Apex SOAP call-outs 所支持的网络服务标准是:

- WSDL 1.1
- SOAP 1.1
- WSI-Basic Profile 1.1
- HTTP

#### 方案二: Apex HTTP callouts

**考虑因素:**

该端点必须能够接收 HTTP 调用.Salesforce 必须能够通过公共互联网访问该端点. 

你可以使用 Apex 的 HTTP call-out, 使用标准的 GET, POST, PUT和 DELETE 方法来调用 REST 服务. 

#### 方案三: Outbound message

**考虑因素:**

- 该端点必须能够实现一个监听器,能够接收从 Salesforce 发送的预定义格式的 SOAP 消息.
- 远程监听器必须参与契约优先的规则,其中规则由 Salesforce 提供.
- 每个出站消息都有自己预定义的 WSDL.

#### 方案四: Platform Events

- 触发器, 流可以订阅事件. 无论事件是如何发布的, 你都可以收到事件通知.
- 使用 CometD 从外部客户端订阅平台事件. 实现自己的 CometD 客户端或使用 EMP Connector, 这是一个开源的, 由社区支持的工具, 它实现了连接到 CometD 并监听通道的所有细节. Salesforce 按照接收顺序依次向 CometD 客户端发送平台事件. 事件通知的顺序基于事件的 replay ID.

### State Management

在集成系统时, 唯一键对于持续状态的跟踪是很重要的. 这里有两个选项:

- Salesforce 存储远程系统的主键或唯一标识符来标识远程记录.
- 远程系统存储 Salesforce 唯一的记录ID或其他唯一的标识符.

下表列出了这种模式下状态管理的注意事项:

| **Master System** | **Description**                                                                        |
|-------------------|----------------------------------------------------------------------------------------|
| Salesforce        | 远程系统存储 Salesforce 的 RecordId 或其他一些来自记录的唯一键.                         |
| Remote system     | Salesforce 必须在远程系统中存储一个唯一标识符的引用.因为这个过程是异步的,存储这个唯一标识符不能成为原始事务的一部分. <br/><br/>Salesforce 在调用远程过程时必须提供唯一ID. 然后, 远程系统必须通过回调方式调用 Salesforce, 使用 Salesforce 的唯一ID来更新 Salesforce 中的记录以存储该远程系统的唯一标识符. <br/><br/> 回调意味着在远程应用程序中进行特定的状态处理,以存储该事务的 Salesforce 唯一标识符, 以便在处理完成后用于回调, 或者 Salesforce 唯一标识符存储在远程系统的记录中.|

### Complex Integration Scenarios

这个模式中的每个解决方案对复杂的集成场景都有不同的考虑:

#### 方案一: Apex Callout

在某些情况下,该模式规定的解决方案可能需要实现多个复杂的集成方案. 最好使用中间件或让 Salesforce 调用一个组合服务来实现. 这些场景包括: 

- 涉及复杂流程逻辑的业务流程和规则的编排
- 对多个系统的调用及其结果进行汇总
- 入站和出站消息的转换
- 在对多个系统的调用之间保持事务完整性

#### 方案二: Outbound messaging	

鉴于出站消息的静态,声明性质, 在 Salesforce 中不能执行复杂的集成方案, 如聚合,协调或转换. 远程系统或中间件必须处理这些类型的操作.

#### 方案三: Platform Events	

鉴于事件的静态,声明性质, 在 Salesforce 中不能执行复杂的集成方案, 如聚合,协调或转换. 远程系统或中间件必须处理这些类型的操作.

### Governor Limits

由于Salesforce平台是多租户的性质, 因此对于出站调用存在一些限制. 这些限制取决于出站调用的类型和调用的时间.

如果是平台事件, 适用不同的限制和分配. 参考[Platforms Events Developer Guide.](https://developer.salesforce.com/docs/atlas.en-us.242.0.platform_events.meta/platform_events/platform_events_intro.htm).

出站消息没有具体的限制. 参考[Salesforce Limits Quick Reference Guide](https://developer.salesforce.com/docs/atlas.en-us.242.0.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_overview.htm).

### Reliable Messaging

可靠的消息传递试图解决保证向远程系统传递消息的问题, 其中的各个组件是不可靠的. 确保远程系统收到消息的方法取决于你选择的解决方案.

| **方案**           | **可靠的信息传递的考虑因素**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Apex callouts      | Salesforce 不提供对可靠消息协议(例如 WS-ReliableMessaging)的明确支持.我们建议接收 Salesforce 消息的远程端点实现可靠的消息系统,例如 JMS 或 MQ. 该系统可以确保将消息完全端到端地传递给最终处理消息的远程系统,并具有保证交付的功能.但是,请注意该系统并不能确保从 Salesforce 到被调用的远程端点的可靠传递. <br><br>确保交付必须通过对 Salesforce 进行自定义处理来实现.需要实现特定的技术, 例如在自定义重试逻辑之外,还要处理远程端点发送的肯定确认.                                                                                                                                                                                                                                                   |
| Outbound messaging | 出站消息提供了一种可靠的消息传递方式.如果从远程系统没有收到肯定确认,该过程会重试最多24小时.该过程仅保证将消息发送到远程侦听器.重试间隔会按指数增加,并以15秒间隔开始,以60分钟间隔结束.如果需要,整个重试周期可以通过向 Salesforce 支持团队发送请求来延长至7天,但自动重试只限于24小时.所有在24小时后失败的消息都会被放入队列中,管理员必须监视此队列,以检查是否有任何超过24小时交付期限的消息,并在必要时手动重试.<br><br>在大多数实现中,远程侦听器会调用另一个远程服务. 理想情况下, 通过可靠的消息系统调用远程服务可以确保完全端到端的可靠传递.对于 Salesforce 出站消息,肯定确认发生在远程侦听器已成功将其自己的消息放置在本地队列之后.一旦Salesforce收到肯定确认,自动重试就会停止. |
| Platform Events    | 平台事件通过暂时将事件消息保留在事件总线中来提供可靠的消息传递.订阅者可以使用事件消息的 Replay ID 从事件总线重播消息, 以便匹配到错过的事件消息.<br><br>事件总线是一个分布式系统,并不像事务性数据库一样具有相同的保证性.因此,Salesforce 无法为事件发布请求提供同步响应.事件被排队和缓冲, 并尝试异步地发布事件. 在极少数情况下, 事件消息可能在初始或后续尝试期间未被保存在分布式系统中.这意味着事件未被传递给订阅者,且无法恢复.                                                                                                                                                                                                                                                                 |
### Middleware Capabilities

下面的表格强调了一个中间件系统在这种模式中所应具备的理想特性.

| **Property**                                                                      | Mandatory                        | **Desirable** | **Not Required** |
|-----------------------------------------------------------------------------------|----------------------------------|---------------|------------------|
| Event handling                                                                    |                                  | X             |                  |
| Protocol conversion                                                               |                                  | X             |                  |
| Translation and transformation                                                    |                                  | X             |                  |
| Queuing and buffering                                                             | X                                |               |                  |
| Synchronous transport protocols                                                   |                                  |               | X                |
| Asynchronous transport protocols                                                  | X                                |               |                  |
| Mediation routing                                                                 |                                  | X             |                  |
| Process choreography and service orchestration                                    |                                  | X             |                  |
| Transactionality (encryption, signing, reliable delivery, transaction management) | X                                |               |                  |
| Routing                                                                           |                                  |               | X                |
| Extract, transform, and load                                                      |                                  |               | X                |
| Long Polling                                                                      | X (required for platform events) |               |                  |

### 解决方案变体 - 平台事件:发布行为和事务

当平台事件消息为立即发布时,事件发布不会关心发布过程的事务边界. 在事务完成之前或即使事务失败, 事件消息都可以被发布. 这种行为可能会导致问题, 如果订阅者期望在发布事务提交后找到数据, 但是当订阅者接收事件消息时, 可能会发现数据并不存在. 为解决此问题, 请在事件定义中将平台事件发布行为设置为 "提交后发布". 可以设置在平台事件定义中的发布行为有:

- 选择 **提交后发布** 选项,可以在事务成功提交后才发布事件消息.如果订阅者依赖于发布事务提交的数据,请选择此选项.例如,一个进程发布了一个事件消息并创建了一个任务记录.订阅该事件的第二个进程被触发,并期望找到这个任务记录.选择此行为的另一个原因是您不希望在事务失败时发布事件消息.
- 选择 **立即发布** 选项,可以在发布调用执行时发布事件消息.如果您希望不管事务是否成功都要发布事件消息,请选择此选项.如果发布者和订阅者是独立的,并且订阅者不依赖于发布者提交的数据,请选择此选项.例如,立即发布行为适用于用于记录目的的事件.使用此选项,订阅者可能会在发布者事务提交数据之前收到事件消息.

### 解决方案变体 - 出站消息和消息排序

Salesforce出站消息传递不能保证其消息的交付顺序,因为一条消息可以在24小时内重试.在远程系统中,有多种处理消息顺序的方法.

- Salesforce 为每个出站消息实例发送一个唯一的消息ID. 远程系统会丢弃具有重复消息ID的消息.
- Salesforce 只发送 RecordId. 远程系统对 Salesforce 进行回调, 以获得处理请求的必要数据.

### 解决方案变体 - 出站消息和删除

Salesforce工作流规则无法跟踪记录的删除.规则只能跟踪记录的插入或更新.因此,您不能从记录的删除直接启动出站消息.但是,您可以通过以下过程间接地触发消息.

- 创建一个自定义对象来存储被删除记录的关键信息.
- 创建一个Apex触发器,在记录被删除时触发, 以便存储信息, 例如在自定义对象中的唯一标识符.
- 实施一个工作流规则, 在创建自定义对象记录的基础上启动一个出站消息.

重要的是，通过在 Salesforce 中存储远程系统的唯一标识符或在远程系统中存储 Salesforce 的唯一标识符来启用状态跟踪。

## Example

一家电信公司希望使用 Salesforce 作为前端，通过从潜在客户到商业机会的过程创建帐户。当商业机会是 Close & Win 时，Salesforce 中将创建一个订单，但是后端 ERP 系统是数据主系统。订单必须保存到 Salesforce 业务机会记录中，并通过更改业务机会的状态以表明已创建订单。

适用以下限制条件:

- 不需要在 Salesforce 中进行定制开发.
- 您不需要在业务机会转换为订单后立即收到订单编号的通知.
- 该组织拥有支持 CometD 协议并能够订阅平台事件的 ESB.

这个例子最好使用Salesforce平台事件实现，但需要 ESB 订阅该平台事件.

在 Salesforce 这边:

- 创建一个 Process Builder 流程来启动平台事件（例如，当机会状态改变为Close-Won）.
- 创建一个新的平台事件, 发布业务机会的详细信息.

在远程系统一侧:

- ESB 使用 CometD 协议订阅了 Salesforce 平台事件.
- ESB会收到一个或多个通知，表明该机会将被转换为订单.
- ESB将消息转发到后端 ERP 系统, 这样就可以创建订单.
- 在ERP系统中创建订单后，将使用SessionId作为认证令牌，通过单独的线程回调到Salesforce系统，以更新对应机会的订单编号和状态。您可以使用Salesforce平台事件、Salesforce SOAP API、REST API或Apex Web服务等经过文档化的模式解决方案进行回调操作.

这个例子展示了以下内容:

- 异步调用远程过程的实现.
- 端到端的可靠传递.
- 对Salesforce的后续回调，以更新记录的状态.



