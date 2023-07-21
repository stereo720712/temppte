# Express transaction API


### <a name="1">API specification </a>

Merchants can use the test server for testing. It is recommended that the live environment be switched after the test environment is docked. If you encounter a problem, please check the documentation for common problems first. If you still can’t solve it, you can consult related technical personnel.

- Test environment (<span style = "color: red"> Please do not make any payments to the orders in the test environment.</span>)
> Interface domain name: `https://open-v2.chippaytest.com/`</br>


- Live environment
> Interface domain name: `https://open-v2.chippay.com/` </br>


<br>

#### Interface assembly
<span style = "color: red"> interface = interface domain name + interface suffix 


> Test environment: `https://open-v2.chippaytest.com/cola/apiOpen/addOrder` </br>
Live  environment: `https://open-v2.chippay.com/cola/apiOpen/addOrder` </br>

### <a name="2"> Key pair generation & Signature Rules</a>

> The signature is based on algorithm SHA256withRSA. The generated private key signs the request parameters (in abcd order, the second letter if the same initial letter is encountered, and so on), and the output is the signature (the string is assigned to the parameter sign).

```txt

  If the optional parameters are not passed to ChipPay, please do not participate in the signature. If you pass the data, whether it is empty or other data, you need to participate in the signature.


  The signature string is asked to be sorted according to the parameter name. (First compare the first letter of all parameter names, by Alphabetical order and if you encounter the same initial, compare the second letter, and so on).


```

* Asymmetric signature public and private key generation instructions
   * Use RSA to generate asymmetric public and private keys, the key strength is 1024 bits, and the export format is DER PKCS#8. Please refer to the SDK for specific generation methods.
   * The merchant keeps the private key. The private key needs to be kept properly and cannot be leaked to others. The public key is given to the platform
   * The merchant uses the private key to sign the request, and the platform uses the public key to verify the signature

-  Request example, sign the following parameters

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

2、Generate【to be signed string】

````java
      areaCode=86&asyncUrl=127.0.0.1:8090/v1/demo/return.php&coinAmount=20&coinSign=USDT&companyId=12511234561&companyOrderNum=NB1594174982945&idCardNum=430524143201097878&idCardType=1&kyc=2&orderPayChannel=3&orderTime=1594174982945&orderType=1&payCoinSign=cny&phone=18900000001&syncUrl=127.0.0.1:8090/v1/demo/return.php&total=200&username=范思哲

````

3、Use SHA256withRSA algorithm. The generated private key signs the sorted string, and the signed result is assigned to the parameter sign:

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

### <a name="3">Deal APIs </a>

##### <span style = "color: red">Important note</span>
* <span style = "color: red">Express buying & Express selling API (It is recommended to use Express batch selling API first for selling orders.): The interface returns the order submitted result in real-time, and one client could only generate one order in the processing period.

* <span style = "color: red">Express batch selling API: One client is allowed to send multiple selling orders at the same time, orders will be queued for processing. The successful submission of this batch does not mean that all orders can be successfully generated. If an order request is not generated (e.g. caused by the reason of insufficient merchant balance.), there will be a callback with tradeStatus=2 for the order! </span>
* <span style = "color: red">If the API does not respond or returns a time out error (e.g. error code 502, 504), please confirm if the order has been placed successfully after 1 minute by the Query order interface, or log in to the ChipPay merchant back office to verify if the order has been successfully generated to avoid inconsistent order processing between us.  </span>
* <span style = "color: red"> The order number cannot be repeated. please do not change the order number and send multiple requests repeatedly for one withdrawal application to avoid repeated payment.  </span>  


##### <a name="3a"> Express buying & Express selling API (It is recommended to use Express batch selling API first for selling orders.</a>

* Request url
   
   > {{domain}}/cola/apiOpen/addOrder
* Protocol and data format
   
   > <span style = "color: red;font-size:20px;font-weight:bold"> httppost+json</span>
* Request Headers   
   
   ><span style = "color: red;font-size:20px;font-weight:bold"> Content-Type:application/json   </span> 
* Request parameters

