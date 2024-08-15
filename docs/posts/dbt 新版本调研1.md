---
date:
  created: 2024-08-14
  # updated: 2024-08-11
authors:
  - wjw
categories:
  - 工具
tags:
  - dbt
readtime: 10
draft: false
comments: true
---

# dbt v1.8调研（一）

上次使用 [dbt](https://www.getdbt.com/){target="_blank"} 还是 0.x 版本。当时就觉得这个框架的思路非常好，可以把软件工程的一些概念代入到数据开发中。但是当时版本还有一些低，有一些我生产在用到的功能还无法实现，如先删后插模式，单元测试等。当时只在生产做了小规模使用，没有全面使用。但是我全面推广了 dbt docs 。用于发布我们的数仓的文档，非常好用。

最近看到 dbt 发布了 1.8 大体看了一下文档，发现想要的基本上都有了，准备调研一下，有机会在生产推广。

<!-- more -->

# 计划调研的功能

## core vs cloud

这里对 core 和 cloud 的功能进行一下对比，并看一下有没有替代方案。这里讨论替代的原因并不是为了重复造轮子或者其他原因。

考虑使用cloud的成本是一小部分原因，还有更大的原因是技术选型风险，目前国内要使用国外的 saas 服务，还是有一定技术风险的。这里不展开说了。

| feature      | dbt core                          | dbt cloud| 备注｜
| :--------- | :----------------------------------: | :----------------------------------:|:----|
| dbt Explorer     |   |:material-checkbox-marked-circle:| 应该可以通过工具实现 |
| dbt Mesh (multi-project) support       | | :material-checkbox-marked-circle:| 这个功能也比较有意义，有利于治理复杂项目，官方不支持的话，可能很难做到了 |
| dbt Semantic Layer    | |:material-checkbox-marked-circle:| 这个功能确实有意义，确保指标在使用时语意一致性，之前尝试过没有成功，后面有机会研究一下|
|SQL or Python-based transformations|:material-checkbox-marked-circle:|:material-checkbox-marked-circle:||
|Built-in testing|:material-checkbox-marked-circle:|:material-checkbox-marked-circle:||
|Documentation|:material-checkbox-marked-circle:|:material-checkbox-marked-circle:||
|Command Line Interface (CLI)|:material-checkbox-marked-circle:|:material-checkbox-marked-circle:||
|Browser-based IDE||:material-checkbox-marked-circle:|不重要|
|Low code visual editor||:material-checkbox-marked-circle:|不重要|
|dbt Assist (AI)||:material-checkbox-marked-circle:|应该也不太重要吧|
|Native job orchestration||:material-checkbox-marked-circle:|后面我研究一下 airflow 集成|
|Defer to production||:material-checkbox-marked-circle:|很有启发性的功能，我觉得应该也可以通过 airflow 实现|
|Automated version control||:material-checkbox-marked-circle:|不重要|
|Continuous Integration||:material-checkbox-marked-circle:|很早用药 这个我会单独深入研究，到时候单发一篇文章吧|
|Column-level lineage||:material-checkbox-marked-circle:|这个比较重要，不过有开源方案，后面但说一篇|
|Logging and alerting||:material-checkbox-marked-circle:|考虑通过 airflow 实现|
|Single sign-on (SSO)||:material-checkbox-marked-circle:|不重要|
|Role-based access control (RBAC)||:material-checkbox-marked-circle:|不重要|
|Audit logging||:material-checkbox-marked-circle:|不重要|
|SLA guarantees||:material-checkbox-marked-circle:|这个问题会转化为 airflow 的 SLA|
|Advanced security||:material-checkbox-marked-circle:|不重要|
|Enterprise support and training||:material-checkbox-marked-circle:|自学吧|
|Cloud-native||:material-checkbox-marked-circle:|自己管理|
|Elastic scaling||:material-checkbox-marked-circle:|自己管理|
|Managed dbt version upgrades||:material-checkbox-marked-circle:|这个是双刃剑，我觉得手动更安全，特别是 dbt大版本更新的时候。|

以上多次提到 airflow，我是有计划将 airflow 与 dbt core 进行集成的。这个问题初步调研没有官方支持的方案。也没有很“优雅”的方案。后面我单独调研一下

# 后面计划调研内容

- [ ] 一个实验性项目，可以就用 [jaffle shop](https://github.com/dbt-labs/jaffle-shop-generator){target="_blank"} 做数据源
- [ ] unit test
- [ ] data contract
- [ ] dbt + airflow
- [ ] CI
