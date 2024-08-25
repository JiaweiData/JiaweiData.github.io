---
date:
  created: 2024-08-25
  # updated: 2024-08-17
authors:
  - wjw
categories:
  - 工具
tags:
  - dbt
readtime: 15
draft: false
comments: true
---

# dbt v1.8调研（二）主要概念

已经开发了一个[实验性的项目](https://github.com/JiaweiData/dbt-learning)，用来验证功能。

实验的主要特性和功能：

<!-- more -->

## [models](https://docs.getdbt.com/docs/build/models)

transformations(转换逻辑)的核心承载者，基于模板语言（jinja）实现的[sql脚本](https://docs.getdbt.com/docs/build/sql-models)来描述数据转换的逻辑，对于某些数据引擎，也支持[python models](https://docs.getdbt.com/docs/build/python-models) 描述数据逻辑。

支持在model中引用[宏(macro)](https://docs.getdbt.com/docs/build/macros))来封装通用逻辑或函数。

除了sql脚本，还需要在model目录中定义[anyfilename.yml](https://docs.getdbt.com/reference/model-properties)来描述model的元数据，[这里](https://jiaweidata.github.io/dbt-learning/#!/model/model.dbt_learning.stores)有个例子。

model还支持以下模式和策略。

- 全量
- 增量 用于数据总量大，增量相对较小的转换场景
  - [delete+insert](https://docs.getdbt.com/reference/resource-configs/trino-configs#deleteinsert-strategy)
  - [merge](https://docs.getdbt.com/reference/resource-configs/trino-configs#merge-strategy)
  - [append](https://docs.getdbt.com/reference/resource-configs/trino-configs#append-strategy)
  - **[custom_strategy](https://docs.getdbt.com/docs/build/incremental-strategy#custom-strategies)** 自定义策略
- 快照 用于计算拉链表，对于缓慢变化维第二种类型的封装
    - [timestamp](https://docs.getdbt.com/reference/resource-configs/strategy#use-the-timestamp-strategy) 基于增量时间戳字段捕获数据变更
    - [check_cols](https://docs.getdbt.com/reference/resource-configs/strategy#use-the-check_cols-strategy) 基于指定列变化捕获数据变更

官方建议所有model第一次创建都用全量，速度变慢后再考虑改为增量。

### [model groups](https://docs.getdbt.com/docs/collaborate/govern/model-access#groups)

[model groups](https://docs.getdbt.com/docs/build/model-groups) 是对model进行分组，方便管理，比如将多个model放在一个组中，然后统一设置一个配置。

这里引入软件功能的逻辑代码组织方式。model是对数据计算逻辑进行描述的最小单位，那么group就是一组model，组内的model应该是实现某个或者某累需求的集合，由单独的个人或团队负责，对外有统一输入和输出，类似调用接口或者API。通过后面会提到的contract， 可以严格定义对外输出。再结合model access。就可以完成功能的逻辑代码组织和封装，避免内部(private)逻辑对外暴露，可以完成内部重构。

所以在1.8版本，支持了package / group / model 三个粒度由大到小的组织形式。 并且可以分别控制model的[引用权限](https://docs.getdbt.com/docs/collaborate/govern/model-access#access-modifiers)。

### [model contrat](https://docs.getdbt.com/docs/collaborate/govern/model-contracts)

对于model输出数据结构的强制校验，在运行```dbt build```时，会校验model的输出数据结构是否与contract一致。

目前的校验逻辑是，对于一个model contract中定义的输出字段，在model的实际计算结果结构中必须都有，且数据类型也匹配。

## [sources](https://docs.getdbt.com/docs/build/sources)

source与model类似，也是描述数据源的元数据，与model的区别是没有数据转换逻辑，因此没有.sql脚本，只有.yml文件。

主要作用是精确描述数据转换的数据来源。方便数据血缘管理。

## [seeds](https://docs.getdbt.com/docs/build/seeds)

seeds是dbt中用于描述数据源的元数据，与source的区别是，seeds适合的是小数据量，不频繁发生变化的，需要进行版本控制的小量数据来源，通常是csv格式。官方给的一个例子是国家名称与编码表，与model的区别是没有数据转换逻辑，因此没有.sql脚本，只有.yml文件。

注意不要用seed去加载增量数据。

<!-- ## data_tests 单说 -->



## [macros](https://docs.getdbt.com/docs/build/jinja-macros#macros)

macros是dbt中用于封装通用逻辑或函数的模板语言(jinja)脚本，可以定义通用的sql脚本。

## document

这里是指dbt项目通过运行```dbt docs generate```可以生成一个静态的dbt项目文档网页，方便团队成员查看。 这里有一个[例子](https://jiaweidata.github.io/dbt-learning/#!/overview)。

## exposures

这里是dbt中用于描述计算逻辑的下游需求的抽象。例如一个报表等。

cloud 中的[data helth](https://docs.getdbt.com/blog/dbt-explorer#ok-but-is-it-fresh-is-it-right) 也是基于exposures实现的。



<!-- ## package -->

## 其他杂项lint pre-commt version ci

1. lint: sqlfluff
    - [.sqlfluff](https://github.com/JiaweiData/dbt-learning/blob/main/.sqlfluff)
2. pre-commit: pre-commit hook
    - [.pre-commit-config.yaml](https://github.com/JiaweiData/dbt-learning/blob/main/.pre-commit-config.yaml)
3. 约定式提交及版本控制
    - [conventional commit](https://www.conventionalcommits.org/en/v1.0.0/)
    - [semver](https://semver.org/)
    - [commitizen](https://commitizen-tools.github.io/commitizen/)
        - [.cz.toml](https://github.com/JiaweiData/dbt-learning/blob/main/.cz.toml)
4. 自动发布文档，通过[github actions](https://github.com/JiaweiData/dbt-learning/blob/main/.github/workflows/ci.yml)
5. 开发环境相关 tox
    - [tox.ini](https://github.com/JiaweiData/dbt-learning/blob/main/tox.ini)