|Parameters|Paraphrase|Content Type|Length|Required(optional)|Default Value|
|--|--|--|--|--|--|
|companyId|Merchant Id|long|20|Y| |
|kyc|Real name authentication.e.g.Only support kyc=2|string||Y|2 |
|username|User name|string|Chinese: 2-15 digits;English: 2-35 digits|Y| |
|areaCode| International area code |string||N|86|
|phone|Phone number must be 8-11 digits|string||Y| |
|email|User email address, supported only if payCoinSign is `vnd` ,you’ll need to transfer either phone or email.|string||N| |
|orderType|Order type （1.buy 2.sell）|Integer||Y| |
|idCardType|Certificate type (1. ID card 2. passport 3. others).e.g. If no value is passed, the default value is 1.|Integer||N| |
|idCardNum|Identity Number|string||N| |
|payCardNo|Bank card number (required for express selling order)|string||N| |
|payCardBank|Payee’s bank name (required for express selling order). If payCoinSign is `vnd`, please fill in the bank name accurately refer to “<a href="#vnd_bank_area">The Full List of Bank in VND Area.</a>” |string||N| |
|payCardBranch|Account opening branch|string||N| |
|companyOrderNum|Merchant order number|string||Y| |
|coinSign|Coin sign（USDT）|string||Y| |
|payCoinSign|Payment currency, must be in lowercase letters (cny,vnd)|string||Y| |
|coinAmount|USDT quantity of the order. ( coinAmount and total are alternative parameters, if both are sent, total would take precedence. If you only send coinAmount, the converted fiat currency amount will be rounded up to the whole digit and displayed on the cashier page if it's originally not an integer.)|BigDecimal||Y| |
|total|Fiat currency amount of the order. (It could only be an integer when sent with an express buying order.)|BigDecimal||N| |
|orderPayChannel|If payCoinSign is cny, it supports 3. Bank Card Payment. If payCoinSign is vnd, it supports 1. MOMO 3. Bank Card Payment. (vnd express selling orders only support 3.bank card)|Integer||N|3|
|displayUnitPrice|Exchange rate displayed on the cashier page, please refer to  Dealing Rules & Common Questions|BigDecimal||N| |
|orderTime|Timestamp of the order (The current timestamp is used, and a difference of 5 minutes from the current time is considered invalid) |string||Y| |
|syncUrl|Synchronous callback url|string||Y| |
|asyncUrl|Asynchronous callback url|string||Y| |
|sign|Signature string|string||Y| &nbsp;|


* Request parameters example

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

 * Data response example
  
```json
   {
    "code": 200,
    "msg": "获取快捷订单订单成功",
    "data":{
        "link": "https://open.chippaytest.com/v2/buy/transition.html?orderNo=256547567563234&success=true&phone=176****7618&auth=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImp0aSI6IjYifQ.eyJqdGkiOiI2IiwiaWF0IjoxNTY4OTQ4Nzk2LCJleHAiOjE1Njg5NTA1OTZ9.Ba4VRAhiekIFOrh8Ryu56UlUHqNSOVmAiNv4EGITxcU",
        "orderNo": 256547567563234
    },
    "success": true
   }
   
```

##### <a name="3a1">Express batch selling API</a>

* Request url
   
   > {{domain}}/cola/apiOpen/merchantOrder/apiBatchSellOrder
* Protocol and data format
   
   > <span style = "color: red;font-size:20px;font-weight:bold">httppost+json</span>
* Request Headers
   
   > <span style = "color: red;font-size:20px;font-weight:bold">Content-Type:application/json   </span>
   
* Request parameters

|Parameters|Array parameter|Paraphrase|Content Type|Length|Required(optional)|Default Value|
|--|--|--|--|--|--|--|
|companyId|-|Merchant Id|long|20|Y| | 
|payCoinSign|-|Payment currency, must be in lowercase letters (cny,vnd)|string||N|cny
|asyncUrl|-|Asynchronous callback url|string||Y| |
|sign|-|Signature string|string||Y| |
|apiRecordList||Order array|object||Y| |
||username|User name |string||Y | |
||areaCode|International area code|string||N| |
||phone|Phone number must be 8-11 digits|string||Y| |
||email|User email address, supported only if payCoinSign is `vnd` ,you’ll need to transfer either phone or email.|string||N| |
||payCardNo|Bank card number|string||Y| |
||payCardBank|Account opening bank, If payCoinSign is `vnd`, please fill in the bank name accurately refer to “<a href="#vnd_bank_area">The Full List of Bank in VND Area.</a>”|string||Y| |
||payCardBranch|Acount opening branch|string||N| |
||companyOrderNum|Merchant order number|string||Y| |
||coinAmount|USDT quantity of the order. ( coinAmount and total are alternative parameters, and when both are filled in, total takes precedence)|BigDecimal||Y| |
||total|Fiat currency amount of the order. ( coinAmount and total are alternative parameters, and when both are filled in, total takes precedence) |BigDecimal||N| &nbsp; |


*  Request parameters example

 ```json
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
 ```
 * Generate【to be signed string】
  
  ```xml
      apiRecordList=[{areaCode=86,coinAmount=100,companyOrderNum=BNB999999999,payCardBank=中国银行,payCardBranch=上海,payCardNo=622848202009358999,phone=18320926666,total=200,username=李四},{areaCode=86,coinAmount=100,companyOrderNum=BNB888888888,payCardBank=中国银行,payCardBranch=深圳,payCardNo=622848202009358999,phone=18320928888,username=周五},{areaCode=86,companyOrderNum=BNB99999888,payCardBank=中国银行,payCardBranch=香港,payCardNo=622848202009358999,phone=18320929999,total=100,username=吴六}]&asyncUrl=199.99.99.99&companyId=12511234561&payCoinSign=cny
  ```

* The SHA256withRSA algorithm is used. The generated private key signs the sorted string, and the signed result is assigned to the parameter sign

```json
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
```

 * Data response example
  
   ```json
      {
         "code": 200,
         "msg": "success",
         "data": null,
         "success": true
      }
   
   ```



##### <a name="3b">Asynchronous callback notice （asyncUrl in dealing's api）</a>   


#####  Public key of the callbacks
> Use the public key to verify the callbacks sent from ChipPay.

>Test environment
```txt
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCBjEj/DylMlxONCDkkZQxh+woiD4goiG5WM+Ju3V2hmJpjpGCqXDClf4TLTymZMyM4GF0JL1euwgaacZ/pcxVHXpyGg8UstFUPrw7SStYURk4CLIWjuCrzZwALLGFQFNxQGFsXCR1WwpE08byw0asTWTL4VB9YlYRiV8huB/gcqwIDAQAB
```
>Live environment
```txt 
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCPV284s9ydOOZGCUFIw1/0d2mtC2XX8Y6oFVYtBqhno5hhI9qzUOZ+U2Raqfu8JAcbxqXaVX7MUjxlSWSHOJ5X2yiQ5GsNgNvpTKlOnv37iC/iJdajaqzyxC1mDfW+M8X6IQsWyvoRkNZ8V8WfmCPtFL7viGPbE9XKZfZApZRgXwIDAQAB
```


* Notify url was sent form the merchant, after the order completed, the information will be returned to the merchant. The format will be the same as response.
* This interface is provided by the merchant: parameter [`asyncUrl`] in the "Express buying & Express selling API.

    * After ChipPay receives the deposit order information and processes it, the ChipPay server will asynchronously request this interface.

    *  This interface of the merchant must return a json string. If code == 200 in json, ChipPay will consider the request successful and will not request it again.
    * Error response (for example: callback response '404,502' and so on are classified as error callback, and the system default callback '2' times)

      * Failed callbacks (for example, callback response delay or other web page information, etc.) Default system callback '5' times)
    * <span style = "color: red">There will be two callbacks for abnormal orders. The trading status is "Trade failed" and "Trade successful" respectively. Please refer to the FREQUENTLY asked Questions for details</span>【 <a href='#8'>about abnormal orders</a>】



