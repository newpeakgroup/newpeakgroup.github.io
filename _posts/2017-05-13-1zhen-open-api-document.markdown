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
|   1.1  | 2017年05月26日 17时48分01秒    | 章培昊         |  修改Sign计算方法   |
|   1.2  | 2017年06月01日 09时31分23秒    | 章培昊         |  时间戳作为参数进行Sign计算   |
|   1.3  | 2017年06月12日 15时58分26秒    | 章培昊         |  增加问诊查询接口   |
|   1.4  | 2017年08月22日 15时58分26秒    | 周蒙蒙         |  新增添加患者接口   |

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

## API接口

### 问诊接口

整理中...

### 查询用户诊单接口

#### 请求

URL:`partner/inquerylist/my`

请求方式: POST

请求参数: form-data

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|partner|合作方标识|String|32|是|／|==需要申请==|
|system|合作方标识|String|32|是|／|固定值partner|
|user_id|用户名|String|32|是|是|要查询的用户唯一标识|
|count|每页的问诊数|Integer|32bit|是|否|分页查询里，每一页包含的问诊数，最大50|
|start_num|页码|Integer|32bit|是|否|分页查询里，要查询的页数。例如：第一页传1，第二页传2|
|ts|签名时间戳|Long|64bit|是|是|unix时间戳(1970年1月1日0时开始的秒数)，线上环境有15分钟超时，请保证调用系统的时钟同步。|
|sign|签名|String|32|是|／|生成方法参考“Sign计算规则”|

#### 返回

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|ret|接口调用返回码|Integer|32bit|是|1: 成功, 其他: 错误码|
|msg|接口调用结果消息|String|256|是|如果调用失败，这里有错误信息|
|data|数据结果|Object|/|是|数据定义，参考"[诊单查询结果](#inquery_result)"|

* **诊单查询结果**<a name="inquery_result"/>

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|count||Integer|32bit|是|诊单查询结果总数量（不是分页内诊单数量）|
|partner_inquery_list|诊单数组|Array(Object)|/|是|诊单查询结果（分页数据），诊单对象定义，参考"[诊单对象](#inquery_data)"|

* **诊单对象**<a name="inquery_data"/>

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|create_time|诊单创建时间|Long|64bit|是|创建时间戳（1970年1月1日0时开始的秒数）|
|finish_flag|诊单完成标志|String|32|是|包含：待处理，处理中，已完成，已取消|
|guest_dept|科室|String|32|是|例如：妇科|
|inquery_id|诊单ID|Integer|32bit|是|诊单表唯一主键|
|mobile|注册用户电话号码|String|32bit|是|注册用户电话号码|
|pay_flag|支付状态标志|String|32|是|包含：无需支付；待支付；已支付|
|photos|用户上传的照片|String|256|是|多个图片用逗号分隔。例如: hc/1.jpg,hc/2.jpg，图片服务域名参考"[1诊开放平台图片服务地址](#img_host)"|
|recipe_list|处方数组|Array(Object)|/|是|处方数组，诊单对象定义，参考"[处方对象](#recipe_data)"|
|self_desc|自诉|String|512|是||
|service_type|服务类型|String|32|是|选项包含：电话问诊，图文问诊，视频问诊|

* **处方对象**<a name="recipe_data"/>

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|doctor_name|医生姓名|String|64|是||
|patient_name|患者姓名|String|64|是||
|patient_phone|患者电话|String|64|是||
|pdf_url|处方PDF文件地址|String|256|是|例如：recipe/1.pdf，图片服务域名参考"[1诊开放平台图片服务地址](#img_host)"|
|recipe_code|处方编号|String|64|是||
|recipe_time|处方创建时间|String|16|是|格式："YYYY-MM-DD"，例如：2017-06-12|
|sex|性别|String|1|是|选项包含：F, M|
|medical_list|药品列表|Array(Object)|/|是|处方数组，诊单对象定义，参考"[药品对象](#medical_list)"


* **药品对象**<a name="medical_list"/>

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|drug_code|药品ID|String|64|是||
|drug_name|药品名称|String|64|是||
|specification|药品规格|String|256|是|例如：100mg*7片|
|specification_num|药品数量|String|64|是|例如：11|
|url|药品连接|String|16|是|格式：http://m.111.com.cn/product/1191.html（如果该药品是第三方药品库，那么药品展示连接就是第3提供的连接）|
|usage|用法用量|String|1|是|/|



### 按照时间范围查询诊单接口

#### 请求

URL:`partner/inquerylist/range`

请求方式: POST

