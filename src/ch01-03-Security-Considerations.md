# 安全方面的考虑

为了成为企业组合中有效的成员,所有应用都必须与相关的安全机制创建和集成.现代IT策略采用本地和基于云的服务的组合.

虽然将云对云服务整合通常侧重于Web服务和相关授权,但连接本地和云服务经常会引入更高的复杂性.本节描述了安全工具,技术和Salesforce特定考虑事项.

# Reverse Proxy Server

_反向代理是位于 Web 服务器前面的服务器,将客户端(例如Web浏览器)的请求转发给这些 Web 服务器.反向代理通常用于增加安全性,性能和可靠性._ <sup>1</sup>

它是 _一种代理服务器类型, 代表客户端从一个或多个服务器上检索资源. 然后将这些资源返回给客户端, 就好像它们源自代理服务器本身. 与正向代理不同, 正向代理是其相关客户端联系任何服务器的中间人, 而反向代理是其相关服务器被任何客户端联系的中间人._ <sup>2</sup>

在 Salesforce 实施中, 通常通过外部网关产品提供此类服务.例如,可以使用开源选项, 如 Apache HTTP, lighttpd 和 nginix. 商业产品包括 IBM WebSeal 和 Computer Associates SiteMinder. 这些产品可以很容易地被配置为代表内部请求者代理和管理所有出站的 Salesforce 请求.

# Encryption

一些企业需要在内部部署和基于云的应用程序组合之间对选定的交易或数据字段进行加密. 如果您的组织必须遵守额外的合规性要求, 您可以实施替代方案,包括:

- 企业内部的商业加密网关服务,包括 Salesforce自己的,CipherCloud,IBM DataPower,Computer Associates. 对于每个解决方案, 在 HTTP(S) 请求执行之前, 通过发送和接收加密的有效负载或在加密或解密特定数据字段时在事务边界调用加密引擎或网关.
- 基于云的选项,例如Salesforce Shield Platform Encryption.Shield Platform Encryption为您的数据提供了一个全新的安全层,同时保留了关键的平台功能.使用先进的密钥派生系统,您选择的数据在静止状态下进行加密.您可以比以往更安全地保护数据.有关更多信息,请参考 Salesforce online help.

# Specialized WS-* Protocol Support

为了满足安全协议(如WS-*)的要求,我们推荐这些替代方案.

- Security/XML 网关--将 WS-Security 凭证(IBM WebSeal或 Datapower, Layer7, TIBCO等)注入到事务流本身. 这种方法不需要改变应用程序级别的 Web 服务或来自 Salesforce 的 Web 服务调用. 你也可以在整个 Salesforce 安装中重复使用这种方法. 然而, 它需要额外的设计,配置,测试和维护, 以管理适当的 WS-Security 注入到现有的安全网关方法.
- 传输级加密——使用双向 SSL 和 IP 限制加密通信通道. 虽然这种方法本身并不直接实施 WS-* 协议, 但它可以保护本地应用程序和 Salesforce 之间的通信通道, 而无需传递用户名和密码. 它也不需要更改 Salesforce 生成的类. 但是, 可能需要对本地 Web 服务进行一些修改(在应用程序本身或中间件/ESB 层).
- Salesforce 自定义开发—通过 WSDL2Apex 工具将 WS-Security 标头添加到出站 SOAP 请求. 这会从用于调用内部服务的 WSDL 文件生成一个类 Java 的 Apex 类. 虽然此方法不需要更改后端 Web 服务或 DMZ 中的其他组件,但它确实需要:

- 增加构建和测试的工作量
- 在 Apex 代码中手工编写 WS-Security 属性(包括XML序列化)是一个相对复杂和繁琐的过程.
- 更多的长期维护工作

> **Note:**
>
> 最后一种选项不推荐，因为它过于复杂，并且此类集成需要根据Salesforce的定期更新进行定期审查.

<sup>1</sup> _What Is a Reverse Proxy?,_ Cloudflare, last accessed April 11, 2019, [https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/).

<sup>2</sup> _Reverse proxy,_ Wikipedia, last accessed April 11, 2019, [http://en.wikipedia.org/wiki/Reverse_proxy](http://en.wikipedia.org/wiki/Reverse_proxy)


