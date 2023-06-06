# 远程系统请求呼入

## Context

您使用 Salesforce 跟踪潜在客户, 管理销售渠道, 创建商机, 并捕捉转化潜在客户为客户的订单细节. 但是, Salesforce 并非包含或处理订单的系统. 订单由外部(远程)系统进行管理. 该远程系统需要在订单通过其处理阶段时在 Salesforce 中更新订单状态.

## Problem

远程系统如何与 Salesforce 建立连接和完成认证,以便通知与 Salesforce 有的关外部事件, 比如: 创建记录和更新现有记录的情况?

## Forces

应用基于这种模式的解决方案时,需要考虑各种因素.

- 远程调用 Salesforce 的目的是使用事件驱动架构将外部发生的事件通知到 Salesforce 吗?还是为了在特定记录上执行 CRUD 操作?如果您使用事件驱动架构,事件生产者(远程进程)与Salesforce 的事件消费者是解耦的.
- Salesforce 的调用是否要求远程进程在继续处理之前等待响应?远程调用 Salesforce 的请求始终是同步的请求-响应过程, 但如果不需要响应来模拟异步调用, 则远程进程可以丢弃响应.
- 每个交易是针对单个 Salesforce 对象还是多个相关对象进行操作?
- 消息的格式是什么(例如,SOAP 或 REST, 或两者都是通过 HTTP)?
- 消息的大小是较小还是较大?
- 如果远程系统支持 SOAP 协议的功能,那么远程系统能够参与以契约为先的方法吗?即 Salesforce 主导契约的方法.这里可以使用的 Salesforce 的 SOAP API,因为 Salesforce 提供了预定义的 WSDL.
- 是否需要事务处理?
- 你对 Salesforce 的定制化程度有多高容忍?

## Solution

下面内容包含了这个集成问题的各种解决方案.

1. SOAP API(Fit: Best)
   
   **Solution:**

   - 可访问性: Salesforce 提供了一个 SOAP API, 远程系统可以使用它来:
     - 发布事件以通知你的 Salesforce 组织.
     - 查询你的 Org 中的数据.
     - 创建,更新和删除数据
     - 获得你的 Org 的元数据.
     - 运行实用程序执行管理任务.
  
   - 同步 API: 在进行 API 调用后, 远程客户端应用程序会等待, 直到它收到来自服务的响应. 不支持对 Salesforce 的异步调用.
  
   - 生成的WSDL: Salesforce 为远程系统提供两种 WSDL:
     - Enterprise WSDL: 提供一个强类型的 WSDL, 该 WSDL 是针对 Salesforce 组织的.
     - Partner WSDL: 包含一个松散类型的 WSDL, 该 WSDL 并不特定于 Salesforce 组织.
  
   - 安全性: 调用 SOAP API 的客户端必须有一个有效的登录,并获得一个会话以执行任何 API 调用.该 API 遵循 Salesforce 中根据登录用户的配置文件配置的对象级和字段级安全.
  
   - 事务/提交行为: 默认情况下, 每个 API 调用都允许在某些记录标记为错误时部分成功.这可以更改为 "all or nothing" 的行为, 如果发生任何错误, 则所有结果都将回滚. 无法跨多个 API 调用进行事务处理.为了克服这个限制, 单个 API 调用可以影响多个对象. 
  
   - 批量数据: 包括超过 2,000 条记录的任何数据操作都是使用 Bulk API 2.0 的一个很好的选择,可以成功地准备,执行和管理利用 Bulk 框架的异步工作流.少于2,000条记录的作业可以在 REST(例如: Composite API)或 SOAP 中进行 "批量化" 的同步调用.
  
   - 事件驱动架构: 平台事件的定义与您定义 Salesforce 对象的方式相同.通过 SOAP API 发布一个事件与创建 Salesforce 记录的方式是一样的.

