# 数据可视化

## Context

您使用 Salesforce 来跟踪潜在客户, 管理销售渠道, 创建商机, 并记录将潜在客户转化为客户的订单详细信息. 然而, Salesforce 并不是包含或处理订单的系统. 订单是由外部(远程)系统管理的. 但销售代表希望在不必学习或使用外部系统的情况下, 在 Salesforce 中查看和更新实时订单信息.

## Problem

在 Salesforce 中,您如何查看,搜索和修改存储在 Salesforce 之外的数据,而不把数据从外部系统移到 Salesforce 中?

## Forces

在应用基于这种模式的解决方案时,有一些因素需要考虑:

- 你想在Salesforce中构建一个声明式/点击式的出站集成或UI混合吗?
- 你是否有大量的数据不想复制到你的 Salesforce 组织?
- 你是否需要在任何时间访问少量的远程系统数据?
- 你需要实时获取最新数据吗?
- 您是否将数据存储在云端或后台系统中, 但想在您的 Salesforce 组织中显示或处理这些数据?
- 您是否对将某些类型的数据存储在Salesforce中有担心?

## Solution

下面包含了解决这一集成问题的一些方案.

1. Salesforce Connect(Fit: Best)

    **Solution:**

    使用 Salesforce Connect 可以访问外部数据源, 以及您的 Salesforce 数据. 无需在 Salesforce 中复制数据, 即可实时从遗留系统(如SAP, Microsoft和 Oracle)中提取数据.

    Salesforce Connect 将外部系统中的数据表映射到您的组织中的 External objects. External objects 类似于自定义对象, 但它们映射的是 Salesforce 组织之外的数据. Salesforce Connect 使用与外部数据的实时连接, 始终保持 External objects 的最新状态. 访问 External objects 会实时从外部系统获取数据.

    Salesforce Connect可以让你:

    - 查询外部系统的数据.
    - 在外部系统中创建,更新和删除数据.
    - 通过列表视图,详细页面,记录动态,自定义标签页和页面布局访问外部对象.
    - 定义外部对象与标准或自定义对象之间的关系,以整合来自不同来源的数据.
    - 在外部对象页面上启用 Chatter feeds, 来进行协作.
    - 运行外部数据的report(有限制).
    - 在 Salesforce 移动应用上查看数据.

    要使用 Salesforce Connect 访问存储在外部系统中的数据, 您可以使用以下适配器之一:
    - OData 2.0 适配器或OData 4.0 适配器 - 连接到任何 OData 2.0 或 4.0 生产者公开的数据.
    - 跨组织适配器 — 连接到存储在另一个 Salesforce 组织中的数据.跨组织适配器使用标准的 Lightning Platform REST API. 与 OData 不同, 跨组织适配器直接连接到另一个组织,而无需中间的 Web 服务.
    - 通过 Apex 创建的自定义适配器--如果 OData 和 cross-org 适配器不适合你的需求,可以用 Apex Connector Framework 开发你自己的适配器.

2. Request and Reply(Fit: Suboptimal)

    **Solution:**

    使用 Salesforce web service APIs 来进行临时数据请求,以访问和更新外部系统的数据.此解决方案包括以下几种方法:
  
    使用Salesforce SOAP API. 自定义的 Visualforce 页面或按钮以同步方式发起 Apex SOAP 调用. 在 Salesforce 中, 您可以使用 WSDL 并生成相应的代理Apex类. 该类提供了调用远程服务所需的逻辑. 然后, Visualforce 页面上的用户发起的操作将调用一个Apex控制器, 该动作执行此代理Apex类以执行远程调用.Visualforce页面需要对Salesforce应用程序进行定制.

    使用Salesforce REST API. 自定义 Visualforce 页面或按钮以同步的方式启动 Apex HTTP 调用(REST服务). 在Salesforce中, 你可以使用标准的GET, POST, PUT和DELETE 方法来调用 HTTP 服务. 你可以使用几个HTTP类来与RESTful服务集成. Visualforce页面上的用户发起的动作会调用Apex控制器动作, 执行这些代理Apex类来执行远程调用. Visualforce页面需要对 Salesforce 应用程序进行定制.

    关于这个解决方案的更多信息,请看[Remote Process Invocation—Request and Reply](/ch01-01-request-and-Reply.html).

