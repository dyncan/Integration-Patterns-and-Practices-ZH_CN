- [Design Pattern Catalog](#design-pattern-catalog)
  - [Remote Process Invocation—Request and Reply](#remote-process-invocationrequest-and-reply)
    - [Context](#context)
    - [Problem](#problem)
    - [Forces](#forces)
    - [Solution](#solution)
      - [方案一 (方案匹配度: Best)](#方案一-方案匹配度-best)
      - [方案二 (方案匹配度: Best)](#方案二-方案匹配度-best)
      - [方案三 (方案匹配度: Best)](#方案三-方案匹配度-best)
      - [方案四 (方案匹配度: Suboptimal)](#方案四-方案匹配度-suboptimal)
      - [方案四 (方案匹配度: Suboptimal)](#方案四-方案匹配度-suboptimal-1)
    - [Sketch](#sketch)
    - [Results](#results)

# Design Pattern Catalog

## Remote Process Invocation—Request and Reply

### Context

您使用 Salesforce 来跟踪潜在客户,管理您的销售渠道,创建业务机会. 但是,Salesforce 系统不包含或处理订单. 在 Salesforce 中获取到订单详细信息后, 将在远程系统中创建订单, 该系统管理订单直至流程结束.

当您在项目中实施这种模式时,Salesforce 调用远程系统来创建订单, 然后等待订单成功创建. 如果成功, 远程系统会同步响应订单状态和订单号. 作为同一事务的一部分, Salesforce 在内部更新订单号和状态.订单号被用作外键, 用于随后对远程系统的更新.

### Problem

当 Salesforce 中发生事件时,如何启动远程系统中的进程,将所需信息传递给该进程,接收来自远程系统的响应,然后使用该响应数据更新 Salesforce 的数据?

### Forces

在使用基于这一模式的解决方案时,要考虑以下因素.

- 对远程系统的调用是否要求 Salesforce 在继续处理之前等待响应? 对远程系统的调用是同步请求-回复还是异步请求?
- 如果对远程系统的调用是同步的, Salesforce 是否必须将响应作为初始调用的同一事务的一部分来处理?
- 信息量是小还是大?
- 集成是基于特定事件的发生, 如 Salesforce 用户界面上的按钮点击, 还是基于 DML 的事件?
- 远程系统的 API 是否能够以低延迟响应请求? 在高峰期,有多少用户可能会执行这项交易?

### Solution

下面将包含了这些集成问题的解决方案.

#### 方案一 (方案匹配度: Best)

增强型的外部服务(External Services)调用 REST API 

**建议:**

增强型外部服务允许您以声明的方式调用外部托管服务(无需代码). 这个功能最好在满足以下条件时使用:

- 外部托管的服务是一个 RESTful 服务,其定义以 OpenAPI 2.0 JSON 的格式提供.
- 请求和响应定义包含原始数据类型, 例如布尔值, 日期时间, 双精度, 整数, 字符串或原始数据类型数组. 支持嵌套对象类型和发送参数, 例如 HTTP 请求中的标头.
- 该事务可以从一个 flow 中调用.

#### 方案二 (方案匹配度: Best)

- Salesforce Lightning: Lightning 组件或页面启动了一个同步的 Apex SOAP 或 REST 的 Call-out.
- Salesforce Classic: 自定义 Visualforce 页面或按钮会启动一个同步的Apex SOAP Call-out.
- 如果远程 API 构成了高延迟响应的风险(关于这里适用的限制,请参考最新的文档),那么建议使用异步Call-out,也称为 [Continuation](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/apex_continuations.htm), 以避免达到同步 Apex 事务治理者的限制.

**建议:**

Salesforce 使您能够使用 WSDL 并生成结果代理 Apex 类. 此类提供调用远程服务所需的逻辑.

Salesforce 还允许您使用标准的 GET, POST, PUT 和 DELETE 方法来调用 HTTP(REST) 服务.

Visualforce 页面或 Lightning 页面上的用户发起的动作会调用 Apex 控制器, 然后执行这个代理 Apex 类来执行远程调用. Visualforce 页面和 Lightning 页面需要自定义 Salesforce 应用程序.

#### 方案三 (方案匹配度: Best)

一个自定义的 Visualforce 页面或按钮会启动一个同步的Apex HTTP Call-out.

**建议:**

Salesforce 使您能够使用标准的 GET,POST,PUT 和 DELETE 方法调用 HTTP 服务. 您可以使用多个 HTTP 类来与 RESTful 服务集成. 也可以通过手动构造 SOAP 消息来集成到基于 SOAP 的服务. 不推荐使用后者,因为 Salesforce 可能会使用 WSDL 来生成代理类.

Visualforce 页面上的用户启动操作随后调用 Apex 控制器操作,该操作随后执行此代理 Apex 类以执行远程调用. Visualforce 页面需要自定义 Salesforce 应用程序.

#### 方案四 (方案匹配度: Suboptimal)

从 Salesforce 数据的更改来调用同步触发器去执行异步 Apex SOAP 或 HTTP Call-out.

**建议:**

您可以使用 Apex 触发器根据记录数据的变化来执行自动化.

一个 Apex 代理类可以通过使用 Apex 触发器作为 DML 操作的结果来执行. 但是, 在触发器上下文中进行的所有调用都必须异步执行. 因此, 不建议在这种集成问题中使用此解决方案. 此解决方案更适合于 Remote Process Invocation—Fire and Forget 模式.

#### 方案四 (方案匹配度: Suboptimal)

Batch Apex job 执行一个同步的Apex SOAP或HTTP Call-out.

**建议:**

你可以从一个批处理作业中对远程系统进行调用.这种解决方案允许在 Salesforce 中批量执行远程流程并处理来自远程系统的响应.然而,一个给定的批处理对调用的数量有限制.欲了解更多信息,请阅读 [Governor Limits](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_remote_process_invocation_state.htm#governor_limits_header).

给定的批处理运行可以执行多个事务上下文(通常以 200 条记录为间隔). 每个事务上下文都会重置 limits.

### Sketch

此图说明了使用 Apex 进行同步远程进程调用.

<p align="center">
    <img src="https://github.com/dyncan/Integration-Patterns-and-Practices-ZH_CN/blob/main/src/img/pattern-callout-to-remote-system-flow.png?raw=true">
</p>

执行流程:

  1. 在 Visualforce 或 Lightning 页面上触发一个动作(例如: 点击按钮).
  2. 浏览器(在 Lightning 组件中, 通过客户端控制器)执行 HTTP POST,然后在相应的 Apex 控制器上执行具体操作.
  3. 控制器执行对远程服务的实际调用.
  4. 来自远程系统的响应被返回到 Apex 控制器.控制器处理该响应,根据实际场景来更新 Salesforce 中的数据, 并重新将响应数据显示在页面上.

在必须跟踪后续状态的情况下, 远程系统会返回存储在 Salesforce 记录中的唯一标识符.

### Results