*  Notify parameters

|Parameters|Paraphrase|Content Tyep|Length|Required(optional)|Default Value|
|--|--|--|--|--|--|
|companyOrderNum|Merchant order number|string||Y| |
|otcOrderNum|ChipPay order number|string||Y| | 
|coinAmount|Trade order quantity|bigDecimal to string type||N| |
|coinSign|Coin sign|string||Y| |
|orderType|Trade order direction (1.Express buy & 2.Express sell)|int to string type||Y| |
|tradeStatus|Trade status (0:failed ,1:success,2: Express batch selling order generated failed)|Integer||Y| |
|tradeOrderTime|Order time (Beijing time)|string||N| |
|unitPrice|Unit price of digital currency|bigDecimal to string type||N| |
|total|Fiat currency amount of the order.|bigDecimal to string type||N| |
|successAmount|Amount of digital currency received|bigDecimal to string type||N| |
|sign|Signature string|string||Y|&nbsp; |

* Notify parameters example
  
```json  
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
```

* Response parameters example
  

```json
 {
    "code": 200,
    "msg": "success",
    "data": {
        "otcOrderNum": "ChipPay order number",
        "companyOrderNum": "Merchant order number"
    },
    "success":true
  }
```


### <a name="4">Query APIs </a>
##### <a name="4a">Query order</a>

