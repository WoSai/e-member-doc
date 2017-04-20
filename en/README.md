# Electronic Member (E-Member)

标签（空格分隔）： E-Member

---

[TOC]

##1.Introduction
This document is used to illustrate how to connect the E-member system and get data of member.
 > * Step 1: Contact ShouQianBa-Stanley(13917862326) for technical support.
 > * Step 2: Apply for appid and secret from ShouQianBa. The secret is valid for one month. So you should update the secret periodically. (to chapter 2.1)
 > * Step 3: Develop a function to receive information of member. (to chapter 3)
 > * Step 4: Develop a function to query information of member. (to chapter 4)


##1.1 The process of business interaction
```seq
Title:The Sequence Diagram of scan QR code to get membership
Customer->Channel(Wechat/Alipay):Scan QR code
Channel(Wechat/Alipay)->Customer:Display page of apply for membership
Customer->Channel(Wechat/Alipay):Fill in the form
Channel(Wechat/Alipay)->ISV(ShouQianBa):Send an apply for membership request
ISV(ShouQianBa)-->Merchant:Send an asynchronous request of member registration(function3)
ISV(ShouQianBa)->Channel(Wechat/Alipay):Return response
Channel(Wechat/Alipay)->Customer:Display the membership card
Customer->Channel(Wechat/Alipay):Query base information of member
Channel(Wechat/Alipay)->Customer:Display base information of member
Customer->ISV(ShouQianBa):Query member points/grade/special offers
ISV(ShouQianBa)->Merchant:Send a request to get information of member
Merchant->ISV(ShouQianBa):Return the information
ISV(ShouQianBa)->Customer:Display member points/grade/special offers
```

##2.Signature verification
In the Internet environment, use the interfaces you should do signature verification.

###2.1 Update secret
    Apply for appid and secret from ShouQianBa. The secret is valid for one month. So you should update the secret periodically.
 - api url:https://m.wosai.cn/api/sign/v2
 - request type: post
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|appid|Unique identity from ISV|varchar(20)|Y||
|expired|the useful-life of secret|bigint|Y|unit in seconds|
|sign|MD5 signature|varchar|Y|to chapter 2.2|


 - example:
 
```javascript
{
    "appid":"2200000001",
    "expired":"86400",
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq"
}
```


 - return parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|code|response code|varchar(6)|Y|200 success,otherwise exception|
|message|error message|varchar(6)|N|exist when code is not 200|
|data|new secret|varchar(32)|N|exist when code is 200|



 - example:
 
```javascript
{
    "code":"200",
    "data":{
        "appid":"2200000001",
        "expired":"86400",
        "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq"
    }
}
```

###2.2 How to make a MD5 signature
    MD5 signature is used to make api calling more secure, using secret and parameters to make a MD5 signature, see the follow steps:
 - Signature introduction:

    All parameters about api calling and QR code generating are used to make a signature, for example parameters include: appid / storied / channel / notifyTime / id, and secret=34719280830192 , appid=2200000001, notifyTime=1468780992, channel=alipay, storeId=5308, id=61028309128301298, the follow these steps to make a signature:
```
Step 1: store secret and parameters into an array=['2200000001=appid','1468780992=notifyTime','alipay=channel','61028309128301298=id','5308=storeId','34719280830192=secret'] 
Step 2: sort this array to get a sortArray=['1468780992=notifyTime','2200000001=appid','34719280830192=secret','5308=storeId','61028309128301298=id','alipay=channel'] 
Step 3: using '&' to connect all these parameters then get a string source='1468780992=notifyTime&2200000001=appid&34719280830192=secret&5308=storeId&61028309128301298=id&alipay=channel'
Step 4: using MD5 to get a signature, and make the signature combined with capital chars, sign=Upper(MD5(source))
```

