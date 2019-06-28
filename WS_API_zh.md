### **1.  访问地址**
```
wss://api.qb.com/api/ws/v1
```


### **2.  数据压缩**

WebSocket API 返回的所有数据都进行了 GZIP 压缩，需要 client 在收到数据之后解压。



### **3.  请求格式**

WebSocket API 支持订阅服务。订阅请求格式如下：
```
{
  "id": "id generate by client",    //string类型
  "sub": "topic to sub",            //string
}
```
订阅成功返回数据的格式：
```
{
  "id": "id client passed",
  "status": "ok",
  "subbed": "topic to sub",
  "ts": 1489474081              //单位秒 Long类型
}
```
订阅成功后有数据更新会自动推送给客户端，格式：
```
{
  "topic": "message topic",     //客户端订阅的topic
  "Data": ... ,                 //返回实体
  "ts": 1489474081              //单位秒 Long类型
}
```
订阅后可以取消订阅，格式：
```
{
  "id": "id generate by client",        //string类型
  "unsub": "topic to unsub",            //string
}
```
取消订阅成功返回数据的格式：
```
{
  "id": "id client passed",
  "status": "ok",
  "unsubbed": "topic to sub",
  "ts": 1489474081  //单位秒 Long类型
}
```


### **4.  签名**

连接到Websocket API服务器后第一次请求需要发送签名（成功后该ws连接后续请求不需要签名），sign=XXXX。 生成签名的步骤及规则：

*  将请求中所有参数（值为空字符串除外）以参数名按字典序排序；
*  将排序好的参数键值对用"&"拼接（value用urlencode编码）,例如：k1=v1&k2=v2&k3=v3；
*  用Hmac SHA256算法对步骤2）中生成的进行hash h=hmacsha256("secret",'k1=v1&k2=v2&k3=v3')
*  对上述hash值进行base64编码,并转小写 sign=lower(base64(h))

公共参数：

| 参数名	| 类型 |  说明 |  是否必须  |  默认值  |  备注  |
| --- | --- | --- | --- | --- | --- |
|accessKey	| string  | 平台分配的accessKey	  |Y	| | |
|signV	| string  | 签名版本，固定为1	  |Y	|1 | |
|ts	        |string	  |客户端当前时间戳（精确到秒）|Y		| | 客户端必须校对本地时间，服务器时间差不能大于30秒 |
|sign	|string	|签名	|Y	| | |

### **5.  心跳**

WebSocket API 支持双向心跳，无论是 Server 还是 Client 都可以发起 ping message，对方返回 pong message。

WebSocket Server 发送心跳：
```
{"ping": 72837823273}
```
WebSocket Client 应该返回：
```
{"pong": 72837823273}
```
注：返回的数据里面的 "pong" 的值为收到的 "ping" 的值 注：WebSocket Client 和 WebSocket Server 建立连接之后，WebSocket Server 每隔 5s（这个频率可能会变化） 会向 WebSocket Client 发起一次心跳，WebSocket Client 忽略心跳2次后，WebSocket Server 将会主动断开连接；WebSocket Client发送最近2次心跳message中的其中一个ping的值，WebSocket Server都会保持WebSocket连接。

    ┌────────┐                         ┌────────┐
    │ Client │                         │ Server │
    └───┬────┘                         └───┬────┘
        │         {"ping": 72837823273}    │
        │<─────────────────────────────────┤
        │                                  │ wait 5s
        │                                  ├───┐
        │                                  │<──┘
        │         {"ping": 72837823274}    │
        │<─────────────────────────────────┤
        │                                  │
        │ {"pong": 72837823273}            │
        ├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄>│
        │                                  │

注：WebSocket Client 发送最近2次心跳 message 中的其中一个 "ping" 的值，WebSocket Server 都会保持 WebSocket 连接

    ┌────────┐                         ┌────────┐
    │ Client │                         │ Server │
    └───┬────┘                         └───┬────┘
        │         {"ping": 8534534554442}  │
        │<─────────────────────────────────┤
        │                                  │ wait 5s
        │                                  ├───┐
        │                                  │<──┘
        │         {"ping": 8534534554443}  │
        │<─────────────────────────────────┤
        │                                  │ wait 5s
        │                                  ├───┐
        │                                  │<──┘
        │                                  │
        │                                  │ close WebSocket connection
        │                                  ├───┐
        │                                  │<──┘
        │                                  │

注：WebSocket Client 忽略心跳2次后，WebSocket Server 将会主动断开连接。 WebSocket Client 发送心跳：
```
{"ping": 435375662}
```
注：发送的 message 里面，"ping" 的值必须为 Long 类型，否则返回错误信息：
```
{
  "ts": 1561366448,
  "status": "error",
  "err-code": "0x00002003",
  "err-msg": "invalid ping"
}
```
WebSocket Server 会返回：
```   
{"pong": 435375662}
```
注：返回的数据里面的 "pong" 的值为收到的 "ping" 的值



### **6.  订阅列表**

**k线信息**

*  topic： market.$symbol.kline.$period

*  参数：

   1.  $symbol 交易对，列如"btc_usdt"

   2.  $period 可选值：1min、5min、15min、30min、1hour、2hour、4hour、6hour、12hour、1day、1week

返回：