## Sketch

下图说明了如何使用 Salesforce Connect 从外部系统中使用OData适配器获取数据.

<p align="center">
    <img src="img/Data-virtualization-using-Salesforce-Connect.png?raw=true">
</p>

在此情况下:

- 浏览器执行一个AJAX调用,进而对应的外部对象适配器执行一个操作.
- 适配器将该操作转换为OData请求,并通过集成和服务层向远程系统发送HTTP GET请求.
- 远程系统通过集成和服务层向Salesforce返回一个JSON响应.
- 该响应从OData转换为外部对象,然后呈现回浏览器.

## Results

这种模式相关解决方案的应用允许用户界面发起调用,可以将交易结果显示给最终用户.

### 调用机制

调用机制取决于为实现这一模式而选择的解决方案.

| **Calling Mechanism**                    | **Description**                                                                                                                                                                                                                                                                                                                                                                                                          |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| External Objects                         | Salesforce Connect 将 Salesforce 外部对象映射到外部系统中的数据表. Salesforce Connect 不会将数据复制到您的组织中, 而是按需实时访问数据. 尽管数据不存储在您的组织中, 但 Salesforce Connect 与 Lightning 平台提供了无缝集成. 外部对象也可以使用 Salesforce 功能, 例如全局搜索,查找关系,Record feed 和 Salesforce 移动应用程序. 外部对象也可用于 Apex, SOSL, SOQL查询, Salesforce API以及通过元数据API, 变更集和包进行部署. |
| Lighting Components or Visualforce Pages | 当远程流程作为涉及用户界面的端到端流程的一部分被触发,并且其结果必须在Salesforce记录中显示或更新时使用.例如,一个将信用卡付款提交给外部支付网关并立即返回显示给用户的付款结果的流程.由用户界面事件触发的集成通常需要创建自定义Lightning组件或Visualforce页面.                                                                                                                                                         |


### Error Handling

