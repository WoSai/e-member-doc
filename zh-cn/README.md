# 电子会员

标签（空格分隔）： 未分类

---

[TOC]

##1.接入说明
本文档用于说明商户如何接入电子会员系统的会员数据。
 > * 第一步：联系收钱吧-stanley(13917862326)安排技术对接；
 > * 第二步：向收钱吧申请appid和密钥secret。注：为确保访问安全，秘钥有效期为一个月，商户应定期更新密钥(查看章节2.1)；
 > * 第三步：开发接收会员信息推送的接口（查看章节3）；
 > * 第四步：开发查询会员信息的接口（查看章节4）；


##1.1 业务交互流程
```seq
Title:扫码激活会员时序图
消费者->渠道(微信/支付宝):扫码激活
渠道(微信/支付宝)->消费者:显示会员激活界面
消费者->渠道(微信/支付宝):填写表单激活
渠道(微信/支付宝)->ISV(收钱吧):发起激活会员请求
ISV(收钱吧)-->商户:异步推送会员注册请求(接口3)
ISV(收钱吧)->渠道(微信/支付宝):返回请求结果
渠道(微信/支付宝)->消费者:显示会员卡
消费者->渠道(微信/支付宝):查看会员基本信息
渠道(微信/支付宝)->消费者:显示会员基本信息
消费者->ISV(收钱吧):查看会员积分/积分等级/会员权益信息
ISV(收钱吧)->商户:发起查询会员信息请求(接口4)
商户->ISV(收钱吧):返回会员信息
ISV(收钱吧)->消费者:显示会员积分/积分等级/会员权益信息
```

##2.签名说明
为保障数据安全，对暴露在互联网环境的接口的调用应当采用https协议通信，并对参数进行签名和验签。

###2.1 签到
    为了保证安全，由收钱吧提供给商户端的密钥应该定期更新，更新通过签到接口完成，新的秘钥的有效期默认为30天。
 - 接口地址：https://m.wosai.cn/api/sign/v2
 - 访问方式：post
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|appid|ISV分配给商户唯一标识|varchar(20)|Y||
|expired|秘钥有效期|bigint|Y|单位为秒|
|sign|参数签名|varchar|Y|详见2.2.如何构造签名|


 - 参数示例：
 
```javascript
{
    "appid":"2200000001",
    "expired":"86400",
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq"
}
```


 - 返回说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|code|响应消息代码|varchar(6)|Y|200为成功，其它为异常|
|message|错误消息提示|varchar(6)|N|code为非200时返回|
|data|新的密钥|varchar(32)|N|code为200时返回|



 - 返回示例：
 
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

###2.2 如何构造签名
    签名是在进行接口调用的过程中，利用密钥和接口包含的所有参数进行MD5签名的过程，具体流程如下：
 - 签名说明：

    接口调用或者生成二维码所有传递的参数都需要一起进行签名，假设本次参数包括appid、storeId、channel、notifyTime、id等5个参数，并且密钥secret=34719280830192，参数中appid=2200000001，notifyTime=1468780992，channel=alipay，storeId=5308，id=61028309128301298，则签名步骤如下：
```
第一步：参数及密钥拼装成6个元素存入到数组array=['2200000001=appid','1468780992=notifyTime','alipay=channel','61028309128301298=id','5308=storeId','34719280830192=secret']；
第二步：对array数组根据ASCII码进行排序得到sortArray=['1468780992=notifyTime','2200000001=appid','34719280830192=secret','5308=storeId','61028309128301298=id','alipay=channel']；
第三步：使用连接符‘&’对数组进行拼接，得到字符串source='1468780992=notifyTime&2200000001=appid&34719280830192=secret&5308=storeId&61028309128301298=id&alipay=channel'；
第四步：对source进行‘md5’32位大写加密，得到sign=Upper(MD5(source))；
```

##3.会员卡信息推送
    消费者在领取/激活/更新会员信息时，收钱吧电子会员系统会根据商户的订阅向商户推送会员信息。推送频率为(实时/2m/10m/1h/6h/12h/24h)，直到24h之后或收到商户的相应为SUCCESS为止。

 - 访问方式：post
 - 参数格式：json
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|sign|参数签名|varchar(128)|Y|详见2.2.如何构造签名|
|notifyTime|消息通知时间|timestamp|Y|1468780992|
|notifyType|消息类型|varchar(10)|Y|active/update|
|channel|渠道|varchar(10)|Y|激活时必填，非激活时选填，wechat/alipay|
|id|渠道标识|varchar(32)|Y||
|storeId|门店标识|varchar(32)|Y|激活时必填，非激活选填|
|name|昵称|varchar(20)|Y|
|birthday|生日|varchar(10)|Y|1977-04-18|
|mobile|手机号|varchar(13)|Y||
|email|邮箱|varchar(40)|N|
|gender|性别|Integer(1)|N|1：男性，2：女性，0：未知|
|headImg|头像图片地址|varchar(200)|N|
|country|国家|varchar(20)|N|
|province|省份|varchar(20)|N|
|city|城市|varchar(20)|N|
|address|地址|varchar(200)|N|
|industry|行业|varchar(40)|N|
|memberId|已入驻会员编号|varchar(40)|N|商户可根据此老会员编号和手机号校验用户是否为已入驻用户|


 - 参数示例：
 
