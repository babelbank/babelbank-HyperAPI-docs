# Babel RESTFul API说明

------

# 介绍

通过了解以下信息，您可以方便的使用 Babel 提供的 API 来接入 Babel 交易平台。


# 

# API 签名认证

Babel 使用 API key 和 API secret 进行验证，请联系Babel开通 API key 和 API secret。

Babel 的 API 请求，除公开的 API 外都需要携带 API key 以及签名

签名前准备的数据如下：

`HTTP_METHOD` + `HTTP_REQUEST_URI` + `TIMESTAMP` + `POST_BODY`

连接完成后，进行 `Base64` 编码，对编码后的数据进行 `HMAC-SHA1` 签名，并对签名进行二次 `Base64` 编码，各部分解释如下：

<aside class="warning">
请注意需要进行两次 `Base64` 编码！
</aside>

### HTTP_METHOD

`GET`, `POST`, `DELETE`, `PUT` 需要大写

### HTTP_REQUEST_URI

`https://api.babelbank.io/v1/` 为 v1 API 的请求前缀版本号

后面再加上真正要访问的资源路径，如 `orders?param1=value1`，最终即 `https://api.babelbank.io/v1/orders?param1=value1`

对于请求的 URI 中的参数，需要按照按照字母表排序！

即如果请求的 URI 为 `https://api.babelbank.io/v1/orders?c=value1&b=value2&a=value3`，则进行签名时，应先将请求参数按照字母表排序，最终进行签名的 URI 为 `https://api.babelbank.io/v1/orders?a=value3&b=value2&c=value1`，
请注意，原请求 URI 中的三个参数顺序为 `c, b, a`，排序后为 `a, b, c`。

### TIMESTAMP

访问 API 时的 UNIX EPOCH 时间戳，需要和服务器之间的时间差少于 30 秒

### POST_BODY

如果是 `POST` 请求，`POST` 请求数据也需要被签名，签名规则如下：

所有请求的 key 按照字母顺序排序，然后进行 url 参数化，并使用 `&` 连接。

<aside class="warning">
请注意 POST_BODY 的键值需要按照字母表排序！
</aside>

> 如果请求数据为：

```json
{
  "username": "username",
  "password": "passowrd"
}
```

> 则先将 key 按照字母排序，然后进行 url 参数化，即：

```
password=password
username=username
```

> 因为 `p` 在字母表中的排序在 `u` 之前，所以 `password` 要放在 `username` 之前，然后使用 `&` 进行连接，即：

```
password=password&username=username
```

## 完整示例

> 对于如下的请求：

```
POST https://api.babelbank.io/v1/orders

{
  "type": "limit",
  "side": "buy",
  "amount": "100.0",
  "price": "100.0",
  "symbol": "btcusdt"
}

timestamp: 1523069544359
```

> 签名前的准备数据如下：

```
POSThttps://api.babelbank.io/v1/orders1523069544359amount=100.0&price=100.0&side=buy&symbol=btcusdt&type=limit
```

> 进行 Base64 编码，得到：

```
UE9TVGh0dHBzOi8vYXBpLmZjb2luLmNvbS92Mi9vcmRlcnMxNTIzMDY5NTQ0MzU5YW1vdW50PTEwMC4wJnByaWNlPTEwMC4wJnNpZGU9YnV5JnN5bWJvbD1idGN1c2R0JnR5cGU9bGltaXQ=
```

> 拷贝在申请 API Key 时获得的秘钥（API SECRET），下面的签名结果采用 `3600d0a74aa3410fb3b1996cca2419c8` 作为示例，

> 对得到的结果使用秘钥进行 `HMAC-SHA1` 签名，并对二进制结果进行 `Base64` 编码，得到：

```
DeP6oftldIrys06uq3B7Lkh3a0U=
```

> 即生成了用于向 API 服务器进行验证的最终签名

## 参数名称

