---
title: 维度表属性
---


## 1 概述

对于已经上报的事件属性和用户属性，可以通过上传维度表将原先上传的数据映射为另一种展示值或计算值，使初始埋点时的属性值不与展示值相同。

维度表属性相比于虚拟属性，可从系统外部引入原先上传的数据中未包含的信息，而非仅基于系统内数据的逻辑转化。

可在事件属性管理、用户属性管理中「维度表」列，创建某属性的维度表属性。

## 2 适用角色与用途

| 角色                                     | 用途                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| 埋点设计人员（数据产品经理/分析师）      | 避免埋点设计中的过度冗余字段，提高数据模型范式度和埋点设计灵活度 |
| 数据分析人员（数据产品经理/分析师/运营） | 自助引入系统外部数据作为分析维度，满足更多个性化分析需求     |

## 3 创建维度表属性

可在事件属性管理、用户属性管理中「维度表」列，创建该属性的维度表属性。

维度表属性可基于预置属性、自定义属性和虚拟属性创建，但不包括以下类型的虚拟属性：1 基于维度表属性创建的虚拟属性； 2 基于用户属性创建的虚拟事件属性。

![创建维度表属性](/img/customEvent/dimension_table_1.png)

### 3.1 上传入口

在需要添加维度表的基础属性字段上选择「上传」维度表属性。

![上传入口](/img/customEvent/dimension_table_2.png)

### 3.2 文件编码要求 {#encoding}

系统支持 `UTF-8` 或 `UTF-8 with Bom` 编码格式的 `CSV` 文件，文件大小不超过 2G。

可使用 Excel 在保存文档时选择「**CSV UTF-8(逗号分隔符)（.csv）**」格式；

![Excel](/img/customEvent/dimension_table_7.png)

或使用 WPS 在保存文档时选择「**CSV (逗号分隔符)（*.csv）**」格式；

![WPS](/img/customEvent/dimension_table_8.png)

同样可使用 Sublime、NotePad ++ 等文本编辑工具，将文件以 `UTF-8` 或 `UTF-8 with Bom` 编码保存；

![Sublime、NotePad ++ 等](/img/customEvent/dimension_table_9.png)

### 3.3 文件格式要求

首行内容将作为维度表属性的字段名称，需要以英文开头，英文、数字或 `_` 组成；

首列内容将和原始属性进行关联，取值需要和原始属性值对应，并保证值唯一，如遇到重复，则以首条为准，后续重复信息将被抛弃；

维度表属性不超过 10 列，内容会与录入的该属性的数据类型进行适配，规则如下：

| 所选数据类型 | 文件内容                                                     |
| :----------- | :----------------------------------------------------------- |
| 文本         | 任何内容均可                                                 |
| 数值         | 数字均可，0 开头的，把 0 抹掉                                   |
| 时间         | 时间戳，或 yyyy-MM-dd HH:mm:ss.SSS、yyyy-MM-dd HH:mm:ss、yyyy-MM-dd |
| 布尔         | true 或 false                                                |

对于所选数据类型与文件内容类型不一致的行，该行将被完全抛弃，请注意保持数据类型一致。

### 3.4 填写显示名和字段类型

根据需求填写映射后的维度表字段显示名和数据类型。

![填写显示名和字段类型](/img/customEvent/dimension_table_3.png)

### 3.5 解析结果

展示解析总行数、成功行数、错误丢弃行数与错误丢弃原因。

![解析结果](/img/customEvent/dimension_table_4.png)

### 3.6 替换

若解析结果不符合预期，可更新文件后进行替换。

![替换](/img/customEvent/dimension_table_5.png)

## 4 维度表属性的使用

### 4.1 管理维度表属性

可在事件属性管理或用户属性管理页面中，对维度表属性进行管理。

维度表属性折叠在基础属性列中，展开后可进行进一步操作。

![管理维度表属性](/img/customEvent/dimension_table_6.png)

### 4.2 模型中使用的注意事项

维度表属性在使用上和通常的属性一致，根据类型决定其计算逻辑以及筛选条件。

事件属性的维度表属性可以在其基础属性关联的事件中被使用。

用户属性的维度表属性，可用场景等同一般的用户属性。

## 5 最佳实践

### 5.1 提高埋点设计灵活度

采集用户的游戏商城浏览、下单信息时，可只采集商品 ID，商品名称、售价等信息可通过基于「商品 ID」创建维度表属性满足分析需求。

| 商品 ID（基础属性） | （维度表属性） | 售价（维度表属性） |
| :----------------- | :------------- | :----------------- |
| 1                  | 补签券         | 50                 |
| 2                  | 世界频道喇叭   | 10                 |

### 5.2 引入外部信息作为分析维度

系统中采集了用户设备的机型信息，结合通过爬虫收集各机型在电商平台的售价信息，从而得到各类机型的档次，用以对辨别用户价值。

| 机型（基础属性） | 售价（维度表属性） | 用户价值（维度表属性） |
| :--------------- | :----------------- | :--------------------- |
| model_1          | 999                | 低                     |
| model_2          | 1299               | 低                     |
| model_3          | 1999               | 中                     |
| model_4          | 4999               | 高                     |



 