##3.Send information of membership card
    Customers get/activate/update their information, E-Member will keep sending this information to seller and the frequency of send information is real-time/2min/10min/1hour/6hours/12hours/24hours until get the response "SUCCESS" from seller.

 - request type: post
 - parameters format: json
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|sign|MD5 signature|varchar(128)|Y|to chapter 2.2|
|notifyTime|time of notify|timestamp|Y|1468780992|
|notifyType|type of notify|varchar(10)|Y|active/update|
|channel|channel|varchar(10)|Y|activate  member must fill in, otherwise not. wechat/alipay|
|id|channel identification|varchar(32)|Y||
|storeId|store identification|varchar(32)|Y|activate  member must fill in, otherwise not|
|name|nickname|varchar(20)|Y|
|birthday|birthday|varchar(10)|Y|1977-04-18|
|mobile|cell-phone number|varchar(13)|Y||
|email|email|varchar(40)|N|
|gender|gender|Integer(1)|N|1:male/2:female/0:unknown|
|headImg|path of picture|varchar(200)|N|
|country|country|varchar(20)|N|
|province|province|varchar(20)|N|
|city|city|varchar(20)|N|
|address|address|varchar(200)|N|
|industry|industry|varchar(40)|N|
|memberId|member id|varchar(40)|N|Merchant can know customer have or not the membership according to customer’s old member id and cell-phone number.|


 - example:
 
```javascript
{
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq",
    "notifyTime":"1468780992",
    "notifyType":"active",
    "channel":"wechat",
    "id":"691822b1cab64c879feed51045821ab1",
    "storeId":"da6692c0c1f04f0986d9bf687b7d9908",
    "name":"Tom",
    "birthday":"1977-04-18",
    "mobile":"13112121212",
    "email":"dakang@rmmy.com",
    "gender":"1",
    "headImg":"https://m.wosai.cn/resource/img/dakangshuji/200",
    "country":"China",
    "province":"ZheJiang",
    "city":"Shanghai",
    "address":"XX street XX number",
    "industry":"IT",
    "memberId":"f8a3dae64f74431dbcc68216af82bdb7"
}
```


 - return:



 - example:
 
```javascript
SUCCESS
```

##4.Get information of membership card according to card identification
    Customers can get their base information in the page of membership card. And can get all information from this function of seller’s development. The information such as member points, grade, special offers.


 - request type: post
 - parameters format: json
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|sign|MD5 signature|varchar(128)|Y|to chapter 2.2|
|id|channel identification|varchar(32)|Y||


 - example:
 
```javascript
{
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq",
    "id":"691822b1cab64c879feed51045821ab1"
}
```


 - return parameters:
|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|code|response code|varchar(6)Y|||
|message|response message|varchar(60)|N||
|data|service entity|{}||
| - id|channel identification|varchar(32)|Y||
| - storeId|store identification|varchar(32)|Y|activate  member must fill in, otherwise not|
| - name|nickname|varchar(20)|Y|
| - birthday|birthday|varchar(10)|Y|1977-04-18|
| - mobile|cell-phone number|varchar(13)|Y||
| - email|email|varchar(40)|N|
| - gender|gender|Integer(1)|N|1:male/2:female/0:unknown|
| - headImg|path of picture|varchar(200)|N|
| - country|country|varchar(20)|N|
| - province|province|varchar(20)|N|
| - city|city|varchar(20)|N|
| - address|address|varchar(200)|N|
| - industry|industry|varchar(40)|N|
| - card|card information|[{}]|Y||
| -  - no|card number|varchar(12)|Y||
| -  - score|card points|bigint|Y||
| -  - level|card grade|varchar(10)|Y|||
| -  - expire|the useful-life of card|varchar(10)|Y|||
| -  - phone|phone|varchar(20)|N||
| -  - rights|rights|[]|N|
| -  - instructions|instructions|[]|N||


 - example:
 
```javascript
{
    "code":"200",
    "message":"SUCCESS",
    "data":{
        "id":"691822b1cab64c879feed51045821ab1",
        "name":"Tom",
        "birthday":"1977-04-18",
        "mobile":"13112121212",
        "email":"dakang@rmmy.com",
        "gender":"1",
        "headImg":"https://m.wosai.cn/resource/img/dakangshuji/200",
        "country":"China",
        "province":"ZheJiang",
        "city":"Shanghai",
        "address":"XX street XX number",
        "industry":"IT",
        "card":[{
            "no":"3000 0000 0001",
            "score":"2000",
            "level":"gold",
            "expire":"lifetime",
            "phone":"400-800800",
            "rights":["shopping discount","free to park"],
            "instructions":["card use by youself","card can not share with other people","H&M reserve the right of final interpretation"]
        }]
    }
}
```