* `BB-ACCESS-KEY`
* `BB-ACCESS-SIGNATURE`
* `BB-ACCESS-TIMESTAMP`

>参数放在Header中

## 说明

可以使用[开发者工具]()（暂未开放）进行在线联调测试。


### API 接口列表

# 查询可用币种 


### HTTP Request

`GET https://api.babelbank.io/v1/loan/currencies`


> 响应结果如下：

```json
{
  "status": 0,
  "data": [
    "BTC",
    "USDT"
  ]
}
```


# 查询可用交易对
此 API 用于获取可用交易对。

### 可用交易对模型说明

可用交易对模型由以下属性构成：

属性 | 类型 | 含义解释
---------- | ------- | -------
`name` | `String` | 交易对
`base_currency` | `String` | 基准币种
`quote_currency` | `String` | 计价货币
`market_price` | `String` | 市场价 * 10的8次方
`avg_price` | `String` | 7日均价 * 10的8次方/借款结算价格
`pledge_rate` | `String` | 借款基准质押率 * 10的8次方，百分比


### HTTP Request

`GET https://api.babelbank.io/v1/loan/symbols`

> 响应结果如下：

```json
{
  "status": 0,
  "data": [
    {
      "name": "BTC|USDT",
      "base_currency": "BTC",
      "quote_currency": "USDT",
      "market_price": "746057000000",
      "avg_price":"738399000000",
      "pledge_rate":"1000000"
    },
    {
      "name": "ETH|USDT",
      "base_currency": "ETH",
      "quote_currency": "USDT",
      "market_price": "746057000000",
      "avg_price":"738399000000",
      "pledge_rate":"1000000"
    }
  ]
}
```



# 查询账户资产
此 API 用于查询用户的资产列表。
### 用户模型说明

用户模型由以下属性构成：

属性 | 类型 | 含义解释
---------- | ------- | -------
`user_id` | `String` | 用户id
`currency` | `String` | 币种
`available` | `String` | 可用余额 * 10的8次方
`frozen` | `String` | 冻结余额 * 10的8次方
`balance` | `String` | 账户余额 * 10的8次方
`withdrawal_addr` | `String` | 提币钱包地址
`deposit_addr` | `String` | 充币地址

### HTTP Request

`GET https://api.babelbank.io/v1/accounts/{user_id}/balance`

### URL 参数

参数 | 描述
--------- | -----------
user_id | 用户 ID

> 响应结果如下：
>
> >用户不存在 status：1001，调用创建用户接口
```json
{
  "status": 0,
  "data": [
        {
            "currency": "BTC",
            "available": "157789000",
            "frozen": "22571356",
            "balance": "135217644",
            "withdrawal_addr": "mn4V3HzqWBAG6fKqjeEHSdik964mTCPBjf",
            "deposit_addr": "2Mtm7aK7q3cya8yDDN2A41fuCkKSE6xGUYG"
        },
        {
            "currency": "USDT",
            "available": "157789000",
            "frozen": "22571356",
            "balance": "135217644",
            "withdrawal_addr": "mn4V3HzqWBAG6fKqjeEHSdik964mTCPBjf",
            "deposit_addr": "2Mtm7aK7q3cya8yDDN2A41fuCkKSE6xGUYG"
        }
     ]
}
```
# 创建新用户（新增）

### HTTP Request

`POST https://api.babelbank.io/v1/accounts`

### 请求参数

参数 | 默认值 | 描述
--------- | ------- | -----------
`user_id` | `String` | 渠道用户唯一标识


> 响应结果如下：

```json
{
    "status": 0,
    "user_id": "9d17a03b852e48c0b3920c7412867623"
}
```


# 获取用户账户交易对列表

### 用户账户交易模型说明

模型由以下属性构成：