将错误处理作为整体解决方案的一部分是很重要的. 当错误发生时(异常或错误代码被返回给调用者), 调用者要管理错误处理. [Salesforce Connect Validator](https://appexchange.salesforce.com/appxListingDetail?listingId=a0N3A00000EJHQ6UAP&tab=e) 是一个免费的工具, 可以运行一些常见的查询, 并关注错误类型和失败原因.

### Benefits

使用 Salesforce Connect 解决方案的一些好处包括:

- 这个解决方案不会占用 Salesforce 的数据存储空间.
- 用户不必担心定期在外部系统和 Salesforce 之间同步数据.
- 可以通过 OData,跨组织适配器或使用自定义的 Apex 适配器以最少的代码快速实现声明式设置.
- 用户可以通过外部对象的形式以类似自定义对象的功能访问外部数据.
- 可以使用全局搜索在连接的外部系统中进行联合搜索.
- 可以运行从云端和本地源访问外部数据的报告.请参考下面的考虑事项.

### Salesforce Connect Considerations

使用 Salesforce Connect解决方案需要有以下考量:

- 外部对象的行为与自定义对象相同, 但有些功能对外部对象不可用.欲了解更多信息,请参考 [Salesforce Compatibility Considerations for Salesforce Connect](https://help.salesforce.com/s/articleView?id=platform_connect_considerations_compatibility.htm&language=en_US&type=5).
- 外部对象会影响报告的性能.欲了解更多信息,请参考 [Report Considerations for Salesforce Connect](https://help.salesforce.com/s/articleView?id=platform_connect_considerations_reports.htm&language=en_US&type=5)
- 关于使用Salesforce Connect适配器的其他注意事项, 请参考 [Considerations for Salesforce Connect—All Adapters](https://help.salesforce.com/s/articleView?id=platform_connect_considerations.htm&language=en_US&type=5)
- 如果您正在考虑使用跨 org 的适配器, 请参考 [Considerations for Salesforce Connect—Cross-Org Adapter](https://help.salesforce.com/s/articleView?id=xorg_considerations.htm&language=en_US&type=5)
- 如果您正在考虑使用 OData 适配器, 请参考 [Considerations for Salesforce Connect—OData 2.0 and 4.0 Adapters](https://help.salesforce.com/s/articleView?id=odata_considerations.htm&language=en_US&type=5)
- 如果你考虑使用自定义 Apex 适配器, 请参考 [Considerations for Salesforce Connect—Custom Adapter](https://help.salesforce.com/s/articleView?id=apex_adapter_considerations.htm&language=en_US&type=5)

### Security Considerations

该模式的解决方案应符合 Salesforce org 级安全标准.建议您使用 HTTPS 协议连接到任何远程系统.有关详细信息,请参考 [Security Considerations](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_security_considerations.htm).

如果您正在使用 OData 连接器,请确保您了解 OData 外部数据源上的跨站请求伪造(CSRF)的特殊行为,限制和建议.有关更多信息,请参考 [CSRF Considerations for Salesforce Connect — OData 2.0 and 4.0 Adapters.](https://help.salesforce.com/s/articleView?id=odata_considerations_csrf.htm&language=en_US&type=5)

## Sidebars

### Timeliness

及时性在这种模式中很重要.请记住以下几点:

- 该请求通常是从用户界面调用的,因此处理过程不能让用户等待.
- 根据外部系统的可用性和连接情况,检索外部数据可能需要很长时间.Salesforce 最大超时值为120秒,用于等待外部系统的响应.
- 远程过程的完成应该及时执行,并在 Salesforce 的超时限制和用户的期望范围内完成.

### Data Volumes

这个模式主要用于小的数据容量,实时活动, 因为它的超时值较小, Apex 调用请求或响应的最大超时也有限制. 在数据有效载荷包含在消息中的批处理活动中,不要使用这种模式.

### Endpoint Capability and Standards Support

端点的功能和标准支持取决于您选择的解决方案.

| **Solution**       | **Endpoint Considerations**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Salesforce Connect | OData API - 使用开放数据协议(Open Data Protocol)访问存储在Salesforce外部的数据. 外部数据必须通过OData生产者进行公开.<br><br>其他API — 使用 Apex 连接器框架开发自定义适配器, 当其他可用的适配器不适合您的需求时. 自定义适配器可以从任何源获取数据. 例如: 可以通过 calouts 来获取一些数据,而其他数据可以通过编程方式进行操作甚至生成.<br><br>连接到 Salesforce--使用 Lightning Platform REST API 来访问存储在其他 Salesforce org 中的数据.<br><br>**通过中间件进行连接**<br><br>通过中间件连接--Salesforce Connect 合作伙伴生态系统与 Salesforce 密切合作,确保他们的中间件网关从他们的服务中暴露出 OData 端点,这样 Salesforce 就可以与他们连接而无需编写额外的代码. |
| Request & Reply    | **Apex SOAP Callouts**<br><br>端点必须能够通过 HTTP 接收 Web 服务调用. Salesforce 必须能够通过公共 Internet 访问端点.<br><br>**Apex HTTP Callouts**<br><br>该端点必须能够接收 HTTP 调用. Salesforce 必须能够通过公共互联网访问该端点.<br><br>你可以使用 Apex 的 HTTP Callout, 使用标准的GET, POST, PUT和 DELETE 方法来调用 RESTful 服务.                                                                                                                                                                                                                                                                                                                          |

### State Management

集成系统时,密钥对于持续状态跟踪很重要. 例如: 如果在远程系统中创建了一条记录, 通常该记录需要某种唯一标识符来支持正在进行的更新. 这里有两个选项:

- Salesforce 存储远程记录的主键或唯一标识符.
- 远程系统存储 Salesforce 唯一记录ID或其他唯一的标识符. 在这种同步模式下, 处理集成键需要特定的考虑.

| **Master System** | **Description**                                                                        |
|-------------------|----------------------------------------------------------------------------------------|
| Salesforce        | 远程系统存储 Salesforce 的RecordId或其他一些来自记录的唯一键.                         |
| Remote system     | 对远程进程的调用从应用程序返回唯一的键,Salesforce 将该键值存储在一个唯一的记录字段中. |

### Complex Integrations

在某些情况下, 这种模式所规定的解决方案可能需要实现复杂的集成场景. 这些场景通常通过中间件来解决. 这些场景包括:

- 跨多个系统的调用和调用结果的聚合
- 对传入和传出消息的转换
- 在调用多个系统时保持事务完整性
- 在 Salesforce 和外部系统之间进行其他流程编排活动

### Governing Limits

不同的限制适用于不同的适配器.欲了解更多详情,请参考 [General Limits for Salesforce Connect](https://help.salesforce.com/s/articleView?id=sf.platform_connect_general_limits.htm&type=5)

### Middleware Capabilities

下表强调了参与这种模式的中间件系统的理想属性.

| Property                                                                          | Mandatory | Desirable | Not required |
|-----------------------------------------------------------------------------------|-----------|-----------|--------------|
| Event handling                                                                    |           | X         |              |
| Protocol conversion                                                               | X         |           |              |
| Translation and transformation                                                    | X         |           |              |
| Queuing and buffering                                                             | X         |           |              |
| Synchronous transport protocols                                                   | X         |           |              |
| Asynchronous transport protocols                                                  |           | X         |              |
| Mediation routing                                                                 |           | X         |              |
| Process choreography and service orchestration                                    |           | X         |              |
| Transactionality (encryption, signing, reliable delivery, transaction management) | X         |           |              |
| Routing                                                                           |           |           | X            |
| Extract, transform, and load                                                      |           |           | X            |
| Long polling                                                                      |           |           | X            |


### External Object Relationships

外部对象支持标准的查找关系, 它使用 18 个字符的 Salesforce 记录 ID 来关联相关记录. 然而, 存储在您的 Salesforce 组织之外的数据往往不包含这些记录ID. 因此, 有两种特殊类型的查找关系可用于外部对象:外部查找和间接查找.

这个表格总结了对外部对象可用的关系类型.

| **Relationship** | **Allowed Child Objects**  | **Allowed Parent Objects** | **Parent Field for Matching Records**                            |
|------------------|----------------------------|----------------------------|------------------------------------------------------------------|
| Lookup           | Standard, Custom, External | Standard, Custom           | The 18-character Salesforce record ID                            |
| External lookup  | Standard, Custom, External | External                   | The External ID standard field                                   |
| Indirect lookup  | External                   | Standard, Custom           | Select a custom field with the External ID and Unique attributes |

### High Data Volume Considerations for Salesforce Connect—OData 2.0 and 4.0 Adapters

如果你的组织在访问外部对象时遇到速率限制, 考虑在相关的外部数据源上选择高数据量选项. 这样做可以绕过大多数速率限制, 但也有一些特殊的行为和限制. 欲了解更多信息,请参考 [High Data Volume Considerations for Salesforce Connect](https://help.salesforce.com/s/articleView?id=sf.odata_considerations_high_data_volume.htm&type=5)

### Client-Driven and Server-Driven Paging for Salesforce Connect—OData 2.0 and 4.0 Adapters

外部数据的 Salesforce Connect 查询通常会有一个很大的结果集,该结果集被分成较小的批次或页面. 您决定是由外部系统(服务器驱动)还是由 Salesforce Connect 的 OData 2.0 或 4.0 适配器(客户端驱动)控制分页行为. 外部数据源上的 Server Driven Pagination 字段指定是使用客户端驱动的分页还是服务器驱动的分页. 如果您在外部数据源上启用服务器驱动的分页,Salesforce 将忽略请求的页面大小,包括 500 行的默认 queryMore() 批处理大小. 外部系统返回的页面决定批次,但每页不能超过2000行. 但是, Salesforce Connect 的 OData 适配器的限制仍然适用.

## Example

一家制造公司使用 Salesforce 来管理客户个案. 客户服务代理希望从后台 ERP 系统访问实时订单信息,以 360 度全方位了解客户,而无需学习和手动在 ERP 中运行报告.

实施此模式规定的解决方案, 您应该:

- 使用 OData 端点配置外部数据源. 您的远程应用程序可能包含对 OData 的本机支持. 对于其他应用程序, Dell Boomi, Informatica, Jitterbit, MuleSoft 和 Progress Software 等主要集成供应商已在 Salesforce Connect 上与 Salesforce 合作构建适配器.
- 直接或通过中间件解决方案,将 Salesforce Connect 指向 OData 端点.
- 将您的外部数据库表与 Salesforce 中的外部对象同步. 当用户访问包含来自这些外部对象的数据的页面时, Salesforce Connect 会实时调用您的后端应用程序.