2. REST API(Fit: Best)

    **Solution:**

   - 可访问性: Salesforce 提供了一个 REST API, 远程系统可以使用它来:
     - 发布事件以通知你的 Salesforce 组织.
     - 查询你的 Org 中的数据.
     - 创建,更新和删除数据
     - 获得你的 Org 的元数据.
     - 运行实用程序执行管理任务.
   
   - 同步 API: 在进行 API 调用后, 远程客户端应用程序会等待, 直到它收到来自服务的响应. 不支持对 Salesforce 的异步调用.
  
   - REST API vs SOAP API: REST 将资源(实体/对象)暴露为 URI, 并使用 HTTP 动词来定义这些资源的 CRUD 操作. 与 SOAP 不同, REST API 不需要预定义的契约, 利用 XML 和 JSON 作为响应, 并且具有松散的类型. REST API 是轻量级的, 为与 Salesforce 交互提供了一种简单的方法. 它的优点包括易于集成和开发, 而且它是与移动应用程序和Web应用程序一起使用的绝佳选择.
  
   - 安全性: 调用 REST API 的客户端必须有一个有效的登录, 并获得一个会话以执行任何 API 调用. 该 API 遵循 Salesforce 中根据登录用户的配置文件配置的对象级和字段级安全.
  
   - 事务/提交行为: 默认情况下,每个记录被视为单独的事务,并单独提交.一个记录的更改失败不会导致其他记录更改回滚.可以将此行为修改为 "all or nothing" 的行为.使用 REST API 复合资源可以在一个 API 调用中进行一系列的更新.
  
   - REST Composite Resources(REST 组合资源): 使用这些 REST API 资源在单个 API 调用中执行多个操作. 还可以使用一个调用的输出作为下一个调用的输入. 所有请求的响应体和 HTTP 状态都以单个响应体返回. 整个请求被视为对 API 限制的单个调用.

   - 批量数据: 包括超过 2,000 条记录的任何数据操作都是使用 Bulk API 2.0 的一个很好的选择,可以成功地准备,执行和管理利用 Bulk 框架的异步工作流.少于2,000条记录的作业可以在 REST(例如: Composite API)或 SOAP 中进行 "批量化" 的同步调用.

   - 事件驱动架构: 平台事件的定义与您定义 Salesforce 对象的方式相同.通过 REST API 发布一个事件与创建 Salesforce 记录的方式是一样的.

3. Apex web services(Fit: Suboptimal)

    **Solution:**

    - Apex 类的方法可以作为网络服务方法暴露给外部应用程序.这种方法是 SOAP API 的替代品, 通常只在必须满足以下额外要求的情况下使用.
      - 需要完整的事务支持(例如: 在一个事务中创建账户,联系人和机会).
      - 在提交之前, 必须在Salesforce端自定义逻辑.
  
    - 使用 Apex Web服务的好处必须与在 Salesforce 中需要维护的额外代码进行权衡.
  
    - 不适用于平台事件, 因为在事件驱动架构中, 消费者的事务预创建逻辑不适用. 要通知 Salesforce 组织发生了一个事件,可以使用 SOAP API, REST API 或 Bulk API 2.0.

4. Apex REST services(Fit: Suboptimal)

    **Solution:**

    - Apex 类可以作为 REST 资源暴露出来, 映射到特定的 URI 上, 并对其定义了 HTTP 动词(例如: POST 或 GET).
    
    - 你可以使用 REST API 复合资源,在一个事务中执行多个更新.
    
    - 与 SOAP 不同,客户端无需消费服务定义/合同(WSDL)并生成客户端存根.远程系统只需要能够构建一个 HTTP 请求并处理返回的结果(XML或JSON).

    - 不适用于平台事件, 因为在事件驱动架构中, 消费者的事务预创建逻辑不适用. 要通知 Salesforce 组织发生了一个事件,可以使用 SOAP API, REST API 或 Bulk API 2.0.


4. Bulk API 2.0	(Fit: Optimal for bulk operations)

    **Solution:**

    - Bulk API 2.0 基于 REST 原则, 并且针对加载或删除大量数据进行了优化. 它具有与 REST API 相同的可访问性和安全行为.
    
    - 任何包括超过 2,000 条记录的数据操作都是使用 Bulk API 2.0 的一个很好选择, 可以成功地准备,执行和管理利用 Bulk 框架的异步工作流. 少于2,000条记录的作业可以在 REST(例如: Composite API)或 SOAP 中进行 "批量化" 的同步调用.
    
    - Bulk API 2.0 允许客户端应用程序通过提交多个批次异步地query, insert, update, upsert, 或删除大量记录,这些记录由 Salesforce 在后台进行处理.相比之下,SOAP API 针对实时客户端应用程序进行了优化,适用于一次更新少量记录的情况.
    
    - 虽然 SOAP API 也可以用来处理大量的记录,但当数据集包含几十万到几百万条记录时,它就变得不太实用了. 这是由于其相对较高的开销和较低的性能特点造成的.
      - 事件驱动架构: 平台事件的定义与您定义 Salesforce 对象的方式相同.通过 Bulk API 2.0 发布事件与创建Salesforce记录是一样的.只支持创建和插入操作.批处理中的事件在处理批处理作业时以异步方式发布到 Salesforce 事件总线(event bus)上.