* Request url
   
   > {{domain}}/cola/apiOpen/common/getOrderInfo
* Protocol and data format
   
   > <span style = "color: red;font-size:21px;font-weight:bold" >httppost+json </span>
* Request Headers   
   
   > <span style = "color: red;font-size:21px;font-weight:bold" >Content-Type:application/json    </span>
* Request parameters
  

 |Parameters|Paraphrase|Content Type|Length|Required(optional)|Default Value|
 |--|--|--|--|--|--|
 |companyId|Merchant Id|long||Y| |
 |companyOrderNum|Merchant order number|string||Y| |
 |sign|Signature string|string||Y| &nbsp;|

* Request parameters example

 ```json
   {
      "companyId":"12511234561", 
      "companyOrderNum":"NS1592747733465",
      "sign":"JRyOrvHsF1BnHcK6j1VgM9JYLOud9biyYP+8cnEU2GcyB5PoF6mJlIYrjCxDJDs67qdRJi1tIs6ePnYycsppKbVzcwJ1GXKgRvXDMIjx8DBPvNSlGxC+cOaHxBzrpcIfFg5NGcVAlB7BoIWc+2tUtcxQGTPSBkTBlteY5RotAHU="	
	} 
 ```

* Data response example
  
  ```json
  {
      "success": true,
      "code": 0,
      "msg": "成功",
      "data": {
         "unitPrice": 7.03,
         "orderType": "sell",
         "companyOrderNum": "NS2062024076170",
         "coinSign": "USDT",
         "otcOrderNum": "12511234561_1592731510161",
         "createTime": "2020-06-21 16:25:10",
         "orderCoinAmount": 35.56188,
         "orderPaySumPrice": 250,
         "payCoinSign": "cny",
         "orderStatus": 3, // 1、success 2、processing 3、failed
         "errorOrderAmount": 18,
         "errorOrderTotal": 126.54
    }
  }
  ```
  
##### <a name="4b">Query user's dealing status</a> 

* Request url
   
   > {{domain}}/cola/apiOpen/common/getUserOrderInfo
* Protocol and data format
   
   ><span style = "color: red;font-size:21px;font-weight:bold" > httppost+json </span>
* Request Headers   
   
   > <span style = "color: red;font-size:21px;font-weight:bold" >Content-Type:application/json    </span>
* Request parameters

|Parameters|Paraphrase|Content Type|Length|Required(optional)|Default Value|
|--|--|--|--|--|--|
|companyId|Merchant Id|long||Y| |
|areaCode|International area code |string||Y| |
|phone|Phone number,must be 8-11 digits.|string||Y| |
|email|User email address, supported only if payCoinSign is `vnd` ,you’ll need to transfer either phone or email.|string| |N| |
|sign|Signature string |string ||Y|&nbsp; |

