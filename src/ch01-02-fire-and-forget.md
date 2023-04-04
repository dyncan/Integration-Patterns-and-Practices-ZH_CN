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