属性 | 类型 | 含义解释
---------- | ------- | -------
`user_id` | `String` | 用户id
`symbol` | `String` | 交易对
`pledge_currency` | `String` | 质押物币种
`pledge_value` | `String` | 质押物数量 * 10的8次方
`borrow_currency` | `String` | 借出币种
`borrow_value` | `String` | 借出币种数量 * 10的8次方
`pledge_rate` | `String` | 质押率 * 10的8次方

### HTTP Request

`GET https://api.babelbank.io/v1/accounts/{user_id}/symbols`


> 响应结果如下：

```json
{
  "status": 0,
  "data": [
    {
            "symbol": "BTC|USDT"
            "pledge_currency": "BTC",
            "borrow_currency": "USDT",
            "pledge_value": "22571356",
            "borrow_value": "100000000000",
            "pledge_rate": "16001020"
    },
    {
            "symbol": "USDT|BTC"
            "pledge_currency": "USDT",
            "borrow_currency": "BTC",
            "pledge_value": "100000000000",
            "borrow_value": "22571356",
            "pledge_rate": "16001020"
    }
  ]
}
```

# 查询用户指定交易对数据

### HTTP Request

`GET https://api.babelbank.io/v1/accounts/{user_id}/symbols/{symbol}`


> 响应结果如下：

```json
{
  "status": 0,
  "data": {
            "symbol": "BTC|USDT"
            "pledge_currency": "BTC",
            "borrow_currency": "USDT",
            "pledge_value": "22571356",
            "borrow_value": "100000000000",
            "pledge_rate": "16001020"
    }
}
```

# 试算用户交易后风险值

### 用户交易对模型说明

模型由以下属性构成：

属性 | 类型 | 含义解释
---------- | ------- | -------
`user_id` | `String` | 用户id
`symbol` | `String` | 交易对
`side` | `String` | 交易方向（`loan借款`、`repayment还款`、`supplementary_pledge补充`）
`currency` | `String` | 币种
`value` | `String` | 数量 * 10的8次方
`pledge_rate` | `String` | 质押率 * 10的8次方，也叫结算后风险值
`avg_price` | `String` | 7日均价 * 10的8次方/借款结算价格

### HTTP Request

`GET https://api.babelbank.io/v1/accounts/risk`

### 请求参数

参数 | 默认值 | 描述
--------- | ------- | -----------
`user_id` | `String` | 用户id
`symbol` | `String` | 交易对
`side` | `String` | 交易方向（`loan借款`、`repayment还款`、`supplementary_pledge补充`)
`currency` | `String` | 币种
`value` | `String` | 数量 * 10的8次方


> 响应结果如下：

```json
{
  "status": 0,
  "data": [
    {
            "symbol": "BTC|USDT"
            "pledge_currency": "BTC",
            "borrow_currency": "USDT",
            "pledge_value": "22571356",
            "borrow_value": "100000000000",
            "pledge_rate": "",
            "avg_price":"738399000000"
    },
    {
            "symbol": "USDT|BTC"
            "pledge_currency": "USDT",
            "borrow_currency": "BTC",
            "pledge_value": "100000000000",
            "borrow_value": "22571356",
            "pledge_rate": "",
            "avg_price":"738399000000"
    }
  ]
}
```



# 订单

## 订单模型说明

订单模型由以下属性构成：

属性 | 类型 | 含义解释
---------- | ------- | -------
`order_id` | `String` | 订单 ID
`symbol` | `String` | 交易对
`side` | `String` | 交易方向（`loan借款/supplementary_pledge补充`、`repayment还款`）
`pledge_currency` | `String` | 质押币种
`pledge_value` | `String` | 质押数量 * 10的8次方
`borrow_currency` | `String` | 借出币种
`borrow_value` | `String` | 借出数量 * 10的8次方
`state` | `String` | 订单状态
`fill_fees` | `String` | 手续费usdt * 10的8次方
`created_at` | `Long` | 创建时间
`updated_at` | `Long` | 更新时间

订单状态说明：

