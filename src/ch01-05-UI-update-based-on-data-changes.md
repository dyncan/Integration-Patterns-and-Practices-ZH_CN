# 基于数据变化的UI更新

## Context

你使用 Salesforce 来管理客户的案例. 一位客户服务代表正在与一位客户通电话, 处理一个案件. 客户进行了订单支付, 客户服务代表需要在 Salesforce 中看到来自支付处理应用程序的实时更新, 表明客户已经成功地支付了订单的未付金额.

## Problem

在 Salesforce 中发生事件时, 用户如何在 Salesforce 用户界面中得到通知, 而无需刷新屏幕导致有可能丢失的工作内容?

## Forces

在应用基于这种模式的解决方案时, 有一些因素需要考虑:

- 被操作的数据是否需要存储在 Salesforce 中?
- 是否可以建立一个自定义的用户界面来查看这些数据?
- 用户是否有调用自定义用户界面的权限?

## Solution

解决这个集成问题的推荐方案是使用Salesforce Streaming API. 该方案由以下部分组成:

- 一个带有查询定义的PushTopic, 允许你:
  - 指定哪些事件触发更新.
  - 选择在通知中包括哪些数据.
- 基于 JavaScript 的 Bayeux 协议实现(目前为 CometD ), 可供用户界面使用.
- 一个 Visualforce 页面或 Lightning 组件
- 一个作为静态资源的 JavaScript 库

## Sketch

下图说明了如何通过实现 Streaming API 将通知传输到 Salesforce 用户界面. 这些通知是由 Salesforce 中的记录更改触发的.

<p align="center">
    <img src="img/UI-Update-in-Salesforce-Triggered-by-a-Data-Change.png?raw=true">
</p>

UI Update in Salesforce Triggered by a Data Change


## Results

### Benefits

与此模式相关的解决方案的应用具有以下好处:

- 消除了编写自定义轮询机制的需求.
- 消除了用户发起反馈循环的需求

### Unsupported Requirements

该解决方案有以下局限性:

- 通知的传递不能保证.
- 通知的顺序不能保证.
- 从 Bulk API 所做的记录更改不会生成通知.

### Security Considerations

遵守标准的 Salesforce 组织级安全性.建议您使用 HTTPS 协议连接到 Streaming API.请参考[ Security Considerations ](https://developer.salesforce.com/docs/atlas.en-us.integration_patterns_and_practices.meta/integration_patterns_and_practices/integ_pat_security_considerations.htm).

## Sidebars

最佳的解决方案是在 Salesforce 中创建一个自定义的用户界面. 当务之急是, 你要考虑到一个适当的用户界面容器, 可以用来渲染自定义用户界面. 支持的浏览器在[ Streaming API Developer Guide ](https://developer.salesforce.com/docs/atlas.en-us.244.0.api_streaming.meta/api_streaming/intro_stream.htm)中列出.

## Example

一家电信公司使用Salesforce来管理客户案例. 客户服务经理希望在他们的一个客户服务代表成功关闭一个案例时得到通知.

按照这种模式所规定的解决方案,客户应该:

- 创建一个 PushTopic, 当一个案件被保存为状态为 "关闭" 时, 发送一个通知.
- 创建一个供客户服务经理使用的自定义用户界面. 这个用户界面订阅了 PushTopic 频道.
- 在自定义用户界面中实现逻辑, 显示由该经理的客户服务代表生成的警报.

