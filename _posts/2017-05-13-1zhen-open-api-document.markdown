---
title: 1诊开放平台接口文档
tags: 新建,1诊开放平台
grammar_cjkRuby: true
---

# 目录

* [目录](#目录)
	* [接口概述](#接口概述)
	* [API接口](#api接口)
		* [用户登录接口](#用户登录接口)
		* [问诊接口](#问诊接口)
		* [查询接口](#查询接口)
	* [H5接口](#h5接口)
		* [问诊页面](#问诊页面)
		* [查询页面](#查询页面)

## 接口概述
>版本说明

| 版本号    | 时间    |  人员   | 说明    |
| --- | --- | --- | --- |
 |   1.0  | 2017年05月24日 14时36分15秒    | 章培昊，汪涛    |  新建版本，不完善   |

>注意事项
1. 请求方式: HTTP + JSON + POST / GET

2. 统一使用UTF-8编码

3. 文档中 partner 变量为具体合作方标识,需要申请

4. 1诊开放平台测试服务地址 https://yzm.test.111.com.cn/

5. 1诊开放平台正式服务地址 https://yzm.111.com.cn/

6. 双方进行通信时使用加密密钥(partner_key)进行安全校验, 密钥请严格保密

7. 出于安全考虑，sign必须在服务器端生成

8. sign 为签名密钥值, 生成方式如下:

```
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import org.apache.commons.codec.binary.Hex;

public class SignTest {

    // 计算 Sign
    private static String getSign(String partner_key, String atime, String user_id)throws NoSuchAlgorithmException{
        String info = partner_key + atime + user_id;
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        byte[] srcBytes = info.getBytes();
        md5.update(srcBytes);
        byte[] resultBytes = md5.digest();
        String resultString = new String(new Hex().encode(resultBytes));
        return resultString.substring(8, 24);
    }

    public static void main( final String[] args ){
        try {
            // 合作方 partner_key， 注意不是 partner
            String partner_key = "XKBP1Oqut0r2LiGV";
            // UNIX TIMESTAMP 最小单位为秒
            String atime = "1467098815";
            // 第三方用户唯一标识，可以为字母与数字组合的字符串。
            String user_id = "A800130";
            // 计算sign结果为: 5afda19c5d65a7a7
            String sign = SignTest.getSign(partner_key, atime, user_id);
            System.out.println(sign);
        }
        catch (Exception ex)
        {
            ex.printStackTrace();
        }

    }
}
```
## API接口
### 用户登录接口

``` stylus
1诊提供给每一个合作商一个特定partner_key值,partner_key为1诊的加密密钥。合作商访问h5服务的时候,提供自定义的user_id来标识每一个用户。使用partner_key、时间戳以及user_id加密生成sign,完成h5服务的验证。
正确的格式为：
/cooperation/wap/login/?user_id=A800130&atime=1467098815&partner=yiyaowang&sign=5afda19c5d65a7a7
```
URL:/cooperation/wap/login/
请求方式: GET
请求参数:

|  名称   |  说明	   |  类型   |  长度   | 必要|备注| 
| --- | --- | --- | --- |--------|-----|
|user_id|用户名	|String|32|是|用户唯一标识,合作方定义|
|partner|合作方标识		|String|32|是|==需要申请==|
|sign|签名		|String|32|是|==生成方法==|
atime|签名时间戳	|Long|64|是|当前UNIX TIMESTAMP签名时间戳 (如:137322417)
返回: 1诊的问诊h5页面


### 问诊接口

### 查询接口

## H5接口

### 问诊页面

### 查询页面