```javascript
{
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq",
    "notifyTime":"1468780992",
    "notifyType":"active",
    "channel":"wechat",
    "id":"691822b1cab64c879feed51045821ab1",
    "storeId":"da6692c0c1f04f0986d9bf687b7d9908",
    "name":"达康书记",
    "birthday":"1977-04-18",
    "mobile":"13112121212",
    "email":"dakang@rmmy.com",
    "gender":"1",
    "headImg":"https://m.wosai.cn/resource/img/dakangshuji/200",
    "country":"中国",
    "province":"汉东省",
    "city":"京州市",
    "address":"XX区XX路XX号大风厂公司X号楼",
    "industry":"金融",
    "memberId":"f8a3dae64f74431dbcc68216af82bdb7"
}
```


 - 返回说明：



 - 返回示例：
 
```javascript
SUCCESS
```

##4.根据会员卡标识查询会员卡信息
    消费者在会员卡界面可以查看用户的基础信息，并可以通过商户提供的本接口查看商户为用户定制的个性化扩展信息，如：会员积分、会员权益、会员消费记录等等。


 - 访问方式：post
 - 参数格式：json
 - 参数说明：

|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|sign|参数签名|varchar(128)|Y|详见2.2.如何构造签名|
|id|渠道标识|varchar(32)|Y||


 - 参数示例：
 
```javascript
{
    "sign":"MDYCGQCNTJhYa4JghYuksPMsE8jO33sq",
    "id":"691822b1cab64c879feed51045821ab1"
}
```


 - 返回说明：
|名称|含义|类型|必填|备注|
|----|:---|:---|:--:|--------|
|code|消息码|varchar(6)Y|||
|message|消息|varchar(60)|N||
|data|业务实体|{}||
| - id|渠道标识|varchar(32)|Y||
| - storeId|门店标识|varchar(32)|Y|激活时必填，非激活选填|
| - name|昵称|varchar(20)|Y|
| - birthday|生日|varchar(10)|Y|1977-04-18|
| - mobile|手机号|varchar(13)|Y||
| - email|邮箱|varchar(40)|N|
| - gender|性别|Integer(1)|N|1：男性，2：女性，0：未知|
| - headImg|头像图片地址|varchar(200)|N|
| - country|国家|varchar(20)|N|
| - province|省份|varchar(20)|N|
| - city|城市|varchar(20)|N|
| - address|地址|varchar(200)|N|
| - industry|行业|varchar(40)|N|
| - card|会员卡信息|[{}]|Y||
| -  - no|会员编号|varchar(12)|Y||
| -  - score|会员积分|bigint|Y||
| -  - level|会员等级|varchar(10)|Y|||
| -  - expire|有效期|varchar(10)|Y|||
| -  - phone|联系方式|varchar(20)|N||
| -  - rights|权益|[]|N|
| -  - instructions|使用须知|[]|N||


 - 返回示例：
 
```javascript
{
    "code":"200",
    "message":"SUCCESS",
    "data":{
        "id":"691822b1cab64c879feed51045821ab1",
        "name":"达康书记",
        "birthday":"1977-04-18",
        "mobile":"13112121212",
        "email":"dakang@rmmy.com",
        "gender":"1",
        "headImg":"https://m.wosai.cn/resource/img/dakangshuji/200",
        "country":"中国",
        "province":"汉东省",
        "city":"京州市",
        "address":"XX区XX路XX号大风厂公司X号楼",
        "industry":"金融",
        "card":[{
            "no":"3000 0000 0001",
            "score":"2000",
            "level":"铂金",
            "expire":"终身有效",
            "phone":"400-800800",
            "rights":["购物9折优惠","积分永不过期","消费免费领取小礼品","生日当天三件商品5折优惠"],
            "instructions":["该会员卡不可分享、转赠，仅限本人使用","H&M保留最终解释权"]
        }]
    }
}
```