请求参数: form-data

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|partner|合作方标识|String|32|是|／|==需要申请==|
|system|合作方标识|String|32|是|／|固定值partner|
|user_id|保留字段|String|32|是|是|只能填'*'|
|begin|范围开始时间|String|/|是|否|时间格式2017-01-01|
|key_str|诊单ID|String|/|是|否|诊单ID：187|
|end|范围结束时间|String|/|是|否|时间格式2017-01-01|
|count|每页的问诊数|Integer|32bit|是|否|分页查询里，每一页包含的问诊数，最大50|
|start_num|页码|Integer|32bit|是|否|分页查询里，要查询的页数。例如：第一页传1，第二页传2|
|ts|签名时间戳|Long|64bit|是|是|unix时间戳(1970年1月1日0时开始的秒数)，线上环境有15分钟超时，请保证调用系统的时钟同步。|
|sign|签名|String|32|是|／|生成方法参考“Sign计算规则”|

#### 返回

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|ret|接口调用返回码|Integer|32bit|是|1: 成功, 其他: 错误码|
|msg|接口调用结果消息|String|256|是|如果调用失败，这里有错误信息|
|data|数据结果|Object|/|是|数据定义，参考"[诊单查询结果](#inquery_result)"|

## HTTP接口

### 问诊页面

#### 请求

URL:`m/cooperation-h5/login.html`

请求方式: HTTP跳转

请求参数:

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|partner|合作方标识|String|32|是|／|==需要申请==|
|user_id|用户名|String|32|是|是|用户唯一标识,合作方定义|
|mobile|手机号|String|11|是|否|用户手机号|
|redirect_url|登入后跳转页面|String|64|是|否|根据业务不同重定向页面不同，需encodeURIComponent处理|
|ts|签名时间戳|Long|64bit|是|是|unix时间戳(1970年1月1日0时开始的秒数)，线上环境有15分钟超时，请保证调用系统的时钟同步。|
|sign|签名|String|32|是|／|生成方法参考“Sign计算规则”|

### 添加患者接口

#### 请求

URL:`app/open/add_patient`

请求方式: POST

请求参数: form-data

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|partner|合作方标识|String|32|是|/|==需要申请==|
|system|合作方标识|String|32|是|／|固定值partner|
|user_id|保留字段|String|32|是|是|用户唯一标识,合作方定义|
|ts|签名时间戳|Long|64bit|是|是|unix时间戳(1970年1月1日0时开始的秒数)，线上环境有15分钟超时，请保证调用系统的时钟同步。|
|sign|签名|String|32|是|／|生成方法参考“Sign计算规则”|
|name|姓名|String|/|是|否|姓名|
|birthday|出生日期|String|/|是|否|时间格式2017-01-01|
|sex|性别|String|/|是|否|F-女;M-男|
|card_num|身份证号码|String|32bit|是|否|身份证号码|
|is_self|是否本人<是否设置为默认就诊人>0:非本人;1:本人|int|32bit|是|否|/|
|mobile|注册用户的手机号码（不是患者的号码）|Integer|32bit|是|否|/|


#### 返回

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|ret|接口调用返回码|Integer|32bit|是|1: 成功, 其他: 错误码|
|msg|接口调用结果消息|String|256|是|如果调用失败，这里有错误信息|
|data|数据结果|Object|/|是|数据定义|
|data.is_real_name|是否实名|String|/|是|N|
|data.patient_id|患者ID|Object|/|是|2607|


### 第3方药品库需要提供的接口

#### 请求

URL:`<合作方提供>`

请求方式: POST

请求参数: form-data

| 名称 | 说明 | 类型 | 长度 | 必要 | 参与Sign计算 |备注|
| :-- | :-- | :-- | -- | -- | -- | :-- |
|key_word|合作方标识|String|32|否|/|搜索关键词（可以搜索药品ID，药品名等）|
|page_index|保留字段|String|32|否|/|页码|
|page_size|签名时间戳|String|32|否|/|显示条数|


#### 返回

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|ret|接口调用返回码|Integer|32bit|是|1: 成功, 其他: 错误码|
|msg|接口调用结果消息|String|256|是|如果调用失败，这里有错误信息|
|data|数据结果|Object|/|是|数据定义|
|data.pageInfo|是否实名|Object|/|是|页码返回参数集合|
|data.drugList|药品对象|Object(Array)|/|是|药品对象|


* **页码对象**<a name="pageInfo"/>

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|totalPage|当前总条数|String|/|是|/|
|pageSize|取条数|String|/|是|/|
|totalCount|总条数|String|/|是|/|
|pageIndex|页码|String|/|是|/|

* **药品对象**<a name="drugList"/>

| 字段 | 说明 | 类型 | 长度 | 必要 |备注|
| :-- | :-- | :-- | -- | -- | :-- |
|drug_code|药品ID|String|/|是|/|
|drug_name|药品名称|String|/|是|/|
|specification|规格|String|/|是|/|
|drug_desc|合作方药品描述|String|/|是|接默认请填"暂无"|
|picture|图片地址|String|/|是|接默认请填"#"|
|brand|品牌|String|/|是|/|
|price|价格|String|/|是|/|
|detail_url|药品连接地址|String|/|是|若无连接默认请填"#"|
|store|库存数|String|/|否|0|
