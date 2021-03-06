# API_Docs

# 现货
## REST行情、交易API<br>
 [现货接口示例](https://documenter.getpostman.com/view/4734616/S17kzWdW)

## WebSocket 行情,交易 API<br>
|**接口类型**|   **接口数据类型**   |**请求方法** |**类型**   |**描述**                     |**需要验签**        |                                                                                                                                            
|----------- |------------------ |------------------------------------------------------------------------------------------------------------------------------------------------------------------- |---------- |---------------------------- |--------------|
|Websocket   |市场行情接口|market.$symbol.kline.$period|sub|订阅 KLine 数据              |否|
|Websocket   |市场行情接口|market.$symbol.kline.$period|req|请求 KLine 数据              |否|
|Websocket   |市场行情接口|market.$symbol.depth.$type | sub|订阅 Market Depth 数据       |否|
|Websocket   |市场行情接口|market.$symbol.depth.$type | req|请求 Market Depth 数据       |否|
|Websocket   |市场行情接口|market.$symbol.detail|       sub|订阅 Market detail 数据      |否|
|Websocket   |市场行情接口|market.$symbol.detail|       sub|请求 Market detail 数据      |否|
|Websocket   |市场行情接口|market.$symbol.trade.detail| req|请求 Trade detail 数据       |否|
|Websocket   |市场行情接口|market.$symbol.trade.detail| sub|订阅 Trade Detail 数据       |否|



## 订阅 KLine 数据 market.$symbol.kline.$period

成功建立和 WebSocket API 的连接之后，向 Server 发送如下格式的数据来订阅数据：
```
{
  "sub": "market.$symbol.kline.$period",
  "id": "id generate by client"
}
```

| 参数名称         | 是否必须  | 类型     | 描述                | 默认值   | 取值范围                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易对                |       | ETH/USDT, LTC/USDT, ETC/BTC, BCH/BTC... |
| period       | true  | string | K线周期          |       | 1min, 5min, 15min, 30min, 60min,day |

**正确订阅的例子**

正确订阅
```
{
  "sub": "market.BTC/USDT.kline.1min",
  "id": "id1"
}
```

订阅成功返回数据的例子
```
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.BTC/USDT.kline.1min",
  "ts": 1489474081631
}
```

之后每当 KLine 有更新时，client 会收到数据，例子
```json
{
  "ch": "market.BTC/USDT.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```

tick 说明
```
  "tick": {
    "id": K线id,
    "amount": 成交量,
    "count": 成交笔数,
    "open": 开盘价,
    "close": 收盘价,当K线为最晚的一根时，是最新成交价
    "low": 最低价,
    "high": 最高价,
    "vol": 成交额, 即 sum(每一笔成交价 * 该笔的成交量)
  }

```

**错误订阅的例子**

错误订阅(错误的 symbol)
```
{
  "sub": "market.invalidsymbol.kline.1min",
  "id": "id2"
}
```

订阅失败返回数据的例子
```json
{
  "id": "id2",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.invalidsymbol.kline.1min",
  "ts": 1494301904959
}
```

错误订阅(错误的 topic)
```
{
  "sub": "market.BTC/USDT.kline.3min",
  "id": "id3"
}
```

订阅失败返回数据的例子
```json
{
  "id": "id3",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.BTC/USDT.kline.3min",
  "ts": 1494310283622
}
```

## 请求 KLine 数据 market.$symbol.kline.$period  

```
{
  "req": "market.$symbol.kline.$period",
  "id": "id generated by client",
  "from": 1533536947, //optional, type: long, 2017-07-28T00:00:00+08:00 至 2050-01-01T00:00:00+08:00 之间的时间点，单位：秒
  "to": 1533536947 //optional, type: long, 2017-07-28T00:00:00+08:00 至 2050-01-01T00:00:00+08:00 之间的时间点，单位：秒，必须比 from 大
}
```

| 参数名称         | 是否必须  | 类型     | 描述                | 默认值   | 取值范围                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易对                |       | ETH/USDT, LTC/USDT, ETC/BTC, BCH/BTC...|
| period       | true  | string | K线周期          |       | 1min, 5min, 15min, 30min, 60min,day  |


* 响应结果

`[t1, t5]`
假设有 t1 ~ t5 的K线.

* from: t1, to: t5, return [t1, t5].
* from: t5, to: t1, which t5 > t1, return [].
* from: t5, return [t5].
* from: t3, return [t3, t5].
* to: t5, return [t1, t5].
* from: t which t3 < t <t4, return [t4, t5].
* to: t which t3 < t <t4, return [t1, t3].
* from: t1 and to: t2, should satisfy 1501171200 < t1 < t2 < 2524579200.

```
一次最多300条
```

#### 请求 KLine 数据的例子

```
{
  "req": "market.BTC/USDT.kline.1min",
  "id": "id10"
}
```

返回数据的例子
```
{
  "rep": "market.BTC/USDT.kline.1min",
  "status": "ok",
  "id": "id10",
  "tick": [
    {
      "amount": 17.4805,
      "count":  27,
      "id":     1494478080,
      "open":   10050.00,
      "close":  10058.00,
      "low":    10050.00,
      "high":   10058.00,
      "vol":    175798.757708
    },
    {
      "amount": 15.7389,
      "count":  28,
      "id":     1494478140,
      "open":   10058.00,
      "close":  10060.00,
      "low":    10056.00,
      "high":   10065.00,
      "vol":    158331.348600
    },
    // more KLine data here
  ]
}
```

## 订阅 Market Depth 数据 market.$symbol.depth.$type

成功建立和 WebSocket API 的连接之后，向 Server 发送如下格式的数据来订阅数据：
```
{
  "sub": "market.$symbol.depth.$type",
  "id": "id generated by client"
}
```

| 参数名称         | 是否必须  | 类型     | 描述                | 默认值   | 取值范围                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易对                |       | BTC/USDT, ETH/USDT, LTC/USDT, ETC/BTC... |
| type         | true  | string | Depth 类型          |       | step1, step2, step3, step4, step5|


正确订阅例子
```
{
  "sub": "market.BTC/USDT.depth.step2",
  "id": "id1"
}
```

订阅成功返回数据的例子
```
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.BTC/USDT.depth.step2",
  "ts": 1489474081631
}
```

之后每当 depth 有更新时，client 会收到数据，例子
```
{
  "ch": "market.BTC/USDT.depth.step2",
  "ts": 1489474082831,
  "tick": {
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560],
    // more Market Depth data here
    ]
    "asks": [
    [10010.9800,0.0099]
    [10011.3900,2.0000]
    //more data here
    ]
  }
}
```

tick 说明
```
  "tick": {
    "bids": [
    [买1价,买1量]
    [买2价,买2量]
    //more data here
    ]
    "asks": [
    [卖1价,卖1量]
    [卖2价,卖2量]
    //more data here
    ]
  }
```

## 请求 Market Depth 数据 market.$symbol.depth.$type

```
{
  "req": "market.$symbol.depth.$type",
  "id": "id generated by client"
}
```

#### 请求 Market Depth 数据的例子
```
{
  "req": "market.BTC/USDT.depth.step2",
  "id": "id10"
}
```

返回数据的例子：
```
{
  "rep": "market.BTC/USDT.depth.step2",
  "status": "ok",
  "id": "id10",
  "tick": {
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560],
    // more Market Depth data here
    ]
    "asks": [
    [10010.9800,0.0099]
    [10011.3900,2.0000]
    //more data here
    ]
  }
}
```

## 订阅 Trade Detail 数据 market.$symbol.trade.detail  

```
{
  "sub": "market.$symbol.trade.detail",
  "id": "id generated by client"
}
```

| 参数名称         | 是否必须  | 类型     | 描述                | 默认值   | 取值范围                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易对                |       | ETH/USDT, LTC/USDT, ETC/BTC, BCH/BTC... |

正确订阅例子：

```
{
  "sub": "market.BTC/USDT.trade.detail",
  "id": "id1"
}
```

订阅成功返回数据的例子：
```
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.BTC/USDT.trade.detail",
  "ts": 1489474081631
}
```

之后每当 Trade Detail 有更新时，client 会收到数据，例子：

```
{
  "ch": "market.BTC/USDT.trade.detail",
  "ts": 1489474082831,
  "tick": {
        "id": 14650745135,
        "ts": 1533265950234,
        "data": [
            {
                "amount": 0.0099,
                "ts": 1533265950234,
                "id": 146507451359183894799,
                "price": 401.74,
                "direction": "buy"
            },
            // more Trade Detail data here
        ]
    }

  ]
  }
}
```

data 说明：
```
  "data": [
    {
      "id":        消息ID,
      "price":     成交价,
      "time":      成交时间,
      "amount":    成交量,
      "direction": 成交方向,
      "tradeId":   成交ID,
      "ts":        时间戳
    }
  ]
```

## 请求 Trade Detail 数据 market.$symbol.trade.detail

```
{
  "req": "market.$symbol.trade.detail",
  "id": "id generated by client"
}
```

* 仅能获取最近 300 个 Trade Detail 数据

#### 请求 Trade Detail 数据的例子
```
{
  "req": "market.BTC/USDT.trade.detail",
  "id": "id11"
}
```

返回数据的例子：
```
{
  "rep": "market.BTC/USDT.trade.detail",
  "status": "ok",
  "id": "id11",
  "data": [
    {
      "id":        601595424,
      "price":     10195.64,
      "time":      1494495766,
      "amount":    0.2943,
      "direction": "buy",
      "tradeId":   601595424,
      "ts":        1494495766000
    },
    {
      "id":        601595423,
      "price":     10195.64,
      "time":      1494495711,
      "amount":    0.2430,
      "direction": "buy",
      "tradeId":   601595423,
      "ts":        1494495711000
    },
    // more Trade Detail data here
  ]
}
```
## 订阅 Market Detail 数据 market.$symbol.detail
```
{
  "req": "market.$symbol.detail",
  "id": "id generated by client"
}
```

* 仅返回当前 Market Detail

#### 订阅 Market Detail 数据的例子

```
{
  "sub": "market.BTC/USDT.detail",
  "id": "id12"
}
```

返回数据的例子：
```json
{
  "ch": "market.BTC/USDT.detail",
  "status": "ok",
  "id": "id12",
  "tick": {
    "amount": 12224.2922,
    "open":   9790.52,
    "close":  10195.00,
    "high":   10300.00,
    "ts":     1494496390000,
    "id":     1494496390,
    "count":  15195,
    "low":    9657.00,
    "vol":    121906001.754751
  }
}
```
## 请求 Market Detail 数据 market.$symbol.detail
```
{
  "req": "market.$symbol.detail",
  "id": "id generated by client"
}
```

* 仅返回当前 Market Detail

#### 请求 Market Detail 数据的例子

```
{
  "req": "market.BTC/USDT.detail",
  "id": "id12"
}
```

返回数据的例子：
```json
{
  "rep": "market.BTC/USDT.detail",
  "status": "ok",
  "id": "id12",
  "tick": {
    "amount": 12224.2922,
    "open":   9790.52,
    "close":  10195.00,
    "high":   10300.00,
    "ts":     1494496390000,
    "id":     1494496390,
    "count":  15195,
    "low":    9657.00,
    "vol":    121906001.754751
  }
}
```


# 合约
## REST 行情、交易 API<br>
[合约接口示例](https://documenter.getpostman.com/view/4734616/S17kzWdV)
## WebSocket 行情,交易 API<br>
|**接口类型**|   **接口数据类型**   |**请求方法** |**类型**   |**描述**                     |**需要验签**        |                                                                                                                                            
  |----------- |------------------ |------------------------------------------------------------------------------------------------------------------------------------------------------------------- |---------- |---------------------------- |--------------|
  |Websocket   |市场行情接口 |         market.$symbol.kline.$period|                                                                                                            sub        |订阅 KLine 数据              |否|
  |Websocket   |市场行情接口|           market.$symbol.kline.$period|                                                                                                                    req        |请求 KLine 数据              |否|
  |Websocket   |市场行情接口           |market.$symbol.depth.$type |                                                                                                                     sub        |订阅 Market Depth 数据       |否|
 |Websocket   |市场行情接口           |market.$symbol.detail|                                                                                                                     sub        |订阅 Market detail 数据       |否|
  |Websocket   |市场行情接口           |market.$symbol.trade.detail |                                                                                                                     req        |请求 Trade detail 数据       |否|
  |Websocket   |市场行情接口           |market.$symbol.trade.detail|        sub |订阅 Trade Detail 数据  | 否  |

#### 订阅 KLine 数据

成功建立和 WebSocket API 的连接之后，向 Server发送如下格式的数据来订阅数据：

    {
    "sub": "market.$symbol.kline.$period",
    "id": "id generate by client"
    }

  |**参数名称**|   **是否必须**  | **类型**  | **描述** |  **默认值**  | **取值范围**
  |--------------| -----------------| ---------- |----------| ------------| --------------------------------------------------------------------------------|
  |symbol  |       true         |  string  |   交易对   |               |如"BTC\_CW"表示BTC当周合约，"BTC\_NW"表示BTC次周合约，"BTC\_CQ"表示BTC季度合约|
  |period    |     true          | string   |  K线周期     |            |1min, 5min, 15min, 30min, 1hour,4hour,1day, 1mon|

正确订阅请求参数的例子：

    {
    "sub": "market.BTC_CQ.kline.1min",
    "id": "id1"
    }

订阅成功返回数据的例子

    {
    "id": "id1",
    "status": "ok",
    "subbed": "market.BTC_CQ.kline.1min",
    "ts": 1489474081631
    }

之后每当 KLine 有更新时，client 会收到数据，例子

    {
     "ch": "market.BTC_CQ.kline.1min",
     "ts": 1489474082831,
     "tick":
        {
         "id": 1489464480,
         "mrid": 268168237,
         "vol": 100,
         "count": 0,
         "open": 7962.62,
         "close": 7962.62,
         "low": 7962.62,
         "high": 7962.62,
         "amount": 0.3 //单位BTC 币种的数量
        }
    }

    tick 说明

    "tick":
      {
       "id": K线id,
       "mrid": 268168237,
       "vol": 成交量张数,
       "count": 成交笔数,
       "open": 开盘价,
       "close": 收盘价,当K线为最晚的一根时，是最新成交价
       "low": 最低价,
       "high": 最高价,
       "amount": BTC, 即 sum(每一笔 该合约面值* 该笔的成交量/成交价)
    }

**错误订阅的列子**

错误订阅(错误的 symbol)

    {
     "sub": "market.invalidsymbol.kline.1min",
     "id": "id2"
    }

订阅失败返回数据的例子

    {
     "id": "id2",
     "status": "error",
     "err-code": "bad-request",
     "err-msg": "invalid topic market.invalidsymbol.kline.1min",
     "ts": 1494301904959
    }

错误订阅(错误的 topic)

    {
     "sub": "market.BTC_CQ.kline.3min",
     "id": "id3"
    }

订阅失败返回数据的例子

    {
     "id": "id3",
     "status": "error",
     "err-code": "bad-request",
     "err-msg": "invalid topic market.BTC_CQ.kline.3min",
     "ts": 1494310283622
    }

#### 请求 KLine 数据 </a>

成功建立和 WebSocket API 的连接之后，向Server发送如下格式的数据来请求数据：

    {
     "req": "market.$symbol.kline.$period",
     "id": "id generated by client",
     "from": "optional, type: long, 2017-07-28T00:00:00+08:00 至2050-01-01T00:00:00+08:00 之间的时间点，单位：秒",
      "to": "optional, type: long, 2017-07-28T00:00:00+08:00 至2050-01-01T00:00:00+08:00 之间的时间点，单位：秒，必须比 from 大"}

| **参数名称** | **是否必须** | **类型** | **描述** | **默认值** |**取值范围** |
| -------- | -------- | ------ | ------ | ------- |---------------------------------------- |
| symbol | true | string |交易对 | |如"BTC\_CW"表示BTC当周合约，"BTC\_NW"表示BTC次周合约，"BTC\_CQ"表示BTC季度合约|
 | period | true | string | K线周期 | | 1min, 5min, 15min, 30min, 1hour,4hour,1day, 1mon|

    [t1, t5] 假设有 t1  ~ t5 的K线：

    from: t1, to: t5, return [t1, t5].
    from: t5, to: t1, which t5  > t1, return [].
    from: t5, return [t5].
    from: t3, return [t3, t5].
    to: t5, return [t1, t5].
    from: t which t3  < t  <t4, return [t4, t5].
    to: t which t3  < t  <t4, return [t1, t3].
    from: t1 and to: t2, should satisfy 1325347200  < t1  < t2  < 2524579200.

**备注**：一次最多300条

请求 KLine 数据请求参数的例子：

    {
    "req": "market.BTC_CQ.kline.1min",
    "id": "id4"
    }

请求成功返回数据的例子：

    {
     "rep": "market.BTC_CQ.kline.1min",
     "status": "ok",
     "id": "id4",
     "tick": [
       {
        "vol": 100,
        "count": 27,
        "id": 1494478080,
        "open": 10050.00,
        "close": 10058.00,
        "low": 10050.00,
        "high": 10058.00,
        "amount": 175798.757708
       },
       {
        "vol": 300,
        "count": 28,
        "id": 1494478140,
        "open": 10058.00,
        "close": 10060.00,
        "low": 10056.00,
        "high": 10065.00,
        "amount": 158331.348600
       }
     ]
    }

#### 订阅 Market Depth 数据 </a>

成功建立和 WebSocket API 的连接之后，向 Server发送如下格式的数据来订阅数据：

    {
    "sub": "market.$symbol.depth.$type",
    "id": "id generated by client"
    }

  |**参数名称**   |**是否必须**   |**类型**   |**描述**     |**默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |------------ |------------ |---------------------------------------------------------------------------------|
  |symbol         |true           |string     |交易对            |        |如"BTC\_CW"表示BTC当周合约，"BTC\_NW"表示BTC次周合约，"BTC\_CQ"表示BTC季度合约.|
  |type           |true           |string     |Depth 类型        |        |(150档数据)  step0, step1, step2, step3, step4, step5（合并深度1-5）,step0时，不合并深度;(20档数据)  step6, step7, step8, step9, step10, step11（合并深度7-11）；step6时，不合并深度|

**备注**：用户选择“合并深度”时，一定报价精度内的市场挂单将予以合并显示。合并深度仅改变显示方式，不改变实际成交价格。

正确订阅请求参数的例子：

    {
    "sub": "market.BTC_CQ.depth.step0",
    "id": "id5"
    }

订阅成功返回数据的例子：

    {
      "id": "id5",
      "status": "ok",
      "subbed": "market.BTC_CQ.depth.step0",
      "ts": 1489474081631
    }

之后每当 depth 有更新时，client 会收到数据，例子：

    {
     "ch": "market.BTC_CQ.depth.step0",
     "ts": 1489474082831,
     "tick":
       {
        "mrid": 269073229,
         "id": 1539843937,
            "bids": [
             [9999.9101,1],
             [9992.3089,2]
                    ],
             "asks": [
              [10010.9800,10],
              [10011.3900,15]
                     ],
	      "ts": 1539843937417,
	       "version": 1539843937,
	       "ch": "market.BTC_CQ.depth.step0"
        }
    }

    tick 说明：
    "tick": {
      "bids": [
        [买1价,买1量]
        [买2价,买2量]
         //more data here
       ],
       "asks": [
        [卖1价,卖1量]
        [卖2价,卖2量]
        //more data here
       ]
    }

#### 订阅 Market Detail 数据 </a>

成功建立和 WebSocket API 的连接之后，向 Server发送如下格式的数据来请求数据:

    {
     "sub": "market.$symbol.detail",
     "id": "id generated by client"
    }

  |**参数名称**   |**是否必须**   |**类型**   |**描述**     |**默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |------------ |------------ |--------------------------------------------------------------------------------|
  |symbol         |true           |string     |交易对      |              |如"BTC\_CW"表示BTC当周合约，"BTC\_NW"表示BTC次周合约，"BTC\_CQ"表示BTC季度合约|

订阅 Market Detail 数据请求参数的例子：

    {
     "sub": "market.BTC_CQ.detail",
     "id": "id6"
    }

请求成功返回数据的例子：

    {
	"ch": "market.BTC_CW.detail",
	"ts": 1539842340724,
	"tick": {
		"id": 1539842340,
		"mrid": 268041138,
		"open": 6740.47,
		"close": 7800,
		"high": 7800,
		"low": 6726.13,
		"amount": 477.1200312075244664773339914558562673572,
		"vol": 32414,
		"count": 1716
	        }
       }

tick数据说明：

    "tick":
         {
           "id": id,
           "mrid": 1494496390000,
           "vol": 成交量(张),
           "count": 成交笔数,
           "open": 开盘价,
           "close": 收盘价,当K线为最晚的一根时，是最新成交价
           "low": 最低价,
           "high": 最高价,
           "amount": 成交量(币), 即 sum(每一笔成交量(张)*单张合约面值/该笔成交价)
         }

#### 请求 Trade Detail 数据

成功建立和 WebSocket API 的连接之后，向 Server 发送如下格式的数据来请求数据：

    {
     "req": "market.$symbol.trade.detail",
     "id": "id generated by client"
    }

仅返回当前 Trade Detail

请求 Market Detail 数据请求参数的例子：

    {
     "req": "market.BTC_CQ.trade.detail",
     "id": "id8"
    }

请求成功返回数据的例子：



    {
     "ch": "market.BTC_CQ.trade.detail",
     "ts": 1489474082831,
     "data": [
              {
               "id":601595424,
               "price":10195.64,
               "amount":100,
               "direction":"buy",
               "ts":1494495766000
               },
              {
              "id":601595423,
              "price":10195.64,
              "amount":200,
              "direction":"buy",
              "ts":1494495711000
              }
            ]
     }

tick数据说明：

    "data": [
      {
       "id": 消息ID,
       "price": 成交价,
       "amount": 成交量（张）,
       "direction": 成交方向,
       "ts": 时间戳
      }
     ]

#### 订阅 Trade Detail 数据 </a>

成功建立和 WebSocket API 的连接之后，向 Server发送如下格式的数据来订阅数据：

    {
    "sub": "market.$symbol.trade.detail",
    "id": "id generated by client"
    }

**备注**：仅能获取最近 300 个 Trade Detail 数据。

  |**参数名称**   |**是否必须**   |**类型**   |**描述**   |**默认值**   |**取值范围**|
  |-------------- |-------------- |---------- |---------- |------------ |--------------------------------------------------------------------------------|
  |symbol         |true           |string     |合约名称    |            |如"BTC\_CW"表示BTC当周合约，"BTC\_NW"表示BTC次周合约，"BTC\_CQ"表示BTC季度合约|

正确订阅请求参数的例子：

    {
     "sub": "market.BTC_CQ.trade.detail",
     "id": "id7"
    }

订阅成功返回数据的例子：

    {
     "id": "id7",
     "status": "ok",
     "subbed": "market.BTC_CQ.trade.detail",
     "ts": 1489474081631
    }

之后每当 Trade Detail 有更新时，client 会收到数据，例子：

    {
	"ch": "market.BTC_NW.trade.detail",
	"ts": 1539831709042,
	"tick": {
		"id": 265842227,
		"ts": 1539831709001,
		"data": [{
			"amount": 20,
			"ts": 1539831709001,
			"id": 265842227259096443,
			"price": 6742.25,
			"direction": "buy"
		        }]
	          }
     }

data 说明：

    "data": [
      {
       "id": 消息ID,
       "price": 成交价,
       "amount": 成交量（张）,
       "direction": 成交方向,
       "ts": 时间戳
      }
     ]

# websocket 请求与订阅说明<br>

### 1. 数据压缩
WebSocket API 返回的所有数据都进行了 GZIP 压缩，需要 client 在收到数据之后解压，推荐使用pako。（[【pako】](https://github.com/nodeca/pako) 是一个支持压缩和解压 GZIP 的库）

### 2. WebSocket库
[【ws】](https://github.com/websockets/ws) 是 Node.js 下的 WebSocket 库。

### 3. 心跳
WebSocket API 支持双向心跳，无论是 Server 还是 Client 都可以发起 `ping` message，对方返回 `pong` message。

WebSocket Server 发送心跳：
```
{"ping": 18212558000}
```
 WebSocket Client 应该返回：
```
 {"pong": 18212558000}
```
注：返回的数据里面的 `"pong"` 的值为收到的 `"ping"` 的值
注：WebSocket Client 和 WebSocket Server 建立连接之后，WebSocket Server 每隔 `5s`（这个频率可能会变化） 会向 WebSocket Client 发起一次心跳，WebSocket Client 忽略心跳2次后，WebSocket Server 将会主动断开连接；WebSocket Client发送最近2次心跳message中的其中一个`ping`的值，WebSocket Server都会保持WebSocket连接。
```
┌────────┐                         ┌────────┐
│ Client │                         │ Server │
└───┬────┘                         └───┬────┘
    │         {"ping": 18212558000}  │
    │<─────────────────────────────────┤
    │                                  │ wait 5s
    │                                  ├───┐
    │                                  │<──┘
    │         {"ping": 18212558000}  │
    │<─────────────────────────────────┤
    │                                  │
    │ {"pong": 18212558000}          │
    ├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄>│
    │                                  │
```
注：WebSocket Client 发送最近2次心跳 message 中的其中一个 `"ping"` 的值，WebSocket Server 都会保持 WebSocket 连接
```
┌────────┐                         ┌────────┐
│ Client │                         │ Server │
└───┬────┘                         └───┬────┘
    │         {"ping": 1523778470416}  │
    │<─────────────────────────────────┤
    │                                  │ wait 5s
    │                                  ├───┐
    │                                  │<──┘
    │         {"ping": 1523778475416}  │
    │<─────────────────────────────────┤
    │                                  │ wait 5s
    │                                  ├───┐
    │                                  │<──┘
    │                                  │
    │                                  │ close WebSocket connection
    │                                  ├───┐
    │                                  │<──┘
    │                                  │

```
