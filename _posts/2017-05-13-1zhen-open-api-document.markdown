---
title: 1诊开放平台接口文档
tags: 新建,1诊开放平台
excerpt: 1诊开放平台接口文档
grammar_cjkRuby: true
---

# 目录
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## 版本

| 版本号    | 时间    |  人员   | 说明    |
| --- | --- | --- | --- |
|   1.0  | 2017年05月24日 14时36分15秒    | 章培昊，汪涛    |  新建版本，不完善   |
{:.mbtablestyle}

## 接口概述

### 接口规则

1. 统一使用UTF-8编码

2. 文档中 partner 变量为具体合作方标识,需要申请

3. 1诊开放平台测试服务地址 https://yzm.test.111.com.cn/（需要配置Host）

4. 1诊开放平台正式服务地址 https://yzm.111.com.cn/

### Sign计算规则

* `认证校验字符串`: 根据每个接口参与Sign计算的参数，字符连接组成认证校验字符串。
* `ts`: 每个接口必须要传unix时间戳(1970年1月1日0时开始的秒数)，线上环境有15分钟超时，请保证调用系统的时钟同步。
* `partner_key`: 密钥。请妥善保存！

```
sign = md5(认证校验字符串, ts, partner_key)
```

## API接口

### 问诊接口


整理中...

### 查询接口


整理中...

## HTTP接口

### 问诊页面

URL:`m/cooperation-h5/login.html`

请求方式: HTTP跳转

请求参数:

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|partner|合作方标识|String|32|是|／|==需要申请==|
|user_id|用户名|String|32|是|是|用户唯一标识,合作方定义|
|redirect_url|登入后跳转页面|String|64|是|是|根据业务不同重定向页面不同|
|ts|签名时间戳|Long|64|是|／|当前UNIX TIMESTAMP签名时间戳 (如:137322417)|
|sign|签名|String|32|是|／|==生成方法==|
|mobile|手机号|String|11|是|／|用户手机号|

### 查询页面


整理中...
