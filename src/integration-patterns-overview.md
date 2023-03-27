- [Integration Patterns Overview](#integration-patterns-overview)
  - [Purpose and Scope](#purpose-and-scope)
  - [Pattern Template](#pattern-template)
    - [Name](#name)
    - [Context](#context)
    - [Problem](#problem)
    - [Forces](#forces)
    - [Solution](#solution)
    - [Sketch](#sketch)
    - [Results](#results)
    - [Sidebars](#sidebars)
    - [Example](#example)
  - [Pattern Summary](#pattern-summary)
    - [List of Patterns](#list-of-patterns)
  - [Pattern Approach](#pattern-approach)
  - [Pattern Selection Guide](#pattern-selection-guide)
  - [Integrating Salesforce with Another System](#integrating-salesforce-with-another-system)
  - [Integrating Another System with Salesforce](#integrating-another-system-with-salesforce)

# Integration Patterns Overview

当您实施 Salesforce 项目时, 您经常需要与其他应用程序集成. 尽管每个集成场景都是独一无二的, 但有一些共同的问题是开发人员必须解决的.

本文档描述了这些常见集成场景的策略(以模式的形式). 每个集成模式都描述了一个特定场景的设计和方法, 而不是具体的实现. 在这个文档中, 你会发现:

- 一些解决关键**原型**集成场景的模式
- 选择矩阵可帮助您确定哪种模式最适合您的场景
- 集成技巧和最佳实践

## Purpose and Scope

本文档适用于需要将 Lightning 平台与其企业中的其他应用程序集成的设计师和架构师. 此内容是 Salesforce 架构师和合作伙伴的许多成功实施的精华.

如果您正在考虑大规模采用基于 Salesforce 的应用程序(或 Lightning 平台), 请阅读集成模式的总结和筛选矩阵, 了解可用的集成功能和选项. 在 Salesforce 集成项目的设计和实施阶段, 架构师和开发人员应考虑这些集成模式的细节和最佳实践.

如果项目实施得当, 这些集成模式可以使您尽可能快地投入到生产环境上使用, 并拥有一套稳定,可扩展和免维护的应用程序架构. Salesforce 自己的咨询架构师在架构审查中会使用这些模式作为参考点, 并且也会积极参与维护和改进它们.

与所有模式一样, 这些内容涵盖了大多数集成场景, 但不是全部. 虽然 Salesforce 允许用户界面(UI)集成-- 例如: `Mashups`, 但这种集成超出了本文档的范围. 如果您觉得您的要求超出了这些模式所描述的范围, 请与您的Salesforce代表讨论.

> **Mashups** 是指将 Salesforce 用户界面(UI)与其他 Web 应用程序或服务集成的能力. 这允许用户在Salesforce的 UI 中直接访问外部网站或应用程序的功能和数据, 而无需离开 Salesforce 界面. 例如,您可以通过在 Salesforce UI 中嵌入 `Google Map` 来快速查看客户所在位置, 或者通过在 Salesforce UI 中嵌入 `Twitterfeed` 来跟踪公司的社交媒体活动等. 
>
> 参考官方的文档, 了解更多关于 [Mashups](https://developer.salesforce.com/docs/atlas.en-us.salesforce_large_data_volumes_bp.meta/salesforce_large_data_volumes_bp/ldv_deployments_techniques_using_mashups.htm) 的内容

## Pattern Template

每个集成模式都遵循一个一致的结构. 这为每个模式所提供的信息提供了一致性, 也使模式的比较更加容易.

### Name

模式标识符,也表示模式中包含的整合类型.

### Context

该模式解决的整体集成场景. 上下文提供了关于用户试图完成什么的信息, 以及应用程序将如何表现以支持这些要求.

### Problem

该模式旨在解决的情景或问题(以问题的形式表达). 在查看每个集成模式时, 请阅读这一部分, 以迅速了解该模式是否适合你的集成场景.

### Forces

使所述场景难以解决的约束和环境.

### Solution

解决集成场景的推荐方法.

### Sketch

一个 UML 序列图,向你展示解决方案如何解决这个场景.

### Results

这一部分解释了如何将解决方案应用于你的集成方案中去, 以及如何解决与该方案相关的细节. 本节还包含应用该模式后可能出现的新挑战.

### Sidebars

与模式相关的其他部分包含关键技术问题,模式的变化,特定于模式的问题等.

### Example

一个端到端的场景, 描述了如何在真实的 Salesforce 场景中使用设计模式. 这一部分的示例解释了集成目标以及如何实施集成模式来实现这些目标.

## Pattern Summary

下面的表格列出了本文档中包含的集成模式.

### List of Patterns

| **Pattern**                                 | **Scenario**                                                                                                                                                                                                                           |
|---------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 远程进程调用-请求和回复 | Salesforce 调用远程系统上的进程,等待该进程完成,然后根据远程系统的响应跟踪状态..                                                                             |
| 远程进程调用——即发即弃   | Salesforce 调用远程系统中的进程,但不等待进程完成. 相反,远程进程接收并确认请求,然后将控制权交还给 Salesforce.                       |
| 批量数据同步化                  | 存储在 Lightning 平台中的数据是根据来自外部系统的更新创建或刷新的,并且当来自 Lightning 平台的更改发送到外部系统时也会同步更新.任何一个方向上的更新都以批处理的方式进行. |
| 远程请求调用                             | 存储在 Lightning 平台中的数据由远程系统创建,检索,更新或删除.                                                                                                                                       |
| 基于数据变化的UI更新            | 由于 Salesforce 数据发生变化,Salesforce 用户界面必须自动更新.                                                                                                                                 |
| 数据可视化                         | Salesforce实时地访问外部数据.这就不需要在Salesforce中持久化数据,然后在 Salesforce 和外部系统之间进行数据协调.                                                            |

## Pattern Approach

本文档中的集成模式分为三类:

- **数据集成**: 这些模式解决了在两个或更多系统中驻留的数据需要进行同步, 以便两个系统始终包含实时性和有意义的数据的要求. 数据集成通常是最简单的集成类型, 但需要适当的信息管理技术来使解决方案可持续且具有成本效益. 这些技术通常包括主数据管理(MDM), 数据治理, 主数据处理, 去重, 数据流设计等方面.
- **流程集成**: 这一类别的模式解决了业务流程利用两个或多个应用程序来完成其任务的需求. 当您实施此类集成的解决方案时, 触发的应用程序必须跨越流程边界调用其他应用程序. 通常, 这些模式也包括协调(其中一个应用程序是中央 "控制器")和编排(其中应用程序是多参与者,没有中央 "控制器"). 这些类型的集成往往需要复杂的设计, 测试和异常处理要求. 此外, 此类复合应用程序通常对底层系统的要求更高, 因为它们通常支持长时间运行的事务, 并具有报告和/或管理进程状态的能力.
- **虚拟集成**: 这个类别的模式解决了用户查看, 搜索和修改存储在外部系统中的数据的需求. 当你实施这种类型的集成的解决方案时, 触发的应用程序必须调用其他应用程序, 并与他们的数据进行实时交互. 这种类型的集成消除了跨系统数据复制的复杂性, 也意味着用户总是与最新的数据进行交互.

> 在这里将 **Virtual Integration** 翻译为 **虚拟集成**. 是因为个人理解虛擬集成应该是通过信息技术和协作工作平台等虚拟手段实现不同组织或企业间的数据交互和协作,而无需在本地复制数据. 在 Salesforce 的上下文中,它描述了一种在 Salesforce 平台上与外部系统进行实时数据交换的方法.

为您的系统选择最佳集成策略并非易事. 有许多方面需要考虑, 也有许多工具可以使用, 其中一些工具比其他工具更适合某些任务. 每个模式都针对特定的关键领域, 包括每个系统的能力, 数据量, 故障处理和事务性.

## Pattern Selection Guide

下面的选择矩阵表列出了模式和它们的一些关键环节, 以帮助你确定哪种模式最适合你的集成要求. 使用下面这些维度对模式进行分类.

<p align="center">
    <img src="https://github.com/dyncan/Integration-Patterns-and-Practices-ZH_CN/blob/main/src/img/pattern-selection-guide.png?raw=true">
</p>

> **Note**
>
> 集成可能需要外部中间件或集成解决方案(例如: _Enterprise Service Bus_), 具体取决于哪些方面适用于您的集成方案.

## Integrating Salesforce with Another System

本表列出了这些模式及其关键环节, 以帮助你确定当你的集成方案从 Salesforce 到另一个系统时, 哪个集成模式最适合你的要求.

| **Type** | **Timing** | **Key Pattern to Consider** |
|----------|------------|-----------------------------|
| 流程集成 | 同步       | 远程进程调用-请求和回复     |
| 流程集成 | 异步       | 远程进程调用——即发即弃      |
| 数据集成 | 同步       | 远程进程调用——请求和回复     |
| 数据集成 | 异步       | 基于数据变化的 UI 更新      |
| 虚拟集成 | 同步       | 数据可视化                 |

## Integrating Another System with Salesforce

本表列出了这些模式及其关键方面,以帮助你在你的集成方案从另一个系统到 Salesforce 时确定最适合你的要求的模式.

| **Type**            | **Timing**   | **Key Pattern to Consider** |
|---------------------|--------------|-----------------------------|
| 流程集成 | 同步  | 远程呼入              |
| 流程集成 | 异步 | 远程呼入              |
| 数据集成    | 同步  | 远程呼入              |
| 数据集成    | 异步 | 批量数据同步  |

> 在 Salesforce 中，**Call-in** 是指外部系统从 Salesforce 中获取数据的过程, 通常是通过调用 Salesforce 的 API 进行数据查询、搜索、更新等操作.
>
> 与 **Call-In** 不同，在 Salesforce 中 **Call-out** 是 Salesforce 主动发送请求获取外部系统数据的过程, 通常是通过调用外部系统的 API 进行数据查询、搜索、更新等操作.

<sup>1</sup> **"Synchronous vs. Asynchronous Communication in Applications Integration"**, MuleSoft, last accessed October 13, 2021, https://www.mulesoft.com/resources/esb/applications-integration.

<sup>2</sup> Ibid.