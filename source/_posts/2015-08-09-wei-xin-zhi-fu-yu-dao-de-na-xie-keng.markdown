---
layout: post
title: "微信支付遇到的那些'坑'"
date: 2015-08-09 20:10:38 +0800
comments: true
categories: iOS
---
##前言
本来以为微信支付和支付宝一样简单，可是一弄才知道有好多坑，现在我记录下来，也让别人少走一些弯路.
[官方文档](https://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=8_1#)
<!--more-->

 1. 报错1>服务器返回：
<xml><return_code><![CDATA[FAIL]]></return_code>
<return_msg><![CDATA[mch_id和appid没有关联关系]]></return_msg>
</xml>
然后就各种百度，各种google，后来才发现，原来是客户申请了两次，appiD和mch_id弄错了，无语~~

2. 报错2>签名错误，各种调试，后来才发现，原来是客户给的APIKey不对，这个apiKey一定要用工具生成才行，自己手写的不行

更正了这两个错误后，微信支付现在变的简单了。



### 微信支付流程如下：

步骤1：用户在商户APP中选择商品，提交订单，选择微信支付。

步骤2：商户后台收到用户支付单，调用微信支付统一下单接口。参见[统一下单API](https://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=9_1)。

步骤3：统一下单接口返回正常的prepay_id，再按签名规范重新生成签名后，将数据传输给APP。参与签名的字段名为appId，partnerId，prepayId，nonceStr，timeStamp，package。注意：package的值格式为Sign=WXPay

步骤4：商户APP调起微信支付。api参见本章节[app端开发步骤说明](https://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=8_5)

步骤5：商户后台接收支付通知。api参见[支付结果通知API](https://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=9_7)

步骤6：商户后台查询支付结果。，api参见[查询订单API](https://pay.weixin.qq.com/wiki/doc/api/app.php?chapter=9_2)