属性  | 含义解释
----------- | -------
`submitted` | 已提交
`pending` | 处理中
`successful` | 成功
`failure` | 失败

## 创建新的订单

### HTTP Request

`POST https://api.babelbank.io/v1/loan/orders`

### 请求参数

参数 | 默认值 | 描述
--------- | ------- | -----------
`user_id` | `String` | 用户id
`symbol` | `String` | 交易对 
`side` | `String` | 交易方向（`loan借款/supplementary_pledge补充`、`repayment还款`）
`pledge_currency` | `String` | 质押币种
`pledge_value` | `String` | 质押数量 * 10的8次方
`borrow_currency` | `String` | 借出币种
`borrow_value` | `String` | 借出数量 * 10的8次方

> 响应结果如下：

```json
{
    "status": 0,
    "data": {
        "order_id": "9d17a03b852e48c0b3920c7412867623",
        "symbol": "BTC|USDT",
        "side": "loan",
        "pledge_currency": "BTC",
        "pledge_value": "100000000",
        "borrow_currency": "USDT",
        "borrow_value": "392050028572",
        "state": "submitted",
        "fill_fees": "0",
        "created_at": 1536113921,
        "updated_at": 1536113921
    }
}
```

## 获取指定订单
此 API 用于返回指定的订单详情。

### HTTP Request

`GET https://api.babelbank.io/v1/loan/orders/{order_id}`

### URL 参数

参数 | 描述
--------- | -----------
order_id | 订单 ID

> 响应结果如下：

```json
{
  "status": 0,
  "data": {
    "order_id": "9d17a03b852e48c0b3920c7412867623",
    "symbol": "BTC|USDT",
    "side": "loan",
    "pledge_currency": "BTC",
    "pledge_value": "100000000",
    "borrow_currency": "USDT",
    "borrow_value": "392050028572",
    "state": "submitted",
    "fill_fees": "0",
    "created_at": 1536111780,
    "updated_at": 1536111780
  }
}
```
## 更新托管账户记录（新增）

### HTTP Request

`POST https://api.babelbank.io/v1/trusteeship`

### 请求参数

参数 | 默认值 | 描述
--------- | ------- | -----------
`side` | `String` | 交易方向（`in进/out出`）
`currency` | `String` | 币种
`value` | `String` | 数量 * 10的8次方
`tx_hash` | `String` | 链上可查Transaction id


> 响应结果如下：

```json
{
    "status": 0
}
```

# 充币提币
## 模型说明

模型由以下属性构成：

属性 | 类型 | 含义解释
---------- | ------- | -------
`order_id` | `String` | 订单 ID
`user_id` | `String` | 用户id
`side` | `String` | 交易方向（`deposit`, `withdraw`）充币/提币
`currency` | `String` | 币种
`value` | `String` | 数量 * 10的8次方
`addr` | `String` | 钱包地址
`tx_hash` | `String` | 链上可查Transaction id
`state` | `String` | 订单状态
`fill_fees` | `String` | 手续费
`created_at` | `Long` | 创建时间



订单状态说明：

属性  | 含义解释
----------- | -------
`submitted` | 已提交
`pending` | 处理中
`successful` | 成功
`failure` | 失败


# 创建新的充币提币请求

### HTTP Request

`POST https://api.babelbank.io/v1/balance`

### 请求参数

参数 | 默认值 | 描述
--------- | ------- | -----------
`user_id` | `String` | 用户id
`side` | `String` | 交易方向（`deposit`, `withdraw`）充币/提币
`currency` | `String` | 币种
`value` | `String` | 数量 * 10的8次方
`addr` | `String` | 钱包地址
`tx_hash` | `String` | 链上可查Transaction id

> 响应结果如下：

```json
{
    "status": 0,
    "order_id": "9d17a03b852e48c0b3920c7412867623"
}
```

## 获取指定充币提币记录订单
此 API 用于返回指定的财务记录订单详情。

### HTTP Request