* Request parameters example

 ```json
   {
         "companyId":"12511234561",
         "areaCode":"86",
         "phone":"18320925789",
         "sign":"ciq0iy+wUO2qjvj06TaRSR4/WStDYAUbdZmfp3iSMzAUS0ZBSkhmvdxaCo9EoiHOtAIlxASCc0e0EyglQ7XjVetcDfBVwyRXfG0iw64jABA5Dzqix/hV4zSXp0oJqAEdEQuCsLReY7DRrOVn8JG2oVCxt2hFn9TwgN7w99YkDpk="
	} 
 ```

* Data response example
  
  ```json
  {
    "code": 200,  
    "msg": "成功",
    "data": {
        "processingBuyOrder": "NB000000003", //	1)Express buying order in progress(merchant order No.)
        "processingSellOrder": "NB000000001", // 2)Express selling order in progress(merchant order No.)
        "cancelNum": 0, //Number of user cancellations (today)
        "freezeStatus":"1" //User account status (2: normal, 1: blocked)
    },
     "success": true,
  }
  ```


##### <a name="4c">Query price</a> 

* Request url
   
   > {{domain}}/cola/quotePriceBusiness/priceConfig/getPrice
* Protocol and data format
   
   > <span style = "color: red;font-size:21px;font-weight:bold" >get </span>
* Request parameters
  

 |Parameters|Paraphrase|Content Type|Length|Required(optional)|Default Value|
 |--|--|--|--|--|--|
 |coinType|Coin Type(cnyusdt, vndusdt)|string||Y| &nbsp;|

* Request parameters example
  
  ````xml
  {{domain}}/cola/quotePriceBusiness/priceConfig/getPrice?coinType=cnyusdt
  ````

* Data response example
  
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


  ##### <a name="4d">Query account balance</a> 

* Request url
   
   > {{domain}}/cola/apiOpen/merchant/getMerchantAmount
* Protocol and data format
   
   > <span style = "color: red;font-size:20px;font-weight:bold">httppost+json </span>
* Request parameters
  

 |Parameters|Paraphrase|Content Type|Length|Required(optional)|Default Value|
 |--|--|--|--|--|--|
 |companyId|Merchant Id|long||Y| |
 |accountType|Account Type(Account Type: 1:BTC,2:ETH,5:USDT)|string||N|5|
 |sign|Signature string|string||Y| &nbsp;|


* Request parameters example
  
```JSON
   {
      "companyId": "12511234561",
      "accountType": "5",
      "sign": "Vjkhrmep86GL03EHs4YAAJE0oi9Cfm4EWTIdIEVe1btIfZJymSMu01bFFamMVv3fAwfB9QBysl3L1RPn0USCmCW9wfGeP12Mn1hKPqM0iCJskZ3ZiYDCaO02T3vgfSdI+XC5urI+XuDoDxTpfX3NDHSI+xtV7KBi/ORczLasweQ="
      
   }  
```

* Data response example
  
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

### <a name="4f">The Full List of Bank In VND Area</a> 
<span id='vnd_bank_area'></span>

|Bank name|
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



