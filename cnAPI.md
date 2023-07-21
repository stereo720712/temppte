# 快捷交易文档

### <a name="1">API说明 </a>

#### ChipPay概述
ChipPay是一家交易数字货币(USDT)平台，为商户提供了快捷买卖数字货币(USDT)服务。主要提供以下API：
- 查询最新交易价格
- 查询订单交易详情
- 查询用户交易情况
- 买入数字货币(USDT）& 卖出数字货币（USDT）


商户可以使用测试服务器进行测试，建议在测试环境对接完成后再切换生产环境。如遇到问题，请先查询文档常见问题说明，如还不能解决，可以咨询相关技术人员。
- 测试环境(<span style="color:red">测试环境请勿真实打款！！！</span>)

  >接口域名：`https://open-v2.chippaytest.com/`
- 正式环境
  >接口域名：`https://open-v2.chippay.com/`


<br>

#### 接口组装
- <span style="color:red">接口 = 接口域名 + 接口后缀</span>
- 比如买入接口

>测试环境
```txt
https://open-v2.chippaytest.com/cola/apiOpen/addOrder
```

>正式环境
```txt
https://open-v2.chippay.com/cola/apiOpen/addOrder
```


### <a name="2">密钥对生成 & 签名规则 </a>

> 签名使用SHA256withRSA算法. 生成的私钥对请求参数（按abcd顺序排列，若遇到相同首字母，则看第二个字母，以此类推）进行签名，得到的输出即为签名（该字符串赋值于参数sign）。


* 非对称签名公私钥生成说明
  * 采用RSA 生成非对称公私钥，密钥强度1024 bit，导出格式为DER PKCS#8，具体生成方式请参考sdk
  * 商户端保留私钥，私钥需要妥善保存，不能泄露给其他人，公钥给到平台
  * 商户用私钥进行请求签名，平台用公钥进行签名校验
  * 请商户配置正确的DTO格式进行对接 请勿使用类似 Map<String, String> 格式，对签名结果有疑问时，请提供所生成的待签名的字符串，确保资料格式正确


```txt
注意：
  选填参数没有传给ChipPay就不要参与签名， 如果传了不论为空还是其他数据都需要参与签名。 

  将待签名字符串要求按照参数名进行排序(首先比较所有参数名的第一个字母，按abcd顺序排列，若遇到相同首 字母，则看第二个字母，以此类推)。

```

-  下单请求示例
   例如：对于如下的参数进行签名

```json	
   {
         "areaCode":"86",
         "asyncUrl":"127.0.0.1:8090/v1/demo/return.php",
         "coinSign":"USDT",
         "coinAmount":"20",
         "companyId":"12511234561",
         "companyOrderNum":"NB1594174982945",
         "idCardNum":"430524143201097878",
         "idCardType":"1",
         "kyc":"2",
         "orderPayChannel":"3",
         "orderTime":"1594174982945",
         "orderType":"1",
         "payCoinSign":"cny",
         "phone":"18900000001",
         "syncUrl":"127.0.0.1:8090/v1/demo/return.php",
         "total":"200",
         "username":"范思哲"
   } 
```
* 生成【待签名的字符串】

````java
       areaCode=86&asyncUrl=127.0.0.1:8090/v1/demo/return.php&coinAmount=20&coinSign=USDT&companyId=12511234561&companyOrderNum=NB1594174982945&idCardNum=430524143201097878&idCardType=1&kyc=2&orderPayChannel=3&orderTime=1594174982945&orderType=1&payCoinSign=cny&phone=18900000001&syncUrl=127.0.0.1:8090/v1/demo/return.php&total=200&username=范思哲


````

* 使用SHA256withRSA算法. 生成的私钥对排序后的字符串进行签名，签名后的结果赋值于参数sign
````json
   {
  "areaCode":"86",
  "asyncUrl":"127.0.0.1:8090/v1/demo/return.php",
  "total":"200",
  "coinAmount":"20",
  "coinSign":"USDT",
  "companyId":"12511234561",
  "companyOrderNum":"NB1594174982945",
  "idCardNum":"430524143201097878",
  "idCardType":"1",
  "kyc":"2",
  "orderPayChannel":"3",
  "orderTime":"1594174982945",
  "orderType":"1",
  "payCoinSign":"cny",
  "phone":"18900000001",
  "syncUrl":"127.0.0.1:8090/v1/demo/return.php",
  "username":"范思哲",
  "sign":"fjQBh3XbPlUWYQ+y+5HM760n39212c3nqHgxOk/d02wBlONPIhO1Y5QM7vS5FDcrbQeIfVpxJPRjESWSitFCReXJHFGkEh2tceKrXyRcupkqzPnquNHrSQuuA4uDp3DeAyqVMxO+N8+rDu3KTcZL8egMIiUFAKjQgX+i4TgeKFU="
}  
````
### <a name="3">交易接口 </a>

##### <a name="8a"><span style = "color: red">重要说明</span></a>
* <span style = "color: red">快捷买单 & 快捷卖单(卖单需求推荐使用快捷批量卖单接口): 接口实时返回下单结果，同一客户在同一时间只能允许存在一笔处理中订单。</span>

* <span style = "color: red">快捷批量卖单: 同一客户同一时间可送多笔订单，订单将队列进行处理。该批次提交成功不代表所有订单都可成功生成，若有订单未生成(如商户余额不足...等原因)则回调的tradeStatus将为2</span>
* <span style = "color: red">请求下单接口如果没有响应或者超时（如：502、504），请在1分钟后通过查询接口确认是否成功，或登入ChipPay商户后台核实订单是否生成，避免双方订单处理不一致。  </span>
* <span style = "color: red"> 订单号不能重复，如果同一笔订单，请不要更换订单号重复下单，避免重复出款。  </span>  


##### <a name="3a">快捷买单 & 快捷卖单</a>


* 请求url

  > {{domain}}/cola/apiOpen/addOrder
* 请求格式

  > <span style = "color: red;font-size:20px;font-weight:bold">httppost+json</span>
* 请求头部

> <span style = "color: red;font-size:20px;font-weight:bold">Content-Type:application/json  </span>

* 请求参数

|参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|
|companyId|商户id|long|20|Y| |
|kyc|用户验证级别|string||Y|2|
|username|用户姓名|string|中文: 2-15 位;英文: 2-35位|Y| |
|areaCode|国际区号 |string||N|86|
|phone|手机号|string||Y| |
|email|用户邮箱，只支持当payCoinSign为`vnd`时传输，phone或者email需择一传输|string||N| |
|orderType|订单类型1、快捷买单 2、快捷卖单|Integer||Y| |
|idCardType|证件类型(1.身份证 2.护照 3.其他)|Integer||N| |
|idCardNum|证件号码|string||N| |
|payCardNo|银行卡号（快捷卖单必填）|string||N| |
|payCardBank|开户银行（快捷卖单必填）,当payCoinSign为`vnd`时需准确填入银行名称,参考<a href="#vnd_bank_area">vnd区银行名称</a>|string||N| |
|payCardBranch|开户支行|string||N| |
|companyOrderNum|商户订单号|string||Y| |
|coinSign|数字货币标识(USDT)|string||Y| |
|payCoinSign|法币币别，须传小写英文(cny，vnd)|string||Y| |
|coinAmount|USDT下单数字货币数量(coinAmount和 total 两个字段二选一，当两个字段都填写的时候，优先处理total)coinAmount参数换算后的法币金额若不为整数，将无条件进位为整数显示于收银台|BigDecimal||Y| |
|total|用户付款的法币总金额(快捷买单只能传整数，快捷卖单不限) |BigDecimal||N| |
|orderPayChannel|当payCoinSign为`cny`时支持 3.银行卡 支付方式，当payCoinSign为`vnd`时支持 1.MOMO 3.Bank card 支付方式[vnd快捷卖单只支持`Bank card` 支付方式]|Integer||N|3|
|displayUnitPrice|客户自定义单价（最多接收四位小数）详见<a href ="#trading_rules">交易规则&常见问题</a> |BigDecimal||N| |
|orderTime|	订单时间戳（使用当前时间戳，与当前时间相差5分钟视为无效） |string||Y||
|syncUrl|同步回调地址|string||Y| |
|asyncUrl|异步回调地址|string||Y| |
|sign|签名|string||Y| &nbsp; |

* 请求示例

 ```json
 {
  "companyId":"12511234561",
  "kyc":"2",
  "username":"范思哲",
  "areaCode":"86",
  "phone":"18900000077",
  "orderType":1,
  "idCardType":1,
  "idCardNum":"430524143201097878",
  "companyOrderNum":"NB1594174989999",
  "coinSign":"USDT",
  "payCoinSign":"cny",
  "coinAmount":20,
  "orderPayChannel":3,
  "orderTime":"1617935991616",
  "total":200,
  "syncUrl":"127.0.0.1:8090/v1/demo/return.php",
  "asyncUrl":"127.0.0.1:8090/v1/demo/return.php",
  "sign":"BaTUESdfEKFTWcEn2a/r2Qenoz2sPN4HQafIqQtfPhlRtSbiprUuFNTBBymg2R5wxMRBCJySlagoPoqSyCAxW39vJPErPmpWbqLyoWjRgs6HSJPUE4HPVFqBHNNopOrKmValtozZKJEWZxTp3g6nv19dBVmgNCLnckx+N8YbC+Q="

} 
 ```

* 返回示例

 ```json
   {
  "code": 200,
  "msg": "成功",
  "data": {
    "link": "https://cashier-v2.chippaytest.com/#/buy?orderNo=12511234561_16179362628822&phone=189****0077",
    "orderNo": "12511234561_16179362628822"
  },
  "success": true
}
 ```


##### <a name="3a1">快捷批量卖单</a>

* 请求url

  > {{domain}}/cola/apiOpen/merchantOrder/apiBatchSellOrder
* 请求格式

  > <span style = "color: red;font-size:20px;font-weight:bold">httppost+json</span>
* 请求头部

> <span style = "color: red;font-size:20px;font-weight:bold">Content-Type:application/json   </span>

* 请求参数

|参数|数组参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|--|
|companyId|-|商户ID|long|20|Y| |
|payCoinSign|-|法币币别，须传小写英文（cny，vnd）|string||N|cny|
|asyncUrl|-|回调地址|string||Y| |
|sign|-|签名|string||Y| |
|apiRecordList||订单数组|object||Y| |
||username|用户名|string||Y| |
||areaCode|区号|string||N| |
||phone|手机号|string||Y| |
||email|用户邮箱，只支持当payCoinSign为`vnd`时传输，phone或者email需择一传输|string||N| |
||payCardNo|银行卡号|string||Y| |
||payCardBank|开户银行，当payCoinSign为`vnd`时需准确填入银行名称，参考<a href="#vnd_bank_area">vnd区银行名称</a>|string||Y| |
||payCardBranch|开户支行|string||N| |
||companyOrderNum|商户订单号|string||Y| |
||coinAmount|USDT下单数字货币数量(coinAmount和 total 两个字段二选一，当两个字段都填写的时候，优先处理total)coinAmount参数换算后的法币金额若不为整数，将无条件进位为整数显示于收银台|BigDecimal||Y| |
||total|用户付款的法币总金额 |BigDecimal||N|&nbsp; |



* 待签名请求参数

 ````json
   {
  "payCoinSign":"cny",
  "companyId": "12511234561",
  "asyncUrl": "199.99.99.99",
  "apiRecordList": [{
    "username": "李四",
    "areaCode": "86",
    "phone": "18320926666",
    "payCardNo": "622848202009358999",
    "payCardBank": "中国银行",
    "payCardBranch": "上海",
    "companyOrderNum": "BNB999999999",
    "coinAmount": "100",
    "total": "200"
  }, {
    "username": "周五",
    "areaCode": "86",
    "phone": "18320928888",
    "payCardNo": "622848202009358999",
    "payCardBank": "中国银行",
    "payCardBranch": "深圳",
    "companyOrderNum": "BNB888888888",
    "coinAmount": "100"
  }, {
    "username": "吴六",
    "areaCode": "86",
    "phone": "18320929999",
    "payCardNo": "622848202009358999",
    "payCardBank": "中国银行",
    "payCardBranch": "香港",
    "companyOrderNum": "BNB99999888",
    "total": "100"
  }]
}
 ````

* 生成【待签名的字符串】
  ````xml
     apiRecordList=[{areaCode=86,coinAmount=100,companyOrderNum=BNB999999999,payCardBank=中国银行,payCardBranch=上海,payCardNo=622848202009358999,phone=18320926666,total=200,username=李四},{areaCode=86,coinAmount=100,companyOrderNum=BNB888888888,payCardBank=中国银行,payCardBranch=深圳,payCardNo=622848202009358999,phone=18320928888,username=周五},{areaCode=86,companyOrderNum=BNB99999888,payCardBank=中国银行,payCardBranch=香港,payCardNo=622848202009358999,phone=18320929999,total=100,username=吴六}]&asyncUrl=199.99.99.99&companyId=12511234561&payCoinSign=cny
  ````

* 使用SHA256withRSA算法. 生成的私钥对排序后的字符串进行签名，签名后的结果赋值于参数sign

````json
{
  "sign":"c1sx47fjg6rbqICGCoiuNVUkJJbYdqR5ehSxVSALql7FBVC/HtmWE1MntSKxQmK7aVNSJ7ZGLD7EHmZWHRVKZVwMI5vyfTAFgT6XMiJFDdS8B6skh3D1an2OhbCOkm/+GhB8Btruqebfi9K3n8ud6C3ioHCCSHIE9PTaSyu3h4g=",
  "payCoinSign":"cny",
  "companyId": "12511234561",
  "asyncUrl": "199.99.99.99",
  "apiRecordList": [{
    "username": "李四",
    "areaCode": "86",
    "phone": "18320926666",
    "payCardNo": "622848202009358999",
    "payCardBank": "中国银行",
    "payCardBranch": "上海",
    "companyOrderNum": "BNB999999999",
    "coinAmount": "100",
    "total": "200"
  }, {
    "username": "周五",
    "areaCode": "86",
    "phone": "18320928888",
    "payCardNo": "622848202009358999",
    "payCardBank": "中国银行",
    "payCardBranch": "深圳",
    "companyOrderNum": "BNB888888888",
    "coinAmount": "100"
  }, {
    "username": "吴六",
    "areaCode": "86",
    "phone": "18320929999",
    "payCardNo": "622848202009358999",
    "payCardBank": "中国银行",
    "payCardBranch": "香港",
    "companyOrderNum": "BNB99999888",
    "total": "100"
  }]
}
````
* 返回示例

  ````json
     {
        "code": 200,
        "msg": "成功",
        "data": null,
        "success": true
     }

  ````


#### <a name="3a">回调通知</a>

#####  回调验签公钥
> 使用回调公钥对ChipPay的回调进行验签

>测试环境
```txt
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCBjEj/DylMlxONCDkkZQxh+woiD4goiG5WM+Ju3V2hmJpjpGCqXDClf4TLTymZMyM4GF0JL1euwgaacZ/pcxVHXpyGg8UstFUPrw7SStYURk4CLIWjuCrzZwALLGFQFNxQGFsXCR1WwpE08byw0asTWTL4VB9YlYRiV8huB/gcqwIDAQAB
```
>正式环境
```txt 
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCPV284s9ydOOZGCUFIw1/0d2mtC2XX8Y6oFVYtBqhno5hhI9qzUOZ+U2Raqfu8JAcbxqXaVX7MUjxlSWSHOJ5X2yiQ5GsNgNvpTKlOnv37iC/iJdajaqzyxC1mDfW+M8X6IQsWyvoRkNZ8V8WfmCPtFL7viGPbE9XKZfZApZRgXwIDAQAB
```

* 通知地址来自商户
  订单完成后，信息将返回给商户。格式与响应相同
  * 这个回调地址是由「API快捷买卖」中的商家:参数[asyncUrl]提供的
  * ChipPay接收到快捷订单信息并进行处理后，ChipPay服务器将异步请求此接口。
  *  商家的这个接口必须返回一个json格式。如果code == 200的json格式，ChipPay会认为请求成功，不会再请求
  *  错误响应(比如:回调响应`404,502`等 归类为错误回调, 系统默认回调`2`次)
  *  失败回调(比如:回调响应延迟或者其他网页信息等.系统默认回调`5`次)
  *  <span style = "color: red">异常处理订单将会有两次回调，交易状态分别为「交易失败」及「交易成功」，详情请参考常见问题</span>【 <a href='#8'>关于异常订单</a>】

*  回调参数

|参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|
|companyOrderNum|商户订单号|string||Y| |
|otcOrderNum|ChipPay平台订单号|  string||Y| |
|coinAmount|订单币种数量|bigDecimal to string type||N| |
|coinSign|数字货币标识：usdt|string||Y| |
|orderType|快捷买卖方向(1.Express buy, 2.Express sell)|int to string type||Y| |
|tradeStatus|交易状态(0:交易失败， 1:交易成功 ，2：快捷批量卖单生成失败)|string||Y| |
|tradeOrderTime|订单交易时间 (北京时间)|string||N| |
|unitPrice|数字货币单价|bigDecimal to string type||N| |
|total|用户付款的法币实际到账金额|bigDecimal to string type||N| |
|successAmount|数字货币到账数量|bigDecimal to string type||N| |
|sign|参数签名|string||Y| &nbsp;  |

* 回调参数示例

````json  
  {
    "coinAmount":"100.00",
    "coinSign":"usdt",
    "companyOrderNum":"hafagafasfadfwerwer32",
    "otcOrderNum":"12511234561_1592731510161",
    "orderType":"1",
    "tradeStatus":"1",
    "tradeOrderTime":"2020-07-15 18:46:04",
    "unitPrice":"7.01",
    "total":"701",
    "successAmount":"100",
    "sign":"TYzuqA6gkS9wAzLA+MTx5/6TF89cH3JsTkC67WDi8u8NlGp5DuCLIltRzO/c8aG9h54dLYyxtFrrTsB9qgWmpXNMXqPX1PNeIoLS4D/l/jh6yIohTwpKvp5Giq7UffnMgyt6ha2OGc8kam6rilX+ZLi7CP6mcx/DYkO46c8b8q0="
}
````

* 响应参数示例


````json
 {
  "code": 200,
  "msg": "success",
  "data": {
    "otcOrderNum": "ChipPay订单号",
    "companyOrderNum": "商户订单号"
  },
  "success":true
}
````

### <a name="4">查询接口 </a>
##### <a name="4a">查询订单信息</a>

* 请求url

  > {{domain}}/cola/apiOpen/common/getOrderInfo
* 请求格式

  > <span style = "color: red;font-size:20px;font-weight:bold">httppost+json</span>
* 请求头部

  > <span style = "color: red;font-size:20px;font-weight:bold">Content-Type:application/json</span>
* 请求参数

|参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|
|companyId|商户id|long||Y| |
|companyOrderNum|商户订单号|string||Y| |
|sign|参数签名|string||Y| &nbsp;|


* 请求参数示例

 ````json
  {
  "companyId":"12511234561",
  "companyOrderNum":"NS1592747733465",
  "sign":"JRyOrvHsF1BnHcK6j1VgM9JYLOud9biyYP+8cnEU2GcyB5PoF6mJlIYrjCxDJDs67qdRJi1tIs6ePnYycsppKbVzcwJ1GXKgRvXDMIjx8DBPvNSlGxC+cOaHxBzrpcIfFg5NGcVAlB7BoIWc+2tUtcxQGTPSBkTBlteY5RotAHU="
} 
 ````

* 响应参数示例

  ````json
  {
      "code": 200,
      "msg": "成功",
      "data": {
          "companyOrderNum": "NS1592747733465",
          "otcOrderNum": "12511234561_1592747800002",
          "coinSign": "USDT",
          "payCoinSign": "cny",
          "orderType": "sell",
          "orderCoinAmount": 28.49003,
          "unitPrice": 7.02,
          "orderPaySumPrice": 200,
          "orderStatus": 3,//1、成功 2、进行中 3、失败
          "createTime": "2021-01-18 15:26:32",
          "errorOrderAmount": 18,
          "errorOrderTotal": 126.54
      },
      "success": true
  }

  ````

##### <a name="4b">查询用户交易信息</a>

* 请求url

  > {{domain}}/cola/apiOpen/common/getUserOrderInfo
* 请求格式

> httppost+json

* 请求头部

  > Content-Type:application/json
* 请求参数

|参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|
|companyId|商户id|long||Y| |
|areaCode|国际区号|string||Y| |
|phone|手机号|string||Y| |
|email|用户邮箱，只支持当payCoinSign为`vnd`时传输，phone或者email需择一传输|string||N| |
|sign|参数签名|string ||Y|&nbsp;|


* 请求参数示例

 ````json
{
  "companyId":"12511234561",
  "areaCode":"86",
  "phone":"18320925789",
  "sign":"ciq0iy+wUO2qjvj06TaRSR4/WStDYAUbdZmfp3iSMzAUS0ZBSkhmvdxaCo9EoiHOtAIlxASCc0e0EyglQ7XjVetcDfBVwyRXfG0iw64jABA5Dzqix/hV4zSXp0oJqAEdEQuCsLReY7DRrOVn8JG2oVCxt2hFn9TwgN7w99YkDpk="
} 
 ````

* 响应参数示例

  ````json
  {
    "code": 200,  
    "msg": "成功",
    "data": {
        "processingBuyOrder": "NB000000003", //	进行中的快捷买单（商户订单号）
        "processingSellOrder": "NB000000001", //	进行中的快捷卖单（商户订单号）
        "cancelNum": 0, //	用户取消次数（今天内）
        "freezeStatus":"1" //用户账户状态（2：正常，1：锁定）
    },
     "success": true,
  }
  ````


##### <a name="4c">查询价格</a>

* 请求url

  > {{domain}}/cola/quotePriceBusiness/priceConfig/getPrice
* 请求格式

  > <span style = "color: red;font-size:20px;font-weight:bold">get</span>
* 请求参数

|参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|
|coinType|币种(cnyusdt, vndusdt)|string||Y|&nbsp;|


* 请求参数示例

  ```xml
  {{domain}}/cola/quotePriceBusiness/priceConfig/getPrice?coinType=cnyusdt
  ```

* 响应参数示例

 ```json     
  {
    "code": 200,
    "msg": "成功",
    "data": {
        "id": 2,
        "coinType": "usdt",
        "buyPrice": 6.98,
        "sellPrice": 0
    },
    "success": true
  }
 ```


##### <a name="4d">查询账户余额</a>

* 请求url

  > {{domain}}/cola/apiOpen/merchant/getMerchantAmount
* 请求格式

  > <span style = "color: red;font-size:20px;font-weight:bold">httppost+json</span>
* 请求参数

|参数|释义|类型|长度|是否必须|默认值|
|--|--|--|--|--|--|
|companyId|商户ID|long||Y| |
|accountType|账户类型 不填默认为5 (账户类型: 1:BTC,2:ETH,5:USDT)|string||N| |
|sign|签名(规则详见签名规则文档)|string||Y| &nbsp;|


* 请求参数示例

```JSON
   {
  "companyId": "12511234561",
  "accountType": "5",
  "sign": "Vjkhrmep86GL03EHs4YAAJE0oi9Cfm4EWTIdIEVe1btIfZJymSMu01bFFamMVv3fAwfB9QBysl3L1RPn0USCmCW9wfGeP12Mn1hKPqM0iCJskZ3ZiYDCaO02T3vgfSdI+XC5urI+XuDoDxTpfX3NDHSI+xtV7KBi/ORczLasweQ="
} 
```

* 响应参数示例

 ```json     
  {
    "code": 200,
    "msg": "成功",
    "data": {
        "assets": "10000000.20101823"
    },
    "success": true
  }
 ```
<span id ="vnd_bank_area"></span>

### <a name="4f">vnd区银行名称</a>

|银行名称|
|--|
|Techcombank|
|Sacombank|
|Vietcombank|
|Asia Commercial Bank|
|DongA Bank|
|Vietin Bank|
|Bank for Investment and Development of Vietnam|
|Eximbank Vietnam|
|Bank Central Asia|
|Mandiri Bank|
|CIMB Niaga|
|Vietnam Prosperity Joint-Stock Commercial Bank|
|Military Commercial Joint Stock Bank|
|Tien Phong Commercial|
|AGRI BANK|
|Lien Viet Post Joint Stock Commercial Bank|
|Saigon – Hanoi Commercial Joint Stock Bank|
|Vietnam Public Joint-stock Commercial Bank|
|SAIGON BANK|
|Southeast Asia Commercial Joint Stock Bank|
|An Binh Commercial Join Stock Bank|
|Bac A Bank|
|Viet Capital Bank|
|Vietnam Maritime Commercial Joint Stock Bank|
|Kien Long Commercial Joint Stock Bank|
|Nam A Commercial Joint Stock Bank|
|National Citizen Commercial Joint Stock Bank|
|HD Bank|
|Orient Commercial Joint Stock Bank|
|Vietnam International Commercial Joint Stock Bank|
|Sai Gon Joint Stock Commercial Bank|
|Saigon Bank for Industry and Trade|
|VietABank|
|BAOVIET Bank|
|Vietnam Thuong Tin Commercial Joint Stock Bank|
|Petrolimex Group Commercial Joint Stock Bank|
|Ocean Bank|
|GP Bank|
|Shinhan Bank|
|First Commercial Bank|
|Australia and New Zealand Banking Group Limited|
|Standard Chartered Bank|
|United Overseas Bank|
|Woori Bank|
|HSBC BANK|
|Vietnam-Russia Joint Venture Bank|
|Vietnam Development Bank|

### <a name="5">状态码</a>

|code|msg|
|--|--|
|200|成功|
|6001|商户不存在|
|6002|用户注册失败|
|6003|创建订单失败|
|6004|增加商户回调失败|
|6005|账变失败|
|6006|保存用户取消记录失败|
|6007|更新订单信息失败|
|6008|保存解冻记录失败|
|6009|解冻记录不存在|
|6010|查询商户信息失败|
|6011|查询交易员信息失败|
|6012|用户未绑定支付方式|
|6013|卖单支付方式不能为空|
|6015|验签失败|
|6016|用户余额不足|
|6017|订单已存在|
|6019|超过下单限制|
|6020|未选到合适交易员|
|6021|下单通道关闭|
|6022|资金密码错误|
|6023|订单不存在|
|6024|用户不存在|
|6025|报价异常|
|6026|买价关闭|
|6027|卖价关闭|
|6028|查询分页失败订单错误|
|6029|退款订单已存在|
|6030|退款订单不存在|
|6031|订单已更新，请刷新页面重试|
|6032|新增退款订单失败|
|6033|退款订单信息更新失败|
|6034|异常订单已存在|
|6035|异常订单申请失败|
|6036|异常订单不存在|
|6037|异常订单更新失败|
|6038|查询订单信息错误|
|6039|退款订单分页查询错误|
|6040|不能重新派单给原来交易员|
|6041|未找到交易员|
|6042|重新派单错误|
|6043|资产不足|
|6044|存在未完成订单|
|6045|订单匹配失败|
|6046|用户未实名|
|6047|获取价格错误|
|6048|新增存币订单详情错误|
|6049|该存币订单详情不存在|
|6050|更新存币订单详情错误|
|6051|存币订单详情入账错误|
|6052|查询存币订单价格错误|
|6053|新增兑换订单错误|
|6054|新增订单回调错误|
|6055|未找到该订单|
|6056|不允许多次重新派单|
|6057|交易员未开启接单|
|6058|订单类型错误|
|6059|报价关闭|
|6060|未获取redis锁|
|6061|没有重新派单|
|6062|订单金额小于最小限制|
|6063|查询用户订单信息异常|
|6064|自定义价格超过限制|
|6065|账户生成失败|
|6066|调用兑换服务异常|
|6067|没有此订单链接|
|6068|参数不能为空，请核对手机号或用户名|
|6069|手机号输入格式不正确|
|6070|发送验证码失败|
|6071|验证码不正确|
|6072|没有快捷卖单权限|
|6073|没有快捷买单权限|
|6074|金额小数点不能大于两位！|
|6075|币数量小数点不能大于四位！|
|6076|金额或数量必须传其中一个！|
|6077|用户取消失败！|
|6078|交易员确认失败！|
|6079|卖单取消失败！|
|6080|申请异常订单失败！|
|6081|由于您订单操作频繁，为了你的账户安全，系统已先暂停您的下单权限，如需开启请联系客服|
|6082|商户订单号不存在！|
|6083|对应的支付渠道未开启|
|6084|对应的支付币种未开启|
|6085|平台账户账变异常|
|6086|回退账户资产失败|
|6087|没有下单权限|
|6088|用户取消次数达到限制！|
|6089|异常订单已审核,请刷新页面|
|6090|解除取消次数|
|6091|订单金额大于最大限制|
|6092|该订单已派单|
|6093|请输入正确的用户姓名格式|
|6094|订单超时|
|6095|当前请求用户过多,稍后重试！|
|6096|获取支付方式限额失败|
|6097|审核异常订单失败|
|6098|交易员余额不足，请充值！|
|6099|接单总开关为空|
|6100|接单白名单为空|
|6101|接单总开关查询失败|
|6102|下单通道关闭，系统正在维护中...|
|6103|当日卖单剩余额度不足！|
|6104|超过单笔限额|
|6105|该商户为绑定用户|
|6106|查询用户取消时长失败|
|6107|查询订单回调信息失败|
|6108|该订单在执行其它操作， 稍后重试！|
|6109|银行卡号只能输入数字和空格！|
|6113|订单时间不在有效期内,请重新下单.|
|6114|无权访问|
|8002|金额必须为整数!|
|8005|请正确输入银行名称|
|8033|无法使用Email下单|
|8034|邮箱格式错误|
|8035|邮箱或者手机号需至少填写一个|
|8036|不能同时使用邮箱和手机号|


### <a name="6">交易规则 & 常见问题</a>
<span name = 'trading_rules'></span>
##### 交易规则:

1. 平台根据 `companyId`  和 `phone`，判断用户是否有未完成订单（用户距上次发起交易已超过一分钟且按下「我已完成汇款」按钮，订单未取消前再次下新单会进入未完成订单），同一用户要完成上一笔订单才能生成新订单;
2. 平台根据 `companyId` 和 `companyOrderNum`，判断是否相同快捷买单;
3. 请求错误会进入错误页面（显示错误信息），正确则进入订单页面。
4. 测试环境`companyId`由Chippay技术团队提供，订单号`companyOrderNum`要求最好加上自己的前缀，避免订单重复。

##### 常见问题：
- <a name='7'>关于 `displayUnitPrice`</a> 字段
  假设ChipPay的USDT报价是6.86CNY/USDT，商户平台欲展示价格是7.00CNY/USDT， 该如何实现？
  在调用接口买币时可通过参数：displayUnitPrice传入客户想要展示的价格：7.00CNY/USDT
  （1）若只传了数量，即：假如数量传了100(USDT) ，则展示数量为：100*6.86/7=98(USDT)，订单实际的成交数量为100 (USDT)
  （2）若传了金额，即：加入金额传了686(CNY)，则展示数量为：686/7=98(USDT),订单实际成交的数量为：686/6.      86=100 (USDT)
  (3) 若同时传了数量和价格，则优先取价格，算法同（2）
  注意：改变后的价格，上下浮动不能超过平台汇率的百分之三 否则无法下单

- <a name='8'>关于异常订单</a>
  「快捷买单」用户未在订单时间内打款、银行延迟到帐、或实付金额与订单金额不符皆会导致订单取消。当交易员在确认收到款后，可透过异常处理流程处理订单，让币入帐
  - 异常订单回调说明
    假设汇率为7 CNY =1 USDT，订单金额为700CNY，用户实付1400CNY

    ChipPay对异常订单会有两次回调，
    第一次回调订单状态为tradeStatus = 0【交易失败】，`coinAmount`为原始订单下单数量即100，`successAmount`为0, `total`为700 (作为参考)；
    第二次回调订单状态为tradeStatus = 1【交易成功】，`coinAmount`为原始订单下单数量即100，`successAmount`为200, `total`为1400。

-  <a name='9'>关于`coinAmount` 参数</a>
   当买币请求参数传coinAmount (USDT数量为准)时，若换算后的CNY金额包含小数点，则系统将自动进位为整数并生成订单，客户需要多支付部分差额。

  - 举例: 当平台汇率为6.51CNY=1USDT时，coinAmount传20，6.51*20=130.2，收银台将显示131CNY金额予客户进行付款。当有以上情况时，我方将回调Total为131、coinAmount为20且successAmount为20给贵方

#### <a name='7'>在线演示&示例代码</a>

- <a href ="https://demo.chippaytest.com/#/">在线演示</a>
- <a href="https://github.com/chippaydev/chippay-sdk-java">Java SDK</a>
- <a href="https://github.com/chippaydev/chippay-sdk-php">PHP SDK</a>
- <a href="https://github.com/chippaydev/chippay-demo-java">Java Demo</a>
