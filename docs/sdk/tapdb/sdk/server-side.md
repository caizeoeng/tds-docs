---
id: server-side-integration
title: 服务端接入文档
sidebar_position: 6
---

你可以直接使用 REST API 进行接入，可以在不依赖 SDK 的情况下直接将数据上报到 TapDB。

## 1.上报事件和属性

数据传输的格式和含义请参考 [数据规则](/sdk/tapdb/sdk/data-spec#数据规则)。

### 1.1.上报地址

请将数据上报到如下 URL

`https://e.tapdb.net/event`

### 1.2.上报内容

首先请按照数据规则构造 json 字符串，例如

```
{
    "index": "test_appid",
    "device_id": "test_device_id",
    "user_id": "test_user_id",
    "type": "track",
    "name": "#EventName",
    "properties": {
        "os": "Android",             
        "device_id1": "000",             
        "device_id2": "000",             
        "device_id3": "000",          
        "device_id4": "000",     
        "width": 256,                    
        "height": 768,                   
        "device_model": "pixel",         
        "os_version": "Android 10.0",
        "provider": "O2",                
        "network": "1",                  
        "channel": "Google Play",        
        "app_version": "1.0",
        "sdk_version": "2.8.0",
        "#custem_event_property_name": "CustomEventPropertyValue"
    }
}
```

对以上数据去掉空格、换行和回车符进行 urlencode

```
%7B%22index%22%3A%22test_appid%22%2C%22device_id%22%3A%22test_device_id%22%2C%22user_id%22%3A%22test_user_id%22%2C%22type%22%3A%22track%22%2C%22name%22%3A%22%23EventName%22%2C%22properties%22%3A%7B%22os%22%3A%22Android%22%2C%22device_id1%22%3A%22000%22%2C%22device_id2%22%3A%22000%22%2C%22device_id3%22%3A%22000%22%2C%22device_id4%22%3A%22000%22%2C%22width%22%3A256%2C%22height%22%3A768%2C%22device_model%22%3A%22pixel%22%2C%22os_version%22%3A%22Android10.0%22%2C%22provider%22%3A%22O2%22%2C%22network%22%3A%221%22%2C%22channel%22%3A%22GooglePlay%22%2C%22app_version%22%3A%221.0%22%2C%22sdk_version%22%3A%222.8.0%22%2C%22%23custem_event_property_name%22%3A%22CustomEventPropertyValue%22%7D%7D
```

将编码后的数据使用 POST 提交到上报地址。

### 1.3.返回

如果返回 Response Code 为 200，且返回内容为 `1` ，则代表数据上报成功，请在埋点管理中进一步查看事件的写入情况。

### 1.4.常见问题

- 若当前事件的主体并非设备或账号，device_id 和 user_id 可以传入任意一个固定值
- 为了保证服务端上报的事件也能使用设备维度进行分析，建议在客户端调用 SDK 的 `GetDeviceID` 接口取得 SDK 为该设备生产的唯一 ID 并上报到 App 的服务端

## 2.特殊类型信息上报

### 2.1.上报在线人数 {#report-online}

由于SDK无法推送准确的在线数据，这里提供服务端在线数据推送接口。游戏服务端可以每隔5分钟自行统计在线人数，通过接口推送到TapDB。TapDB进行数据汇总展现。

*注意：在线人数使用 json 格式上报，这与其他通用事件上报的格式有差别，请注意区分。*

```
接口：https://se.tapdb.net/tapdb/online
方法：POST
格式：json
必需头信息：Content-Type: application/json
```

请求内容：

| 参数名     | 参数类型   | 参数说明             |
| ------- | ------ | ---------------- |
| appid   | string | 游戏的 APPID        |
| onlines | array  | 多条在线数据（最多 100 条） |

其中 onlines 数组的结构为

| 参数名       | 参数类型   | 参数说明                                 |
| --------- | ------ | ------------------------------------ |
| server    | string | 服务器。TapDB 对同一服务器每一个自然 5 分钟仅接受一次数据    |
| online    | number | 在线人数                                 |
| timestamp | number | 当前统计数据的时间戳(秒)。TapDB 会按照自然 5 分钟进行数据对齐 |

示例：

```
{
"appid":"gkjasd13bbsa1sdk",
"onlines":[{
  "server":"s1",
  "online":123,
  "timestamp":1489739590
},{
  "server":"s2",
  "online":188,
  "timestamp":1489739560
}]
}
```

成功判断：返回的HTTP Code为200时认为发送成功，否则认为失败

### 2.2.上报充值记录

由于 SDK 推送可能会不太准确，这里提供服务端充值推送方法。需要忽略掉 SDK 中的相关充值推送接口。

```
接口：https://e.tapdb.net/event
```

内容（注意后面还需要处理一下）：

```
{
  "module": "GameAnalysis", // 固定参数
  "ip": "8.8.8.8", // 可选。充值用户的IP
  "name": "charge", // 固定参数
  "index": "APPID", // 必需。注意APPID需要被替换成TapDB的appid
  "identify": "userId", // 必需。用户ID。必须和SDK的setUser接口传递的userId一样，并且该用户已经通过SDK接口进行过推送
  "properties": {
      "order_id": "100000", // 可选。长度大于0并小于等于256。订单ID。传递订单ID可进行排重，防止计算多次
      "amount": 100, // 必需。大于0并小于等于100000000000。充值金额。单位分，即无论什么币种，都需要乘以100
      "virtual_currency_amount": 100, //获赠虚拟币数量，必传，可为0
      "currency_type": "CNY", // 可选。货币类型。国际通行三字母表示法，为空时默认CNY。参考：人民币 CNY，美元 USD；欧元 EUR
      "product": "item1", // 可选。长度大于0并小于等于256。商品名称
      "payment": "alipay" // 可选。长度大于0并小于等于256。充值渠道
  }
}
```

假如游戏的 appid 为 abcd1234。构建出 json 字符串后，去掉空格和换行符，然后再进行一次 urlencode。再把结果作为 POST 数据推送
先替换换行符和空格，变成：

`{"module":"GameAnalysis","name":"charge","index":"abcd1234","identify":"user_id","properties":{"order_id":"100000","amount":100,"virtual_currency_amount":100,"currency_type":"CNY","product":"item1","payment":"alipay"}}`

然后 urlencode，变成如下形式。某些版本的 urlencode 可能会把「:」和「,」进行编码，不会影响实际使用。

`%7B%22module%22:%22GameAnalysis%22,%22name%22:%22charge%22,%22index%22:%22abcd1234%22,%22identify%22:%22user_id%22,%22properties%22:%7B%22order_id%22:%22100000%22,%22amount%22:100,%22virtual_currency_amount%22:100,%22currency_type%22:%22CNY%22,%22product%22:%22item1%22,%22payment%22:%22alipay%22%7D%7D`

成功判断：返回的HTTP Code为200时认为发送成功，否则认为失败。