### <a name="5">Status Codes </a> 
   |code|msg|
   |--|--|
   |200|success|
   |6001|Merchant does not exist|
   |6002|User registration failed|
   |6003|Failed to create order|
   |6004|Add merchant callback failed|
   |6005|Change account detail failed|
   |6006|Save user cancellation record failed|
   |6007|Update order information failed|
   |6008|Failed to save unfreeze record|
   |6009|Unfreeze record does not exist|
   |6010|Query merchant information failed|
   |6011|Failed to query trader information|
   |6012|The user is not bound to the payment method|
   |6013|Sell ​​order payment method cannot be empty|
   |6015|API Sign verification failed|
   |6016|Insufficient user balance|
   |6017|The order already exists|
   |6019|Exceeds order number limit|
   |6020|No available traders|
   |6021|The payment channel is closed|
   |6022|Fund password error|
   |6023|Order does not exist|
   |6024|User does not exist|
   |6025|Quote abnormal|
   |6026|Bid close|
   |6027|Offer price closed|
   |6028|Query pagination failed, order error|
   |6029|Refund order already exists|
   |6030|Refund order does not exist|
   |6031|Order has been updated, please refresh the page and try again|
   |6032|Failed to add refund order|
   |6033|Failed to update refund order information|
   |6034|Abnormal order already exists|
   |6035|Exception order application failed|
   |6036|Abnormal order does not exist|
   |6037|Exception order update failed|
   |6038|Query order information error|
   |6039|Pagination query error for refund order|
   |6040|Cannot resend the order to the original trader|
   |6041|Trader not found|
   |6042|Reissue order error|
   |6043|Insufficient assets|
   |6044|The user has an outstanding order is processing|
   |6045|Order match failed|
   |6046|User not real-named|
   |6047|Get price error|
   |6048|The new deposit order details are wrong|
   |6049|The deposit order details do not exist|
   |6050|Update the deposit order details error|
   |6051|The deposit order details are entered incorrectly|
   |6052|Query deposit order price error|
   |6053|New redemption order error|
   |6054|Error in adding order callback|
   |6055|The order was not found|
   |6056|Multiple reissues are not allowed|
   |6057|Trader did not open the order|
   |6058|Wrong order type|
   |6059|Quote closed|
   |6060|Redis lock not acquired|
   |6061|No reissue|
   |6062|The order amount is less than the minimum limit|
   |6063|Abnormal in querying the user's order information|
   |6064|DisplayUnitPrice exceeds maximum limit|
   |6065|Account generation failed|
   |6066|Abnormal call to exchange service|
   |6067|No link to this order|
   |6068|The parameter cannot be empty, please check the phone number or user name|
   |6069|The phone number is entered incorrectly or the length of the phone number is not in the range of [8-11] digits|
   |6070|Failed to send verification code|
   |6071|Incorrect verification code|
   |6072|No selling order permission|
   |6073|No buying order permission|
   |6074|Number of digits after the decimal point in amount cannot be more than two!|
   |6075|Number of digits after the decimal point in quantity cannot be more than four! |
   |6076|Either the amount or quantity must be passed!|
   |6077|User cancellation failed! |
   |6078|Trader confirmation failed! |
   |6079|Selling order cancellation failed! |
   |6080|The application of exception handling failed! |
   |6081|Due to your frequent order operations, the system has temporarily suspended your trading authority for the security of your account. If you need to trade please contact customer service.|
   |6082|The merchant order number does not exist! |
   |6083|The corresponding payment method is closed|
   |6084|The corresponding payment currency is closed|
   |6085|Abnormal in changing platform account detail|
   |6086|Failed to refund account assets|
   |6087|No order permission|
   |6088|The number of user cancellations reached the limit! |
   |6089|The abnormal order has been approved, please refresh the page|
   |6090|Reset the number of user cancellations|
   |6091|The order amount is larger than the maximum limit|
   |6092|The order has been dispatched|
   |6093|Please enter the correct user name format|
   |6094|Order timeout|
   |6095|The system is busy, try again later!|
   |6096|Get payment method limit failed|
   |6097|Failed to review abnormal order|
   |6098|The trader's balance is insufficient, please recharge it! |
   |6099|The order delivery switch is empty|
   |6100|The whitelist for receiving order is empty|
   |6101|Failure in querying main switch for receiving orders|
   |6102|The order channel is closed and the system is under maintenance...|
   |6103|The remaining amount of the sell order of the day is insufficient! |
   |6104|Exceeding the single transaction limit|
   |6105|The merchant is a bound user|
   |6106|Failed to query user cancellation duration|
   |6107|Failed to query order callback information|
   |6108|The order is under other operation, try again later!|
   |6109|The bank card number can only be numbers!|
   |6113|The timestamp of the order is not in the valid period, please place a new order.|
   |6114|No permission to access|
   |6121|The order is already processeing, try again later.|
   |6135|Unknown merchant transaction method|
   |8002|Amount must be an integer|
   |8005|Please enter the correct bank name|
   |8010|The user has not performed signature verification|
   |8011|No balance in this account|
   |8017|Get selling order maximum quantity limit failed|
   |8018|Missing selling order data|
   |8019|Selling order quantity exceeds maximum limit|
   |8026|Failure in querying security deposit|
   |8027|Ｗrong payment currency|
   |8032|Merchant not in whitelist|
   |8033|Can not use email to add order|
   |8034|Email format error|
   |8035|Email or phone must be filled in at least one|
   |8036|Email and phone are both present|
   |51008|Abnormal in getting price , please check the currency type!|



