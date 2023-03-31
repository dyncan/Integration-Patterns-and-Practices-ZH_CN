# Purpose and Scope

本文档适用于需要将 Lightning 平台与其企业中的其他应用程序集成的设计师和架构师. 此内容是 Salesforce 架构师和合作伙伴的许多成功实施的精华.

如果您正在考虑大规模采用基于 Salesforce 的应用程序(或 Lightning 平台), 请阅读集成模式的总结和筛选矩阵, 了解可用的集成功能和选项. 在 Salesforce 集成项目的设计和实施阶段, 架构师和开发人员应考虑这些集成模式的细节和最佳实践.

如果项目实施得当, 这些集成模式可以使您尽可能快地投入到生产环境上使用, 并拥有一套稳定,可扩展和免维护的应用程序架构. Salesforce 自己的咨询架构师在架构审查中会使用这些模式作为参考点, 并且也会积极参与维护和改进它们.

与所有模式一样, 这些内容涵盖了大多数集成场景, 但不是全部. 虽然 Salesforce 允许用户界面(UI)集成-- 例如: `Mashups`, 但这种集成超出了本文档的范围. 如果您觉得您的要求超出了这些模式所描述的范围, 请与您的Salesforce代表讨论.

> **Mashups** 是指将 Salesforce 用户界面(UI)与其他 Web 应用程序或服务集成的能力. 这允许用户在Salesforce的 UI 中直接访问外部网站或应用程序的功能和数据, 而无需离开 Salesforce 界面. 例如,您可以通过在 Salesforce UI 中嵌入 `Google Map` 来快速查看客户所在位置, 或者通过在 Salesforce UI 中嵌入 `Twitterfeed` 来跟踪公司的社交媒体活动等. 
>
> 参考官方的文档, 了解更多关于 [Mashups](https://developer.salesforce.com/docs/atlas.en-us.salesforce_large_data_volumes_bp.meta/salesforce_large_data_volumes_bp/ldv_deployments_techniques_using_mashups.htm) 的内容