## Sketch

下图说明了当您使用 REST API 从外部事件发出通知或使用 SOAP API 查询 Salesforce 对象来实现此模式时的事件顺序.使用REST API时,事件的顺序是相同的.

<p align="center">
    <img src="img/Remote-System-Querying-Salesforce-Via-SOAP-API.png?raw=true">
</p>

Remote System Querying Salesforce Via SOAP API

<p align="center">
    <img src="img/Remote-System–Notifying-Salesforce-with-Events-Via-REST-API.png?raw=true">
</p>

Remote System Notifying Salesforce with Events Via REST API

## Results

在事件驱动架构中,远程系统使用 SOAP API,REST API或 Bulk API 2.0 调用 Salesforce,将事件发布到Salesforce事件总线.发布事件会通知所有订阅者.事件订阅者可以是 Salesforce 平台上的,如Process Builder, Flow, 或 Lightning组件, Apex触发器. 事件订阅者也可以是 Salesforce 平台的外部人员,如 CometD 的订阅者.

当直接使用Salesforce对象进行工作时,与该模式相关的解决方案可以实现以下功能:

   - 远程系统调用 Salesforce 的 API 来查询数据库并执行单对象操作(创建,更新,删除等).
   - 远程系统调用 Salesforce REST API 复合资源来执行一系列的对象操作.
   - 远程系统调用自定义构建的Salesforce API(服务),能够支持多对象事务操作和自定义的前/后处理逻辑.

### Calling Mechanisms

调用机制取决于为实现这一模式而选择的解决方案.

| **Calling mechanism**   | **Description**                                                                                                                                                                                                                                                                                                   |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SOAP API                | 远程系统使用Salesforce企业版或合作伙伴WSDL生成客户端存根,进而用于调用标准SOAP API.                                                                                                                                                                                                                               |
| REST API                | 在访问任何Apex REST服务之前,远程系统必须进行认证.远程系统可以使用OAuth 2.0或用户名和密码认证.在这两种情况下,客户端必须用适当的值设置授权HTTP头(OAuth访问令牌或会话ID可以通过对SOAP API的登录调用获得).<br><br>然后,远程系统用适当的动词生成REST调用(HTTP请求)并处理返回的结果(支持JSON和XML数据格式). |
| Apex web service        | 远程系统使用自定义的Apex Web服务WSDL来生成客户端存根,然后使用这些存根来调用自定义的Apex Web服务.                                                                                                                                                                                                                |
| Apex REST service       | 按照REST API,资源URI和适用的动词是用@RestResource,@HttpGet和@HttpPost注释来定义的.                                                                                                                                                                                                                             |
| Bulk API 2.0            | Bulk API 2.0是基于REST的API,因此与REST API调用机制类似.                                                                                                                                                                                                                                                          |
| REST API to invoke Flow | 使用REST API调用自定义可调用操作的端点,以调用自动启动的流程.                                                                                                                                                                                                                                                    |

### Error Handling and Recovery

必须考虑将错误处理和恢复策略作为整体解决方案的一部分.

- 错误处理: 所有的远程调入方法,标准或自定义API,都需要远程系统处理任何后续的错误,如超时和重试的管理.中间件可以用来提供错误处理和恢复的逻辑.
- 恢复机制 - 如果服务质量要求决定需要创建自定义重试机制,则需要确保幂等设计特性.平台事件使订阅者能够使用 Reply ID 在发布这些消息后的一定时间内获取消息.

### Idempotent Design Considerations

幂等性能确保重复调用是安全的, 不会产生负面影响. 如果没有实现幂等性, 那么对相同消息的重复调用可能会产生不同的结果, 可能导致数据完整性问题, 例如创建重复记录, 重复处理事务等.