### <a name="6">Dealing Rules & Common Questions </a> 

##### Dealing rules:

1. According to `companyId` and `phone`, the platform determines whether a user has an incompleted order, and the same user needs to complete the previous order to generate a new order. ( Note: If the user presses the button "Remittance Done" and the  last transaction has been initiated more than one minute, placing a new order before it is cancelled will be redirected to the incompleted order.)
2. Platform based on `companyId` and `companyOrderNum`, to determine whether the same order;
3. If meet an error while requesting, it will enter the error page ( and display an error message), otherwise, it will enter the order page directly.
4. CompanyId of test environment will be provided by Chippay technical team, it is recommended to add prefixes on `companyOrderNum` to avoid order duplication.


##### Common Questions
- <a name='7'> about `displayUnitPrice` Parameter</a>

   Assuming ChipPay rate is 6.86CNY/USDT, and you want your platform rate be 7.00CNY/USDT, how to implement it?
   When calling the interface ，you can be passed through the parameter displayUnitPrice: 7.00CNY/USDT
   (1) If only the coinAmount is transmitted, and the quantity is transmitted 100(USDT), then the display quantity is: 100*6.86/7=98(USDT), and the actual transaction quantity of the order is 100(USDT).
   (2) If the amount is transmitted, and the total is transmitted 686(CNY), then the display quantity is: 686/7=98(USDT), and the actual transaction quantity of orders is: 686/6.86=100 (USDT).
   (3) If the coinAmount and total  are  both transmitted at the same time, the total will be given priority, and the algorithm is the same as (2).
   Note: The fluctuation of the changed price shall not exceed 3% of the exchange rate of the platform, otherwise the order cannot be placed

- <a name='8'> about abnormal orders </a>
   Users who place "Express buying orders"(Fiat currency deposits) who fail to make the payment in time, the payment delayed, or the actual payment amount mismatch the order amount will result in the order cancellation. After the trader confirms receiving the payment, he will process the order through the exception handling process and let the deposit be credited to the merchant account.
  
     - [Description of the asynchronous callback of abnormal order ]
         Suppose the exchange rate is 7 CNY = 1 USDT, the order amount is 700 CNY, and the user actually pays 1400 CNY

         ChipPay will send twice callbacks for abnormal orders,
         The first callback, order status is [transaction failed],` coinAmount` is the original order quantity, which is 100,` successAmount` is 0, and `total `is 700;
         The second callback order status is [Successful transaction],` coinAmount` is the original order quantity, which is 100, `successAmount `is 200, and `total `is 1400.
 - <a name='9'>About the `CoinAmount` parameter</a>
   When the parameter of the API buying request is sent with coinAmount (USDT quantity), if the converted CNY amount contains a decimal point, our system will automatically round up it to the whole digit, the client needs to bear the difference in automatic rounding.
     - e.g. when the platform exchange rate is 6.51CNY=1USDT, and you sent requests with the coinAmount "20", it will display 131CNY on the cashier page for the client to make payment. (Though 6.51*20=130.2) The callback of this order will be sent with the Total "131", coinAmount "20", and successAmount "20".




### <a href='#7'>online demo &code example</a> 
  - <a href ="https://demo.chippaytest.com/#/">online demo</a>
  - <a href="https://github.com/chippaydev/chippay-sdk-java">Java SDK</a>
  - <a href="https://github.com/chippaydev/chippay-sdk-php">PHP SDK</a>
  - <a href="https://github.com/chippaydev/chippay-demo-java">Java Demo</a>
 