```
{
	"topic": "market.btc_usdt.kline.1min",
	"ts": 1561373988,
	"data": [{
		"startTime": 1561373940,     //起始时间
		"endTime": 1561374000,       //结束时间
		"open": 10944.25,            //开盘价
		"close": 10926.57,           //收盘价,当K线为最晚的一根时，是最新成交价
		"high": 10944.25,            //最高价
		"low": 10926.57,             //最低价
		"vol": 39.0135               //交易量
	}]
}
```


**深度信息**

*  topic： market.$symbol.depth.$type

*  参数：

   1.  $symbol 交易对，列如"btc_usdt"

   2.  $type 可选值：depth0、depth1、depth2、depth3、depth4

返回：

```
{
	"topic": "market.btc_usdt.depth.depth1",
	"ts": 1561373978,
	"data": {
		"buy": [{                   // 买盘深度数据
			"price": 10926.6,   // 电子币价格
			"amount": 0.7467    // 总量
		}, {
			"price": 10926.5,
			"amount": 0.7145
		}...],
		"sell": [{                  // 卖盘深度数据
			"price": 10926.8,
			"amount": 0.7164
		}...]
	}
}
```   


**24小时成交信息**

*  topic： market.$symbol.detail

*  参数：

   1.  $symbol 交易对，列如"btc_usdt"

返回：

```
{
	"topic": "market.btc_usdt.detail",
	"ts": 1561373983,
	"data": {
		"r": 0.0108,                   // 涨跌幅
		"price": 10935.4,              // 最新价
		"open": 10818.13,              // 24H开盘价
		"high": 11015.71,              // 最高价
		"low": 10816.66,               // 最低价
		"volume": 2418.694,            // 交易量
		"turnover": 26421703.671852,   // 交易额
		"ps": null                     // 7日k线的点
	}
}
```

**成交记录信息**

*  topic： market.$symbol.trade.detail

*  参数：

   1.  $symbol 交易对，列如"btc_usdt"

返回：

```
{
	"topic": "market.btc_usdt.trade.detail",
	"ts": 1561373988,
	"data": {
		"dealTime": 1561373988,   // 成交时间
		"price": 10926.57,        // 成交价格
		"volume": 0.705,          // 成交量
		"dealType": 0             // 成交类型  0买单 1卖单
	}
}
```

**账户变更信息**

*  topic： accounts

返回：

```
{
	"topic": "accounts",
	"ts": 1561374922,
	"data": {
		"btcVal": 876121.08635932,           //账户总比特币价值
		"dollarVal": 77582096933.83,         //账户总美元价值
		"rmbVal": 8568071590.67,             //账户总人民币价值
		"accounts": [{
			"coinName": "zec",           //币名称
			"fullName": "Zcash",         //币全称
			"avail": 10000000,           //可用余额
			"frozen": 0,                 //冻结金额
			"btcVal": 99650,             //比特币价值
			"dollarVal": 1084592630,     //美元价值
			"rmbVal": 7461895310         //人民币价值
		}, {
			"coinName": "bsv",
			"fullName": "BitcoinSV",
			"avail": 10000000,
			"frozen": 0,
			"btcVal": 218330,
			"dollarVal": 2376418280,
			"rmbVal": 16349534370
		}, ...]
	}
}
```    

**用户订单变化信息**

*  topic： orders.update

返回：

```
{
	"topic": "orders.update",
	"ts": 1561375266,
	"data": {
		"symbol": "btc_usdt",              // 交易对
		"orderType": 0,                    // 订单类型 0买单 1卖单
		"priceType": 0,                    // 市价限价 0限价 1市价
		"orderId": 363110875323736064,     // 订单id
		"orderTime": 1561375266,           // 订单创建时间
		"orderStatus": "partdone",         // 订单状态  undone(未成交)  partdone(部分成交) partdone and canceled(部分成交后完成撤销)  alldone(全部成交) canceled(已撤销) in cancellation(撤销中) failed(成交失败)
		"count": 0.01,                     // 交易量
		"dealDoneCount": 0.01,             // 已成交量
		"price": 10879.82,                 // 数字币价格
		"rmbPrice": 75604.5763683,         // 人民币价格
		"dollarPrice": 10989.18395064,     // 美元价格
		"avgPrice": 10836.7,               // 成交均价
		"rmbAvgPrice": 75304.9326855,      // 成交均价人民币
		"dollarAvgPrice": 10945.6305084,   // 成交均价美元
		"fee": 0.000012,                   // 手续费
		"turnover": 108.367,               // 成交额
		"rmbTurnover": 753.04932686,       // 成交额人民币
		"dollarTurnover": 109.45630508     // 成交额美元
	}
}
```


### **7.  错误编码**
|  errorCode   |  errorMsg   |
| --- | --- |
|0x00001001	| invalid request|
|0x00001002     | unknown request|
|0x00001003	| invalid  access key|
|0x00001004	| access key expired|
|0x00001005	| wrong time|
|0x00001006	| ip forbidden|
|0x00001007	| invalid sign|
|0x00002001	| invalid ping|
|0x00003001	| invalid topic|
|0x00003002	| not exists symbol|
|0x00003003	| symbol not online|
|0x00003004	| invalid period|
|0x00003005	| unsub with not subbed topic|
|0x00003006	| symbol unsupported depth type|
