---
title: 1诊开放平台接口文档
tags: 新建,1诊开放平台
excerpt: 1诊开放平台接口文档
grammar_cjkRuby: true
---

# 1诊开放平台接口文档
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

## 版本

| 版本号 | 时间 | 人员 | 说明 |
| --- | --- | --- | --- |
|   1.0  | 2017年05月24日 14时36分15秒    | 章培昊，汪涛    |  新建版本，不完善   |


## 接口概述

### 接口规则

1. 统一使用UTF-8编码

2. 文档中 partner 变量为具体合作方标识,需要申请

3. 1诊开放平台测试服务地址 https://yzmt.111.com.cn/

4. 1诊开放平台正式服务地址 https://yzm.111.com.cn/

5. 1诊开放平台图片服务地址 https://yzimg.111.com.cn/<a name="img_host"/>


### Sign计算规则

* `partner_key`: 每个合作方的密钥。请妥善保存！
* `待签名参数Map`: 每个接口规定了需要参加签名校验的参数，将这些参数按照参数名为`key`，参数值为`value`，组成Map
* `待签名参数字符串`: 将`待签名参数Map`按key升序排序，按照`key=<value>`格式用`&`连接组成
* `待签名字符串`: `<待签名参数字符串>`+`&key=<partner_key>`
* `Sign`: md5(<待签名字符串>)

Java8代码例子：

```java
final String PARTENER_KEY = "REPLACE_YOUR_SECURITY_KEY_HERE";

HashMap<String, String> signArgs = new HashMap<>();
signArgs.put("user_id", "somebody@company");
signArgs.put("ts", "1496281246");
signArgs.put("other_argument", "value");

Map<String, String> sortedArgs = signArgs.entrySet().stream()
        .sorted(Map.Entry.comparingByKey())
        .collect(Collectors.toMap(Map.Entry::getKey,
                Map.Entry::getValue,
                (oldValue, newValue) ->
                        oldValue, LinkedHashMap::new));
String signString = sortedArgs.entrySet().stream()
        .map((entry) -> entry.getKey() + "=" + entry.getValue())
        .collect(Collectors.joining("&"));

signString = signString+"&key="+PARTENER_KEY;
System.out.println(signString);

byte[] signBytes = signString.getBytes();

try {
    MessageDigest md = MessageDigest.getInstance("MD5");
    byte[] md5Bytes = md.digest(signBytes);
    String md5Result = String.format("%032x",
            new BigInteger(1, md5Bytes));
    System.out.print(md5Result);
} catch (NoSuchAlgorithmException ex) {
    ex.printStackTrace();
}
```

输出结果：

```
other_argument=value&ts=1496281246&user_id=somebody@company&key=REPLACE_YOUR_SECURITY_KEY_HERE
53af7f47af43334ec842546e7c655f75
```

如果发现验证失败，请将`signSring`发给岗岭集团的联调开发技术人员，方便排查。




## HTTP接口

### 问诊页面

#### 请求

URL:`m/cooperation-h5/login.html`

请求方式: HTTP跳转

请求方式: GET

请求参数:

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|partner|合作方标识|String|32|是|／|==需要申请==|
|user_id|用户名|String|32|是|是|用户唯一标识,合作方定义|
|mobile|手机号|String|11|是|否|用户手机号|
|redirect_url|登入后跳转页面|String|64|是|否|根据业务不同重定向页面不同，需encodeURIComponent处理|
|ts|签名时间戳|Long|64bit|是|是|unix时间戳(1970年1月1日0时开始的秒数)，线上环境有15分钟超时，请保证调用系统的时钟同步。|
|sign|签名|String|32|是|／|生成方法参考“Sign计算规则”|

举例：
http://yzm.111.com.cn/m/cooperation-h5/login.html?system=partner&app_version=11&user_id=xxxx&partner=xxx&sign=b4a0654fefa03faf9c505d69cae8138c&ts=1505715822&mobile=13600000000&redirect_url=http%3A%2F%2Fyzm-test.111.com.cn%2Fm%2Fyw-twwz%2Findex.html