远程系统在出现错误或超时时,必须处理多个(重复的)调用,以避免重复插入和冗余更新(尤其是如果下游触发器和工作流规则触发).虽然在Salesforce内部可以处理其中一些情况(尤其是自定义SOAP和REST服务的情况),但我们建议远程系统(或中间件)处理错误处理和幂等设计.

### Security Considerations

不同的模式解决方案会涉及不同的安全考虑.在所有情况下,该平台都使用已登录用户的访问权限(例如配置文件设置,共享规则,权限集等).此外,可以使用配置文件IP限制来限制对特定IP地址范围的API访问.

| **Solution**      | **Security considerations**                                                                                                                                                                                                                                                                                                                                                                                                                    |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SOAP API          | Salesforce支持安全套接字层v3(SSL)和传输层安全(TLS)协议.密码必须至少具有128位的密钥长度.<br><br>远程系统必须使用有效的凭证登录,以获得一个会话ID.如果远程系统已经有一个有效的会话ID,那么它就可以在没有明确登录的情况下调用API.这在本文前面涉及的回调模式中使用,前面的Salesforce出站消息包含了会话ID和记录ID,供后续使用.<br><br>我们建议调用 SOAP API 的客户端缓存会话ID,以最大限度地提高性能,而不是每次调用都获得一个新的会话ID. |
| REST API          | 我们建议远程系统建立OAuth授权信任.然后可以使用HTTP动词对特定资源进行REST调用.还可以使用通过其他方式获得的有效会话ID(例如通过调用SOAP API检索或通过出站消息提供)进行REST调用.<br><br>我们建议调用REST API的客户端缓存和重复使用会话ID以最大化性能,而不是为每个调用获取新的会话ID.                                                                                                                                                         |
| Apex web service  | 我们建议远程系统为授权建立一个OAuth信任.                                                                                                                                                                                                                                                                                                                                                                                                      |
| Apex REST service | 我们建议远程系统为授权建立一个OAuth信任.                                                                                                                                                                                                                                                                                                                                                                                                      |
| Bulk API 2.0      | 我们建议远程系统为授权建立一个OAuth信任.                                                                                                                                                                                                                                                                                                                                                                                                      |

参考 [Security Considerations](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_security_considerations.htm).

## Sidebars

### Timeliness

SOAP API和Apex web service API是同步的.以下是适用的超时时间:

- 会话超时: 如果没有任何活动, 会话就会根据 Salesforce org 的会话超时设置超时.
- 查询超时 - 每个 SOQL 查询都有 120 秒的单独超时限制.

### Data Volumes

数据量的考虑取决于你选择哪种解决方案和通信类型:

| **Solution**         | **Communication type** | **Limits**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|----------------------|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SOAP API or REST API | Synchronous            | SOAP登录-SOAP登录请求的大小被限制在10KB或以下.每个用户每小时最多可以调用3600次 login() 函数.如果你超过这个限制,API会返回一个错误.<br><br><br>创建,更新,删除--远程系统一次最多可以创建,更新或删除200条记录.可以进行多次调用以处理超过200条的记录,但每个请求的规模限制在200条.<br><br>BLOB数据-您可以使用SObject Basic Information,SObject Rows或SObject Collections REST资源来插入或更新Salesforce标准对象中的BLOB数据.对于SObject Basic Information或SObject Rows资源,ContentVersion对象的最大上传文件大小为2GB,所有其他符合条件的标准对象为500MB.使用SObject Collections资源,单个请求中所有文件的最大总大小为500 MB.<br><br>查询结果大小 — 默认情况下,在查询结果对象(批处理大小)中,查询(query())或查询更多(queryMore())调用返回的行数设置为500行.返回的最大行数为2,000行.您可以明确设置批处理大小,但不能保证请求的批处理大小将是实际的批处理大小.这样做是为了最大化性能.如果要返回的行数超过批处理大小,可以使用queryMore() API调用来迭代多个批次.可能还适用其他规则,有关更多信息,请参考 [Salesforce Developer Limits and Allocations Quick Reference](https://developer.salesforce.com/docs/atlas.en-us.244.0.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_overview.htm)<br><br>事件消息--事件消息的最大值为 1 MB.使用 Salesforce APIs 发布事件,将计入您的标准API限制. |
| Bulk API 2.0         | Synchronous            | Bulk API 2.0为异步导入和导出大型数据集进行了优化.<br><br>包括超过 2,000 条记录的任何数据操作都是使用 Bulk API 2.0 的一个很好的选择,可以成功地准备,执行和管理利用 Bulk 框架的异步工作流.少于2,000条记录的作业可以在 REST(例如: Composite API)或 SOAP 中进行 "批量化" 的同步调用.<br><br>Bulk API 2.0在提交批量请求和相关数据时是同步的.数据的实际处理是异步进行的.有关API和批处理限制的更多信息,请参考 [Limits](https://developer.salesforce.com/docs/atlas.en-us.244.0.api_asynch.meta/api_asynch/bulk_common_limits.htm).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

### Endpoint Capability and Standards Support


对 endpoint 的能力和标准支持取决于你选择的解决方案.

| **Solution**      | **Endpoint considerations**                                                                                                                                                                                    |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SOAP API          | 远程系统必须能够实现一个客户端,该客户端可以调用Salesforce SOAP API,基于Salesforce预定义的消息格式.<br><br>远程系统(客户端)必须参与基于契约的实现,其中契约由Salesforce提供(例如,企业或合作伙伴WSDL).  |
| REST API          | 远程系统必须能够实现一个REST客户端,调用Salesforce定义的REST服务,并处理XML或JSON结果.                                                                                                                        |
| Apex web service  | 远程系统必须能够实现一个客户端,可以调用Salesforce定义的预定义格式的SOAP消息.<br><br><br>远程系统必须参与基于代码的实现,其中合同是在Apex web服务实现之后由Salesforce提供的.每个Apex web服务都有自己的WSDL. |
| Apex REST service | 相同的端点考虑适用于REST API.                                                                                                                                                                                 |

### State Management

在整合系统时,主键对于持续状态跟踪非常重要,例如: 如果在远程系统中创建了一条记录,以支持对该记录的后续更新.这里有两个选项:

- Salesforce 为远程记录存储远程系统的主键或唯一标识符.
- 远程系统存储 Salesforce 唯一的记录ID或其他唯一的标识符.

在这种同步模式下,处理集成的唯一键有特定的考虑.

| **Master system** | **Description**                                                                                                                         |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Salesforce        | 在这种情况下,远程系统会存储Salesforce的RecordId或其他一些来自记录的唯一标识符.                                                        |
| Remote system     | 在这种情况下,Salesforce 在远程系统中存储对唯一标识符的引用.因为这个过程是同步的,所以键可以作为同一事务的一部分使用外部ID字段来提供. |

### Complex Integration Scenarios

在处理复杂的集成场景,例如转换和流程编排时,此模式中的每个解决方案都有不同的考虑因素.

| **Solution**                          | **Considerations**                                                                                                                                                           |
|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SOAP API or REST API                  | SOAP API和REST API提供了对对象进行简单事务处理的功能.在Salesforce中无法执行复杂的集成场景,比如聚合,编排和转换.这些场景必须由远程系统或中间件来处理,中间件是首选的方法. |
| Apex web service or Apex REST service | 自定义网络服务可以提供跨对象功能,自定义逻辑和更复杂的事务支持.这个解决方案应该谨慎使用,并且在任何转换,编排和错误处理逻辑中都应该始终考虑中间件的适用性.                 |

### Governor Limits

由于 Salesforce 平台的多租户性质,在使用 API 时存在一些限制.

| **Solution**                             | **Limits**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SOAP API, REST API, and custom Apex APIs | API请求限制- Salesforce对每个24小时周期内的API调用数设置了限制.该限制基于Salesforce版本类型和许可证数量.例如,无限版每个Salesforce或Lightning平台许可证提供5,000个API请求/ 24小时.<br><br>API查询游标限制-用户同时可以打开最多10个查询游标.否则,最旧的10个游标将被释放.如果远程应用程序尝试打开已释放的查询游标,则会出现错误.例如,如果共享集成用户凭据,则需要考虑最大查询游标数.在可能的情况下,中间件应在执行另一个查询之前完成完整的查询(以串行方式),或者每个应用程序应使用指定的集成用户.或者,中间件可能需要以"轮询"的方式在多个用户之间执行请求.<br><br>调用限制: 有关创建,更新和查询限制,请参考[Data Volumes](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_remote_call_in.htm#data_volumes_header). |
| Bulk API 2.0                             | 请参考<br>[Data Volumes](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_remote_call_in.htm#data_volumes_header)<br>.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Platform Events                          | 事件通知限制: standard volume 平台事件每小时最多可发布10万个事件.high-volume 基于使用量的平台事件每小时最多可发布25万个事件.要监控 high-volume 事件使用情况,请使用 REST API 限制资源.<br><br>事件消息大小限制: 最大事件消息大小为1 MB.使用 Salesforce API 发布事件会计入您的标准 API 限制.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

### Reliable Messaging

可靠消息传递旨在解决保证将消息传递到可能不可靠的远程系统的问题.Salesforce SOAP API和REST API是同步的,本身并不提供任何可靠消息协议的显式支持(例如WS-ReliableMessaging).

我们建议远程系统实现一个可靠的消息传递系统,以确保错误和超时情况得到成功处理.从外部系统发布平台事件依赖于Salesforce API,因此SOAP API和REST API的相同考虑因素同样适用.

### Middleware Capabilities

下表显示了构建此模式的中间件系统的理想特性.

| **Property**                                                                          | **Mandatory** | **Desirable**        | **Not required** |
|---------------------------------------------------------------------------------------|---------------|----------------------|------------------|
| Event handling                                                                        |               | X                    |                  |
| Protocol conversion                                                                   |               | X                    |                  |
| Translation and transformation                                                        |               | X                    |                  |
| Queuing and buffering                                                                 | X             |                      |                  |
| Synchronous transport protocols                                                       | X             |                      |                  |
| Asynchronous transport protocols                                                      |               |                      | X                |
| Mediation routing                                                                     |               | X                    |                  |
| Process choreography and service orchestration                                        |               | X                    |                  |
| Transactionality (encryption, signing, reliable delivery, and transaction management) | X             |                      |                  |
| Routing                                                                               |               |                      | X                |
| Extract, transform, and load                                                          |               | X (for bulk/batches) |                  |

## Example

一家打印耗材和服务公司使用Salesforce作为前端来创建和管理打印机耗材和订单.销售资产记录代表打印机,这些记录会定期从客户现场的打印机管理系统(PMS)中更新打印使用统计数据(墨水状态,纸张水平).一旦达到设定的阈值值(例如,低墨水状态或低于30%的低/空纸张水平),多个对该事件感兴趣的应用程序/进程(可变)会收到通知,发送电子邮件或Chatter警报,并创建一个订单记录.PMS存储Salesforce ID(Salesforce是资产记录主控方).


以下约束条件适用:

- 酒店管理系统(PMS)可以参与基于契约的集成,Salesforce提供契约,而 PMS 作为 Salesforce 服务的客户端(使用企业或合作伙伴WSDL定义).
- 在Salesforce中不应进行自定义开发.

这个例子最好使用 Salesforce的 SOAP API 或 REST API来发布事件,并在 Salesforce 中使用声明性自动化工具.使用平台事件的主要原因是订阅者数量是可变且无限的;然而,对于有限记录列表(如订单)的简单更新,可以使用 SOAP 或 REST API 来更新记录.

在 Salesforce:

- 在 Salesforce 中定义一个平台事件,包含来自 PMS 的通知数据.
- 创建一个由打印机事件通知触发的 Process Builder 流程.该流程更新打印机资产并创建一个订单(使用一个自动启动的流程).
- 下载企业或合作伙伴WSDL,并将其提供给远程系统.

在远程系统:

- 从企业或合作伙伴的 WSDL 创建一个客户存根.
- 使用集成用户的凭据, 通过 OAuth Web服务或 bearer token flow, 对 Salesforce 进行身份验证
- 打印机状态事件发生后,PMS 调用 API 创建打印机状态平台事件(含打印机使用统计数据). Salesforce 事件总线通知 Process Builder 订阅者和所有其他订阅者.

当您使用平台事件时, 事件总线可以在高容量平台事件中回放事件长达72小时.使用中间件解决方案发布这些事件, 可以帮助将错误处理纳入发布方. 但是, 如果需要更高的可靠性,则可以在订阅方实现错误处理.

本例演示了以下内容:

- 实现 Salesforce 的同步API客户端(消费者).
- 将回调发送到 Salesforce 来发布平台事件(与之前介绍的请求/响应平台事件模式相一致).