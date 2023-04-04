- [Remote Process Invocation—Request and Reply](#remote-process-invocationrequest-and-reply)
  - [Context](#context)
  - [Problem](#problem)
  - [Forces](#forces)
  - [Solution](#solution)
    - [Option 1 (Match: Best)](#option-1-match-best)
    - [Option 2 (Match: Best)](#option-2-match-best)
    - [Option 3 (Match: Best)](#option-3-match-best)
    - [Option 4 (Match: Suboptimal)](#option-4-match-suboptimal)
    - [Option 5 (Match: Suboptimal)](#option-5-match-suboptimal)
  - [Sketch](#sketch)
  - [Results](#results)
    - [Calling Mechanisms](#calling-mechanisms)
    - [Error Handling and Recovery](#error-handling-and-recovery)
    - [Idempotent Design Considerations](#idempotent-design-considerations)
    - [Security Considerations](#security-considerations)
  - [Sidebars](#sidebars)
    - [Timeliness](#timeliness)
    - [Data Volumes](#data-volumes)
    - [Endpoint Capability and Standards Support](#endpoint-capability-and-standards-support)
    - [State Management](#state-management)
    - [Complex Integration Scenarios](#complex-integration-scenarios)
    - [Governor Limits](#governor-limits)
    - [Middleware Capabilities](#middleware-capabilities)
  - [Example](#example)

# Remote Process Invocation—Request and Reply

## Context

您使用 Salesforce 来跟踪潜在客户,管理您的销售渠道,创建业务机会. 但是,Salesforce 系统不包含或处理订单. 在 Salesforce 中获取到订单详细信息后, 将在远程系统中创建订单, 该系统管理订单直至流程结束.

当您在项目中实施这种模式时,Salesforce 调用远程系统来创建订单, 然后等待订单成功创建. 如果成功, 远程系统会同步响应订单状态和订单号. 作为同一事务的一部分, Salesforce 在内部更新订单号和状态.订单号被用作外键, 用于随后对远程系统的更新.

## Problem

当 Salesforce 中发生事件时,如何启动远程系统中的进程,将所需信息传递给该进程,接收来自远程系统的响应,然后使用该响应数据更新 Salesforce 的数据?

## Forces

在使用基于这一模式的解决方案时,要考虑以下因素.

- 对远程系统的调用是否要求 Salesforce 在继续处理之前等待响应? 对远程系统的调用是同步请求-回复还是异步请求?
- 如果对远程系统的调用是同步的, Salesforce 是否必须将响应作为初始调用的同一事务的一部分来处理?
- 信息量是小还是大?
- 集成是基于特定事件的发生, 如 Salesforce 用户界面上的按钮点击, 还是基于 DML 的事件?
- 远程系统的 API 是否能够以低延迟响应请求? 在高峰期,有多少用户可能会执行这项交易?

## Solution

下面将包含了这些集成问题的解决方案.

### Option 1 (Match: Best)

增强型的外部服务(External Services)调用 REST API 

**建议:**

增强型外部服务允许您以声明的方式调用外部托管服务(无需代码). 这个功能最好在满足以下条件时使用:

- 外部托管的服务是一个 RESTful 服务,其定义以 OpenAPI 2.0 JSON 的格式提供.
- 请求和响应定义包含原始数据类型, 例如布尔值, 日期时间, 双精度, 整数, 字符串或原始数据类型数组. 支持嵌套对象类型和发送参数, 例如 HTTP 请求中的标头.
- 该事务可以从一个 flow 中调用.

### Option 2 (Match: Best)

- Salesforce Lightning: Lightning 组件或页面启动了一个同步的 Apex SOAP 或 REST 的 Call-out.
- Salesforce Classic: 自定义 Visualforce 页面或按钮会启动一个同步的Apex SOAP Call-out.
- 如果远程 API 构成了高延迟响应的风险(关于这里适用的限制,请参考最新的文档),那么建议使用异步Call-out,也称为 [Continuation](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/apex_continuations.htm), 以避免达到同步 Apex 事务治理者的限制.

**建议:**

Salesforce 使您能够使用 WSDL 并生成结果代理 Apex 类. 此类提供调用远程服务所需的逻辑.

Salesforce 还允许您使用标准的 GET, POST, PUT 和 DELETE 方法来调用 HTTP(REST) 服务.

Visualforce 页面或 Lightning 页面上的用户发起的动作会调用 Apex 控制器, 然后执行这个代理 Apex 类来执行远程调用. Visualforce 页面和 Lightning 页面需要自定义 Salesforce 应用程序.

### Option 3 (Match: Best)

一个自定义的 Visualforce 页面或按钮会启动一个同步的Apex HTTP Call-out.

**建议:**

Salesforce 使您能够使用标准的 GET,POST,PUT 和 DELETE 方法调用 HTTP 服务. 您可以使用多个 HTTP 类来与 RESTful 服务集成. 也可以通过手动构造 SOAP 消息来集成到基于 SOAP 的服务. 不推荐使用后者,因为 Salesforce 可能会使用 WSDL 来生成代理类.

Visualforce 页面上的用户启动操作随后调用 Apex 控制器操作,该操作随后执行此代理 Apex 类以执行远程调用. Visualforce 页面需要自定义 Salesforce 应用程序.

### Option 4 (Match: Suboptimal)

从 Salesforce 数据的更改来调用同步触发器去执行异步 Apex SOAP 或 HTTP Call-out.

**建议:**

您可以使用 Apex 触发器根据记录数据的变化来执行自动化.

一个 Apex 代理类可以通过使用 Apex 触发器作为 DML 操作的结果来执行. 但是, 在触发器上下文中进行的所有调用都必须异步执行. 因此, 不建议在这种集成问题中使用此解决方案. 此解决方案更适合于 Remote Process Invocation—Fire and Forget 模式.

### Option 5 (Match: Suboptimal)

Batch Apex job 执行一个同步的Apex SOAP或HTTP Call-out.

**建议:**

你可以从一个批处理作业中对远程系统进行调用.这种解决方案允许在 Salesforce 中批量执行远程流程并处理来自远程系统的响应.然而,一个给定的批处理对调用的数量有限制.欲了解更多信息,请阅读 [Governor Limits](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_remote_process_invocation_state.htm#governor_limits_header).

给定的批处理运行可以执行多个事务上下文(通常以 200 条记录为间隔). 每个事务上下文都会重置 limits.

## Sketch

此图说明了使用 Apex 进行同步远程流程调用.

<p align="center">
    <img src="https://github.com/dyncan/Integration-Patterns-and-Practices-ZH_CN/blob/main/src/img/pattern-callout-to-remote-system-flow.png?raw=true">
</p>

执行流程:

  1. 在 Visualforce 或 Lightning 页面上触发一个动作(例如: 点击按钮).
  2. 浏览器(在 Lightning 组件中, 通过客户端控制器)执行 HTTP POST,然后在相应的 Apex 控制器上执行具体操作.
  3. 控制器执行对远程服务的实际调用.
  4. 来自远程系统的响应被返回到 Apex 控制器.控制器处理该响应,根据实际场景来更新 Salesforce 中的数据, 并重新将响应数据显示在页面上.

在必须跟踪后续状态的情况下, 远程系统会返回存储在 Salesforce 记录中的唯一标识符.

## Results

与此模式相关的解决方案的应用允许事件触发的远程流程调用, 其中 Salesforce 处理了进程的过程.

### Calling Mechanisms

调用机制取决于为实现这一模式而选择的解决方案.

| **Calling Mechanism**                                                                  | **Description**                                                                                                                                                               |
|----------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 增强型的外部服务嵌入到 flow <br>或 Lightning 组件<br>或 Visualforce和 Apex controllers | 当远程流程作为涉及用户界面的端到端进程的一部分被触发时使用,并且结果必须在 Salesforce 记录中显示或更新. <br>例如: 向外部支付网关提交信用卡支付,支付结果立即返回显示给用户. |
| Apex triggers                                                                          | 主要用于使用Apex调用 DML 发起的事件来调用远程流程.关于这种调用机制的更多信息,请参见模式: Remote Process Invocation—Fire and Forget.                                         |
| Apex batch classes                                                                     | 用于批量调用远程流程.关于这种调用机制的更多信息,请参见模式: Remote Process Invocation—Fire and Forget.                                                                      |

### Error Handling and Recovery

将错误处理和恢复策略作为整体解决方案的一部分是很重要的.

- **错误处理**-当错误发生时(异常或错误代码被返回给调用者), 调用者来管理错误信息的处理. 例如: 在终端用户的页面上显示错误信息, 或记录到一个需要进一步操作的 object 中.
- **恢复**-在调用者收到成功的响应之前, 数据的响应变化不会被提交到 Salesforce .例如: 在收到表示成功的响应之前, 数据库中的订单状态不会被更新. 如果有必要, 调用者可以重试该操作.

### Idempotent Design Considerations

幂等能力可以保证重复调用是安全的. 如果没有实现幂等性, 同一条消息的多次调用可能会产生不同的结果, 可能导致数据完整性问题. 潜在的问题包括创建重复记录或重复处理事务.

确保被调用的远程过程具有幂等性非常重要. 如果调用是由用户界面事件触发的, 则几乎不可能保证 Salesforce 只调用一次. 即使 Salesforce 只进行了一次调用, 也不能保证其他进程(例如中间件)会执行相同的操作.

构建幂等接收者的最典型方法是让它根据消费者发送的唯一消息标识符来跟踪重复项. 必须自定义 Apex web service 或 REST Call 来发送唯一的消息 ID.

此外, 在远程系统中创建记录的操作必须在插入之前进行重复项检查. 通过传递来自 Salesforce 的唯一记录 ID 进行检查. 如果该记录存在于远程系统中, 则更新该记录. 在大多数系统中, 此操作称为 Upsert 操作.

### Security Considerations

任何对远程系统的调用必须保持请求的保密性,完整性和可用性. 以下是在这种模式下使用 Apex SOAP 和 HTTP 调用的具体安全考虑.

- 单向SSL默认启用,但支持使用自签名和CA签名证书来实现双向SSL, 以维护客户端和服务器的真实性.
- Salesforce 目前并不支持 WS-Security.
- 必要时,考虑使用 [Apex Crypto](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_restful_crypto.htm) 类方法中的单向哈希或数字签名,以确保请求的完整性.
- 必须通过实施适当的防火墙机制来保护远程系统.

> **Web Service 安全性(WS-Security)** 描述了对 SOAP 消息传递的改进, 即通过消息完整性,消息机密性以及单一消息认证提供保护功能.  WS-Security 机制可用于采纳各种安全模型和加密技术.
>
> WS-Security 是一个消息级别的标准, 主要用于通过 XML 数字签名保护 SOAP 消息, 通过 XML 加密保护机密性以及通过安全性令牌保护凭证传播. Web Service 安全性规范定义了保护消息完整性和机密性的功能,并且提供了使安全相关声明与消息关联的机制.
>
> WS-Security 为安全性令牌与消息的关联提供了一个通用机制. WS-Security 不需要任何特定类型的安全性令牌. 它具有可扩展性, 例如,支持多种安全性令牌格式.

## Sidebars

### Timeliness

在这种模式下,及时性很重要.通常情况下:

- 请求通常是从用户界面发起的, 因此该过程不能让用户等待.
- Salesforce 对来自 Apex 的 call 有一个可配置的超时时间, 最长可达 _120_ 秒.
- 及时地完成远程过程的调用,在 Salesforce 超时限制内结束.
- 外部调用受 Apex 同步事务限制, 因此请确保减轻实例化超过 10 个事务并且每个事务运行时间超过 5 秒钟的风险. 除了确保外部 API 具有良好的性能之外, 降低超时风险的选项还包括:
  - 将外部 Call-out 的超时时间设置为 5 秒
  - 在 Visualforce 或 Lightning 组件中使用 `[continuation](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_class_System_Continuation.htm)` 来处理长时间运行的事务.

### Data Volumes

由于 Apex call 解决方案的请求或响应的超时值较小, 因此此模式主要用于小批量实时活动. 不要在消息中包含数据 payload 的批处理请求中使用此模式.

### Endpoint Capability and Standards Support

对 endpoint 的能力和标准支持取决于你选择的解决方案.

| **Solution**       | _Endpoint Considerations_                                                                                                                                                                                                                                                                                     |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Apex SOAP callouts | 该 endpoint 必须能够通过HTTP接收 web service 调用.Salesforce 必须能够通过公共互联网访问该 endpoint. <br>这个解决方案要求远程系统与 Salesforce 所支持的标准兼容. 在撰写本文时,Salesforce 为 Apex SOAP call-outs 所支持的网络服务标准是:<br>1. WSDL 1.1<br>2. SOAP 1.1<br>3. WSI-Basic Profile 1.1<br>4. HTTP |
| Apex HTTP callouts | 该端点必须能够接收 HTTP 调用.Salesforce 必须能够通过公共互联网访问该端点. <br>你可以使用 Apex 的HTTP call-out, 使用标准的GET,POST,PUT和DELETE方法来调用 REST 服务.                                                                                                                                        |

### State Management

在集成系统时, 唯一键对于持续状态的跟踪是很重要的. 这里有两个选项:

- Salesforce 存储远程系统的主键或唯一标识符来标识远程记录.
- 远程系统存储 Salesforce 唯一的记录ID或其他唯一的标识符.

根据包含主记录的系统,处理系统集成唯一键有一些具体注意事项,如下表所示:

| **Master System** | **Description**                                                                        |
|-------------------|----------------------------------------------------------------------------------------|
| Salesforce        | 远程系统存储 Salesforce 的RecordId或其他一些来自记录的唯一键.                         |
| Remote system     | 对远程进程的调用从应用程序返回唯一的键,Salesforce 将该键值存储在一个唯一的记录字段中. |

### Complex Integration Scenarios

在某些情况下,该模式规定的解决方案可能需要实现多个复杂的集成方案. 最好使用中间件或让 Salesforce 调用一个组合服务来实现. 这些场景包括: 

- 涉及复杂流程逻辑的业务流程和规则的编排
- 对多个系统的调用及其结果进行汇总
- 入站和出站消息的转换
- 在对多个系统的调用之间保持事务完整性

### Governor Limits

有关 Apex 限制的信息,请查看[Execution Governors and Limits](https://developer.salesforce.com/docs/atlas.en-us.242.0.apexcode.meta/apexcode/apex_gov_limits.htm)

### Middleware Capabilities

下表显示了构建此模式的中间件系统的理想特性.

| Property                                                                          | Mandatory | Desirable | Not Required |
|-----------------------------------------------------------------------------------|-----------|-----------|--------------|
| Event handling                                                                    |           | X         |              |
| Protocol conversion                                                               |           | X         |              |
| Translation and transformation                                                    |           | X         |              |
| Queuing and buffering                                                             |           | X         |              |
| Synchronous transport protocols                                                   | X         |           |              |
| Asynchronous transport protocols                                                  |           |           | X            |
| Mediation routing                                                                 |           | X         |              |
| Process choreography and service orchestration                                    |           | X         |              |
| Transactionality (encryption, signing, reliable delivery, transaction management) |           | X         |              |
| Routing                                                                           |           |           | X            |
| Extract, transform, and load                                                      |           |           | X            |
| Long polling                                                                      |           |           | X            |


## Example

一家公用事业公司使用 Salesforce, 并有一个单独的系统包含客户的账单信息. 他们想显示客户账户的账单历史, 而不在 Salesforce 中存储这些数据. 他们有一个现有的 Web Service, 可以返回一个给定账户的账单列表和细节, 但不能在浏览器中显示这些数据.

这一要求可以通过以下方法来实现:

- Salesforce 使用 Apex 代理类中的消费账单历史服务 WSDL.
- 通过创建一个 Lightning 组件和一个自定义 Controller 或一个 Visualforce 页面和自定义 Controller, 执行 Apex 代理类, 并将账号作为唯一的标识符.
- 然后, 自定义 controller 解析来自 Apex call-out 和 Lightning 组件或 Visualforce 页面的返回值, 然后将客户的账单数据呈现给用户.

这个例子证明了:

- 使用存储在 Salesforce Account 对象上的帐号跟踪客户的状态.
- 呼叫方对响应信息进行后续处理.