`GET https://api.babelbank.io/v1/balance/{user_id}/{order_id}`

### URL 参数

参数 | 描述
--------- | -----------
order_id | 交易订单号 
user_id | 用户标识
> 响应结果如下：

```json
{
  "status": 0,
  "data": {
    "order_id": "9d17a03b852e48c0b3920c7412867623",
    "side": "withdraw",
    "currency": "USDT",
    "value": "500000000",
    "addr": "",
    "state": "submitted",
    "fill_fees": "0",
    "tx_hash": "string",
    "created_at": 1536111780,
    "updated_at": 1536111780
  }
}
```


## 获取用户的充币提币记录列表
此 API 用于返回用户的财务记录列表。

### HTTP Request

`GET https://api.babelbank.io/v1/balance/list`

### 查询参数(HTTP Query)

参数 | 默认值 | 描述
--------- | ------- | -----------
`user_id` | `String` | 用户id
`side` | `String` | 交易方向（`deposit`, `withdraw`）充币/提币 
`page_index` |`String`  | 当前页码
`page_size` | `String` |  每页的数量，默认为 20 条


> 响应结果如下：

```json
{
  "status": 0,
  "total": 1,
  "data": [
    {
      "order_id": "9f1302a6b02d11e8a2aef218982b54e4",
      "side": "deposit",
      "currency": "USDT",
      "value": "500000000",
      "addr": "",
      "state": "submitted",
      "fill_fees": "0",
      "tx_hash": "string",
      "created_at": 1536111780,
      "updated_at": 1536111780
    }
  ]
}
```

# Babel现有业务规则
  用户id(user_id)可以包含字母数字A-Z,a-z,0-9,长度64位以内
  USDT单笔最大借款金额10万，单笔最小借款金额1000
  BTC最小提币金额为0.002BTC
  USDT最小提币金额为10USDT
​        
​        
# HTTP header错误代码

错误代码 | 含义解释
---------- | -------
400 | Bad Request -- 错误的请求
401 | Unauthorized -- API key 或者签名，时间戳有误
403 | Forbidden -- 禁止访问
404 | Not Found -- 未找到请求的资源
405 | Method Not Allowed -- 使用的 HTTP 方法不适用于请求的资源
406 | Not Acceptable -- 请求的内容格式不是 JSON
429 | Too Many Requests -- 请求受限，请降低请求频率
500 | Internal Server Error -- 服务内部错误，请稍后再进行尝试
503 | Service Unavailable -- 服务不可用，请稍后再进行尝试

# 服务结果返回代码

> 成功结果事例：

```json
{
  "status": 0
}
```

> 失败结果事例：

```json
{
  "status": 1112,
  "detail": "tx_hash 重复"
}
```

代码 | 含义解释
---------- | -------
0 | 服务器处理成功
500 | 服务器内部错误
1000 | 参数异常
1001 | 用户不存在
1002 | 授信额度不足
1003 | 汇率不正确
1004 | 合作伙伴状态异常
1005 | user_id 包含非法字符
1006 | 交易对不存在
1101 | 不支持的side操作
1102 | value 过低
1103 | pledge_currency 找不到对应币种
1104 | 可用余额不足
1105 | borrow_currency 找不到对应币种
1106 | borrow value 须大于最小值
1107 | borrow value 须小于最大值
1108 | pledge value 须大于最小值
1109 | pledge value 须小于最大值
1110 | currency 找不到对应币种
1111 | tx_hash 参数不存在
1112 | tx_hash 重复
1113 | 提币金额大于可用余额
1114 | 提币金额须大于最小值


# 通知解决方案

## 通知概述

通知为确保可以更及时的获得消息, 选择采用了 WebSocket 方式进行接入.

所有 WebSocket 请求的 URL 为: `wss://api.babelbank.io/v1/ws`

下文会统一术语:

- `topic` 表示订阅的主题，服务器推送的消息类型
- `symbol` 表示对应交易币种. 所有币种区分的 topic 都在 topic 末尾.
- `ts` 表示推送服务器的时间. 是毫秒为单位的数字型字段, unix epoch in millisecond.
- `state` 表示推送服务器推送的消息的订单状态
- `tx_hash` 链上可查Transaction id
订单状态说明：
属性  | 含义解释
----------- | -------
`submitted` | 已提交
`pending` | 处理中
`successful` | 成功
`failure` | 失败

### WebSocket 首次建立链接

服务器会发送一个欢迎信息

- `ts`: 推送服务器当前的时间.

> 服务器返回

```json
{
  "type":"hello",
  "ts":1523693784042
}
```

## 获取推送服务器时间

- `gap`: 推送服务器处理此语句的时间和客户端传输的时间差.
- `ts`: 推送服务器当前的时间.

### WebSocket 请求

```python
import time
import babel

api = babel.authorize('key', 'secret', timestamp)
now_ms = int(time.time())
api.notify.ping(now_ms)
```


> 服务器返回

```json
{
  "type":"ping",
  "ts":1523693784042,
  "gap":112
}
```



# 通知消息实时推送


### WebSocket 订阅消息通知

```

let babel_ws = babel.init_ws()
topics = ["notify.business", "notify.warning", "notify.pitch_account"]
babel_ws.handle(print)
babel_ws.sub(topics)
```

> 订阅成功的响应结果如下：

```json
{
  "type": "topics",
  "topics": ["notify.business", "notify.forced_liquidation", "notify.warning", "notify.pitch_account"]
}
```


> 常规订阅的通知消息格式如下:

- `topic = notify.pitch_account` 当天账目差额(data*10的8次方)>100btc，触发通知预警进行对账，返回当天账目差额
```json
{
  "type":"notify.pitch_account",
  "ts": "1523619211000",
  "data":"10000000000"
}
```

- `topic = notify.business` 业务通知：
    还款 --通知cobo充币成功，进行创建订单还款操作
    提币 --链上成功后异步通知

```json
{
  "type":"notify.business",
  "ts": "1523619211000",
  "data":
   {
     "order_id": "9d17a03b852e48c0b3920c7412867623",
     "tx_hash": "9d17a03b852e48c0b3920c7412867623",
     "state":"successful"
   }
}
```

- `topic = notify.warning` 强平预警通知,每个账户一小时发送一次：
1、预警线 pledge_rate >= 80%
2、行动线 pledge_rate >= 85%
3、强平触发 pledge_rate >= 90%
```json
{
  "type":"notify.warning",
  "ts": "1523619211000",
  "data":
   {
     "user_id": "9d17a03b852e48c0b3920c7412867623",
     "symbol": "BTC|USDT"
     "pledge_currency": "BTC",
     "borrow_currency": "USDT",
     "pledge_value": "22571356",
     "borrow_value": "100000000000",
     "pledge_rate": "80000000"
   }
}
```

- `topic = notify.forced_liquidation` 强平完成通知：
```json
{
    "type":"notify.forced_liquidation",
    "ts": "1523619211000",
    "data":
    {
        "user_id": "9d17a03b852e48c0b3920c7412867623",
        “order_id”: 0c0283dab0ad11e8b8adf218982b54e4
        "symbol": "BTC|USDT"
        "pledge_currency": "BTC",
        "borrow_currency": "USDT",
        "pledge_value": "22571356",
        "borrow_value": "100000000000",
        "pledge_rate": "80000000"
    }
}
```
属性 | 类型 | 含义解释
---------- | ------- | -------
`user_id` | `String` | 用户 ID
`order_id` | `String` | 订单 ID
`symbol` | `String` | 交易对
`pledge_currency` | `String` | 质押币种
`pledge_value` | `String` | 平仓数量 * 10的8次方
`borrow_currency` | `String` | 借出币种
`borrow_value` | `String` | 平仓数量 * 10的8次方

