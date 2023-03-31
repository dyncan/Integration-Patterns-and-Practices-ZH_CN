# Pattern Selection Guide

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

> 在 Salesforce 中,**Call-in** 是指外部系统从 Salesforce 中获取数据的过程, 通常是通过调用 Salesforce 的 API 进行数据查询,搜索,更新等操作.
>
> 与 **Call-In** 不同,在 Salesforce 中 **Call-out** 是 Salesforce 主动发送请求获取外部系统数据的过程, 通常是通过调用外部系统的 API 进行数据查询,搜索,更新等操作.

<sup>1</sup> **"Synchronous vs. Asynchronous Communication in Applications Integration"**, MuleSoft, last accessed October 13, 2021, https://www.mulesoft.com/resources/esb/applications-integration.

<sup>2</sup> Ibid.