# 批量数据同步

## Context

如果您正在将您当前的 CRM 系统迁移到 Salesforce, 并希望实现以下目标: 

- 从当前 CRM 系统中提取和转换客户账户,联系人和商机,并将数据加载到 Salesforce 中(初始数据导入)
- 每周从远程系统中提取,转换和加载客户的账单数据到 Salesforce 中(持续进行).
- 每周从 Salesforce 中提取客户活动信息并将其导入到本地数据仓库中(持续性任务).

## Problem

在考虑到这些导入/导出的动作可能会在工作时间干扰到最终用户的操作, 有些时候可能会涉及大量数据操作, 那么您如何将这些数据导入 Salesforce 或从 Salesforce 导出?

## Forces

在应用基于此模式的解决方案时, 有许多要考虑的因素:

- 数据是否应该存储在 Salesforce 中?如果不是,架构师可以考虑其他的集成选项(例如 mashups).
- 如果数据应存储在Salesforce中,那么是否应根据远程系统中的事件刷新数据?
- 数据是否应定期刷新?
- 这些数据是否支持主要的业务流程?
- 这些数据在 Salesforce 中的可用性是否对分析(报告)需求产生影响?

## Solution

下表包含了解决这些集成问题的各种方案.

|             **Solution**             |   **Fit**  | **Data master** |                                                                                                                                                                                                                                  **Comments**                                                                                                                                                                                                                                  |
|:------------------------------------:|:----------:|:---------------:|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| Salesforce Change Data Capture       | Best       | Salesforce      | Salesforce 的 Change Data Capture(CDC) 发布变更事件, 这些事件代表对 Salesforce 记录的变更操作. 这些更改包括创建新记录, 更新现有记录, 删除记录和撤销删除的记录. <br><br>通过 Change Data Capture, 您可以接收 Salesforce 记录的几乎实时的更改, 并同步外部数据存储中相应的记录. <br><br>Change Data Capture(CDC)负责复制过程中的持续同步部分. 它会发布 Salesforce 数据的增量, 包括新记录和更改记录. Change Data Capture 需要一个集成应用程序来接收事件并在外部系统中执行更新操作. |
| Replication via third-party ETL tool | Best       | Remote system   | 利用第三方ETL工具,对源数据执行增量数据的捕获. <br><br>ETL 工具会根据源数据的变更做出反应,转换数据,然后调用 Salesforce Bulk API 执行 DML 语句.也可以使用 Salesforce SOAP API 来实现.                                                                                                                                                                                                                                                                                      |
| Replication via third-party ETL tool | Good       | Salesforce      | 利用第三方ETL工具,允许你针对 ERP 和 Salesforce数据集运行 Change Data Capture. <br><br>在这个解决方案中,Salesforce 是数据源,你可以使用单个行的时间/状态信息来查询数据并过滤目标结果集.可以通过使用 SOQL 和 SOAP API 以及 query() 方法来实现,或者通过使用 SOAP API 和 getUpdated() 方法.                                                                                                                                                                                  |
| Remote call-in                       | Suboptimal | Remote system   | 远程系统可以通过使用其中一种 API 调用 Salesforce,并在数据更新时执行更新操作.然而,这会在两个系统之间产生相当大量的持续性的流量. <br><br>应该更加注重错误处理和数据锁定.这种模式有可能导致不断的数据更新操作,从而可能影响终端用户的性能.                                                                                                                                                                                                                                  |
| Remote process invocation            | Suboptimal | Salesforce      | Salesforce 有可能调用远程系统并在数据发生更改时执行更新操作.然而,这会在两个系统之间产生相当大量的持续性的流量. <br><br>应该更加注重错误处理和数据锁定.这种模式有可能导致不断的数据更新操作,从而可能影响终端用户的性能.                                                                                                                                                                                                                                                   |

## Sketch

下图说明了这种模式下的事件顺序,其中远程系统是数据主站.

<p align="center">
    <img src="img/Batch-Data-Syncrhonization-Pattern-Remote_Master.png?raw=true">
</p>

下图说明了这种模式下的事件顺序,其中Salesforce是数据主站.

<p align="center">
    <img src="img/Batch-Data-Syncrhonization-Pattern-Salesforce_Master.png?raw=true">
</p>

## Results

在以下情况下, 你可以将来自外部系统的数据与 Salesforce 集成:

- 外部系统是数据主站, Salesforce 是由单个源系统或多个系统提供的数据的使用者.在这种情况下,通常会有一个数据仓库(Data Warehouse)或数据集市(Data Mart),将数据汇总后再导入到 Salesforce 中.
- Salesforce是数据主站: Salesforce 是某些实体的记录系统,Salesforce Change Data Capture 应用程序可以获知 Salesforce 数据的变更情况.

在一个典型的Salesforce集成方案中, 实施团队会做以下工作之一:

- 在源数据集上实施 Change Data Capture.
- 在一个中间的本地数据库中实现一组支持性的数据库结构,称为控制表.

然后, ETL工具被用来创建程序,这些程序将:

- 读取控制表以确定作业的最后运行时间,并提取所需的任何其他控制值.
- 使用上述控制值作为过滤器,查询源数据集.
- 应用预定义的处理规则,包括验证,增强等.
- 使用 ETL 工具的可用连接器/转换功能来创建目标数据集.
- 将数据集写入 Salesforce 对象.
- 如果数据处理成功,更新控制表中的控制值.
- 如果处理失败, 用能够重新启动和退出的值更新控制表.

> **Note:**
>
> 我们建议您在一个 ETL 工具可以访问的环境中创建控制表和相关的数据结构,即使无法访问 Salesforce.这样可以提供足够的弹性.在这个过程中,Salesforce 应该被视为一个分支,而 ETL 基础设施则是中心枢纽.

为了使 ETL 工具充分发挥数据同步功能的最大优势，请考虑以下几点:

- 将 ETL 作业进行链式和顺序处理，以提供一个连贯的过程.
- 使用两个系统的主键来匹配输入的数据.
- 使用特定的 API 方法，只获取最新的数据.
- 如果在主-从或查找关系中导入子记录，应在数据源处使用其父键对导入的数据进行分组，以避免数据锁定。例如，如果您正在导入联系人数据，请确保按父帐户 ID 对联系人数据进行分组，以便可以在一个 API 调用中加载单个帐户的最大联系人数。未能对导入的数据进行分组通常会导致第一个联系人记录被加载，而该帐户的后续联系人记录在 API 调用的上下文中失败.
- 任何导入后的数据处理，如触发器，都应该只有选择性地处理数据.
- 如果你的集成场景涉及到大数据量，请遵循[《大数据量部署的最佳实践》](https://trailhead.salesforce.com/content/learn/modules/large-data-volumes)中的最佳实践。

### Error Handling and Recovery

必须考虑将错误处理和恢复策略作为整体解决方案的一部分。最好的方法取决于你选择的解决方案.

| **Error location**                                | **Error handling and recovery strategy**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|---------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Read from Salesforce using Change Data Capture    | 错误处理: 错误处理必须在远程服务中进行，因为事件实际上被交给远程系统进行进一步处理。由于这种模式是异步的，远程系统处理消息排队、错误等。此外，Change Data Capture 事件不在数据库事务中进行处理。因此，发布的事件无法在事务中回滚。<br><br>恢复: 因为这个模式是异步的，远程系统必须根据服务的服务质量要求发起重试。与每个 Change Data Capture 事件相关联的 Reply ID 是原子性的，并且随着每个发布的事件增加而增加。该 ID 可用于从特定事件中重放⌚️流（例如，基于最后成功捕获的事件）。高容量平台事件消息将存储72小时（3天）。当使用 CometD 客户端订阅频道时，您可以检索过去的事件消息。 |
| Read from Salesforce using a 3rd party ETL system | 错误处理: 如果在读取操作期间发生错误，请对非基础设施相关的错误实现重试。如果反复失败，应在 ETL 操作的上下文中使用控制表/错误表来实现标准处理: 1). Log the error 2). Retry the read operation 3). Terminate if unsuccessful 4). Send a notification<br><br>恢复: 重新启动 ETL 进程，从失败的读操作中恢复.<br><br>如果操作成功，但有失败的记录，立即重启或随后执行 job 应该能解决这个问题。在这种情况下，延迟重启可能是一个更好的解决方案，因为它允许有时间来分流和纠正可能导致错误的数据。                                                                                           |
| Write to Salesforce                               | 错误处理: 在写操作中发生的错误可能是由应用程序中的各种因素造成的。API 调用会返回一个由以下信息组成的结果集。这些信息应被用来重试写操作（如有必要的话）: 1). Record identifying information 2). Success/failure notification 3). A collection of errors for each record<br><br>恢复: 重新启动 ETL 进程，从失败的读操作中恢复.<br><br>如果操作成功，但有失败的记录，立即重启或随后执行 job 应该能解决这个问题。在这种情况下，延迟重启可能是一个更好的解决方案，因为它允许有时间来分流和纠正可能导致错误的数据。                                                                       |
| External master system                            | 应按照主系统的最佳做法来处理错误.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |


### Security Considerations

任何对远程系统的调用必须保持请求的保密性、完整性和可用性。不同的安全考虑因素，取决于你选择的解决方案.

- 需要具有至少 "API Only" 用户权限的 Lightning 平台许可证，才能允许对 Salesforce API 进行经过身份验证的API访问。
- 我们建议使用标准的加密方式来保证密码访问的安全.
- 在调用 Salesforce API 时，请使用 HTTPS 协议。如果有必要，您还可以通过本地安全解决方案代理流量到 Salesforce API.

查看 [Security Considerations](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_security_considerations.htm).

## Sidebars

### Timeliness

及时性在这种模式下并不是非常重要的。然而，必须注意接口设计，以便所有批处理过程在指定的批处理事务内完成。

与所有批处理操作一样，我们强烈建议您在批处理窗口期间注意隔离源系统和目标系统。在工作时间加载批次可能会导致某些数据竞争，从而导致用户的更新失败，或者更重要的是，批量加载（或部分批量加载）失败。

对于有全球业务的组织来说，在同一时间运行所有的批处理程序可能是不可行的，因为系统可能一直在使用中。在这些情况下，可以使用记录类型和其他过滤标准的数据分割技术来避免数据竞争。

### State Management

您可以通过在两个系统之间使用代理键(surrogate keys)来实现状态管理。如果您需要跨 Salesforce 实体进行任何类型的事务管理，则建议您使用Apex的远程调用模式(Remote Call-In pattern)。

标准的乐观记录锁在平台上执行，并且使用 API 进行的任何更新都需要正在编辑记录的用户刷新记录并启动他们的事务。在 Salesforce API 的上下文中，乐观锁是指一种流程，其中:

- Salesforce 并不维护特定用户正在编辑的记录的状态.
- 在读取时，它记录了数据被提取的时间.
- 如果用户更新了记录并保存了它，Salesforce 会检查是否有其他用户在这期间更新了该记录.
- 如果记录已被更新，系统会通知用户已进行了更新，用户应在继续进行更新前检索该记录的最新版本。

### Middleware Capabilities

用来实现这种模式的最有效的外部技术是传统的 ETL 工具。重要的是，所选择的中间件工具要支持 Salesforce Bulk API。

中间件工具支持 getUpdated() 函数是有帮助的，但不是关键。这个函数提供了最接近 Salesforce 平台上标准变更数据捕获能力的实现.

下表强调了参与这种模式的中间件系统的理想属性:

| **Property**                                                                      | **Mandatory**                                   | **Desirable** | **Not required** |
|-----------------------------------------------------------------------------------|-------------------------------------------------|---------------|------------------|
| Event handling                                                                    |                                                 | X             |                  |
| Protocol conversion                                                               |                                                 | X             |                  |
| Translation and transformation                                                    | X                                               | X             |                  |
| Queuing and buffering                                                             | X                                               |               |                  |
| Synchronous transport protocols                                                   |                                                 |               | X                |
| Asynchronous transport protocols                                                  | X                                               |               |                  |
| Mediation routing                                                                 |                                                 | X             |                  |
| Process choreography and service orchestration                                    | X                                               |               |                  |
| Transactionality (encryption, signing, reliable delivery, transaction management) |                                                 | X             |                  |
| Routing                                                                           |                                                 | X             |                  |
| Extract, transform, and load                                                      | X                                               |               |                  |
| Long Polling                                                                      | X (required for Salesforce Change Data Capture) |               |                  |


## Example

一个公用事业公司使用基于主机的批处理程序，将潜在客户分配给个别销售代表和团队。这些信息需要每天晚上导入Salesforce。

客户已经决定使用一个商业上可用的 ETL 工具对源表实施变更数据采集。

该解决方案的工作原理如下:

- 一个类似于cron的调度程序执行一个批处理作业，将潜在客户分配给用户和团队.
- 批处理作业运行并更新数据后，ETL工具使用 change data capture 来识别这些更改。ETL 工具对来自数据存储的变化进行整理。
- ETL 连接器使用 Salesforce SOAP API 将变化加载到 Salesforce。