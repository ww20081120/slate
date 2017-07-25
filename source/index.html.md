---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# 1 简介

## 1.1 范围

本标准规定了VCC业务系统与第三方业务系统的通信协议，供港华集团内部和厂商共同使用，用于在业务开展、设备开发方面为集团公司和各地市公司提供技术依据。

## 1.2 规范性引用文件

下列文件中的条款通过本标准的引用而成为本标准的条款。凡是注日期的引用文件，其随后所有的修改单(不包括勘误的内容)或修订版均不适用于本标准，然而，鼓励根据本标准达成协议的各方研究是否可使用这些文件的最新版本。凡是不注日期的引用文件，其最新版本适用于本标准。

序号|版本|参考文件|归属
----|----|------|----
[1]|V2.0|《TCIS3.0系统与VCC系统接口规范》|港华投资有限公司
[2]|V1.0.20160420|《TCIS2.0系统与VCC系统接口规范》|港华投资有限公司
[3]|V1.6|《港华集团网上客户系统与圈存机通信协议》|港华投资有限公司

## 1.3 协议说明

本章节主要讲述https和ftps两种协议，https用于实时交互接口，例如账户信息查询、圈存机提气及结果上报等接口，sftp用于文件同步类接口，如按日对账接口。[]代表可选参数，{}代表必须参数。

本章节主要讲述https和ftps两种协议，https用于实时交互接口，例如账户信息查询、圈存机提气及结果上报等接口，sftp用于文件同步类接口，如按日对账接口。[]代表可选参数，{}代表必须参数。

### HTTP Request

`GET https://api.towngasvcc.com/vcc-openapi/{resource}?{query_string}`

#### 说明
1. resource为操作业务资源名；
2. query_string由通用参数部分和具体API调用参数部分组成；
3. query_string中的key/value对都必须经过urlencode处理，而且必须是UTF-8编码；
4. 对于GET请求，query_string必须放在QUERY参数中传递，即放在“？”后面；

#### 错误情况下返回参数
> The above command returns JSON structured like this:

```json
{
    "resultCode": "RESULT CODE",
    "resultMsg": "RESULT MESSAGE"
}
```
参数名称 | 类型 | 长度 | 描述 | 是否必须
--------- | ------- | ------- | -------------- | -------
resultCode | string | | 结果码, 参考[附录A](#errorCode) |Y
resultMsg | string | 256 | 错误消息 | Y

# 2	安全机制

由于VCC系统支持多种通信方式作为承载手段，而在某些特定环境下，一些通信方式本身就存在着一定安全风险。因此，本协议的安全机制主要是在应用协议层解决圈存机被非法使用和圈存机与VCC平台的数据交互安全问题，同时尽可能的不依赖于具体的通信方式。

## 2.1 数据交互安全

为保证VCC与业务系统之间的数据交互安全，本协议采用接入密码安全验证和报文摘要的方式，以实现在报文交互中对通信双方的身份验证并确保报文的完整性；采用报文内容加密的方式，以保证报文内容的安全性。

### 通信安全性
本协议通过在报文体之后增加接入密码安全验证并对整个消息体进行摘要处理，以实现VCC圈存机与VCC平台之间交互报文来源的身份验证并保证报文的完整性，从而确保VCC圈存机与VCC平台的通信安全。

#### 消息的摘要算法
请求消息接入安全验证报文包含如下几部分：

<table>
<tr>
<td colspan="2">类型</td>
<td>说明</td>
</tr>
<tr>
<td rowspan="2">请求头</td>
<td>普通参数</td>
<td>普通参数由多个键值队组成，多个参数用&分割，如appId=xxx&deviceId=XXX</td>
</tr>
<tr>
<td>签名</td>
<td>签名也为固定key的键值对，key为sign，值为UpperCase(MD5(sort(普通参数)+报文体+接入密码))。其中接入密码，是VCC终端首次接入VCC平台后，由VCC平台分配的一个零时会话密钥，签名算法在计算时，普通参数需要去掉参数间所有的连接符，如“=” 、“&”符号。如果请求头中value为空，key不能参与计算；如果value不为空，则必须参与加密。sort为普通参数在参与签名时需要按照key的字母顺序进行排序。UpperCase为MD5的字符都转换为大写字母。
</td>
</tr>
</table>

响应消息接入安全验证报文包含如下几部分:

类型|说明
----|-----------
头参数|由多个固定key的键值对，包含结果码、结果码描述、时间戳
普通参数|由多个键值对组成，由具体业务接口协议定义。
签名|签名值为UpperCase(MD5(sort(头参数、普通参数) + 接入密码))。其中接入密码，是VCC终端首次接入VCC平台后，由VCC平台分配的一个零时会话密钥。 sort为将头参数、普通参数的key按字母顺序排序。如果参数值为空，则不参与签名计算。UpperCase为MD5的字符都转换为大写字母。

### 数据安全性
本协议支持通过对内容体中的数据进行加密，以保证数据在传输过程中的安全性。当VCC平台接收到VCC圈存机发上来的报文时，首先对HTTPS报文体中加密的消息进行解密，然后对签名进行接入安全验证；若通过验证，再执行报文中承载的命令。

#### 基础密钥的预置、分发与变更
VCC圈存机在使用前需要预置基础密钥，基础密钥是由字母和数字组成的32位长度的字符串，由VCC平台生成。对于支持短信的圈存机，基础密钥可以通过短信接口下发到圈存机中，圈存机对基础密钥的存储需要通过密文的方式进行存储。
#### 会话密钥的分发与变更
VCC在进行业务操作前，需要请求会话密钥，会话密钥参与后续业务交互的消息验证。会话密钥请求时会返回一个会话密钥的有效期，在有效期超时前，需要重新请求会话密钥，进行更新。

# 3 接入授权

## 1002服务会话密钥请求（企业）
<span id="token">对于</span>支持短信的圈存机可以使用该接口设置或修改圈存机的基础密钥，对于不支持短信的圈存机可以直接预置基础密钥。当基础密钥被重置后，会话密钥将会失效。

### 承载协议

HTTPS协议，请求方式GET，响应数据为JSON格式。

`GET /oauth2/getToken?seq=xxxxxxxxxxxxx&appId=xxxx&sign=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 交互流程

![image](https://github.com/ww20081120/slate/raw/master/source/images/p1.png)

### 请求参数

> To authorize, use this code:

```shell
curl "https://api.towngasvcc.com/vcc-openapi/oauth2/getToken?seq=SEQ&appId=APP_ID&sign=SIGN"
```

参数名称 | 类型 | 长度 | 描述 | 是否必须
--------- | ------- | ------- | -------------- | -------
seq | string | 32 | 消息序列号，前4位为接口编码1001，5～18位为时间戳，格式为yyyyMMddHHmmss，19～32位为消息流水号，00000000000001～99999999999999，达到最大值后可以循环使用。|Y
appId | string | 20 | VCC平台分配的应用ID | Y
sign | string | 32 | 安全加密签名，算法参考[摘要算法](#sgin)，其中基础密钥需要第三方开发商向VCC平台申请，申请邮箱 sun.lei06@towngas.com.cn，ww20081120@139.com | Y

### 返回参数

> The above command returns JSON structured like this:

```json
{
    "expireTime": 1486488177,
    "token": "1Z5M011v4P5joM9VqMr0"
}
```

参数名称 | 类型 | 长度 | 描述 | 是否必须
--------- | ------- | ------- | -------------- | -------
expireTime | int | | 会话密钥失效时间，单位为秒，默认7200 | Y
token| string | 20 | 20位的接入令牌，由字母数字组成 | Y

# 4 基础服务

## 2001 文件上传

接口中经常有需要用到一些多媒体素材的场景，例如在工单中上传身份证、房产证照片等操作，是通过mediaId来进行的。通过本接口，可以上传多媒体文件。

### 承载协议

HTTPS协议，请求方式 POST/FORM，响应数据为JSON格式。

`/media/upload?seq=SEQ&token=TOKEN&type=TYPE`

### 请求参数

> 调用示例（使用curl命令，用FORM表单方式上传一个多媒体文件）：

```shell
curl -F media=@test.jpg "https://api.towngasvcc.com/vcc-openapi/media/upload?seq=SEQ&token=TOKEN&type=TYPE"
```

参数名称 | 类型 | 长度 | 描述 | 是否必须
--------- | ------- | ------- | -------------- | -------
seq | string | 32 | 消息序列号，前4位为接口编码1001，5～18位为时间戳，格式为yyyyMMddHHmmss，19～32位为消息流水号，00000000000001～99999999999999，达到最大值后可以循环使用。|Y
token | string | 20 | 20位的接入令牌，由[1002会话密钥请求接口](#token)获取| Y
type | string | | 媒体文件类型，分别有图片（image）、语音（voice）和缩略图（thumb）| Y
media | string | |	是	form-data中媒体文件标识，有filename、filelength、content-type等信息 | Y

<aside class="warning">
注意点：<br/>
1、临时素材media_id是可复用的。<br/>
2、上传临时素材的格式、大小限制与公众平台官网一致。<br/>
    图片（image）: 2M，支持PNG\JPEG\JPG\GIF格式<br/>
    语音（voice）：2M，播放长度不超过60s，支持AMR\MP3格式<br/>
    缩略图（thumb）：64KB，支持JPG格式<br/>
</aside>

### 返回参数

> 正确情况下的返回JSON数据包结果如下：

```json
{
    "type": "TYPE",
    "mediaId": "MEDIA_ID",
    "createTime": 123456789
}
```

参数名称 | 类型 | 长度 | 描述 | 是否必须
--------- | ------- | ------- | -------------- | -------
type | string | |	媒体文件类型，分别有图片（image）、语音（voice）、视频（video）和缩略图（thumb，主要用于视频与音乐格式的缩略图）| Y
mediaId | string | 32 | 媒体文件上传后，获取标识 | Y
createTime | datetime |	媒体文件上传时间戳 格式为yyyyMMddhhmmss | Y

## 2002 文件下载

可以使用本接口获取素材（即下载多媒体文件）。

### 承载协议

HTTPS协议，请求方式 GET。

`/media/get?seq=SEQ&token=TOKEN&&mediaId=MEDIA_ID`

### 请求参数

> 请求示例（示例为通过curl命令获取多媒体文件）：

```shell
curl -I -G "https://api.towngasvcc.com/vcc-openapi/media/get?seq=SEQ&token=TOKEN&&mediaId=MEDIA_ID"
```

参数名称 | 类型 | 长度 | 描述 | 是否必须
--------- | ------- | ------- | -------------- | -------
seq | string | 32 | 消息序列号，前4位为接口编码1001，5～18位为时间戳，格式为yyyyMMddHHmmss，19～32位为消息流水号，00000000000001～99999999999999，达到最大值后可以循环使用。|Y
token | string | 20 | 20位的接入令牌，由[1002会话密钥请求接口](#token)获取| Y
media | string | |	是	form-data中媒体文件标识，有filename、filelength、content-type等信息 | Y

### 返回参数

> 正确情况下的返回结果如下：

```shell
HTTP/1.1 200 OK
Connection: close
Content-Type: image/jpeg 
Content-disposition: attachment; filename="MEDIA_ID.jpg"
Date: Sun, 06 Jan 2013 10:20:18 GMT
Cache-Control: no-cache, must-revalidate
Content-Length: 339721
curl -G "https://api.towngasvcc.com/vcc-openapi/media/get?seq=SEQ&token=TOKEN&&mediaId=MEDIA_ID"
```

# 5 燃气数据查询

提供燃气相关用户、余额、账单、业务办理记录等信息查询

## 3001 地址信息增量同步（企业）


## 3002 供气信息增量同步（企业）

## 3003 气户数据增量同步（企业）

## 3004 表具信息增量同步（企业）

## 3005 气户信息查询接口

## 3006 绑定户号查询

## 3007 根据地址查询气户信息

## 3008 联系人信息查询

## 3009 表具信息查询

## 3010 IC卡信息查询

## 3011 气价查询

## 3012 已使用气量查询

## 3013 预存金额查询

## 3014 欠费账单查询

## 3015 气费账单查询

## 3016 IC卡充值历史查询

## 3017 气量账单查询

## 3018 多户号批量查询气量账单（企业）

## 3019 IC卡气价查询

## 3020 抄表历史查询

## 3021 内购燃气具销售记录查询

## 3022 外购燃气具销售记录查询

## 3023 安检记录查询

## 3024 隐患情况查询

## 3025 数据字典查询

# 6 燃气业务办理

提供燃气相关户号绑定、解绑、业务工单申请、缴费、圈存等业务办理

## 4001 气户与会员信息绑定接口

## 4002 气户与会员信息解绑接口

## 4003 联系人信息变更

## 4004 自报读数

## 4005 自报读数试算

## 4006 IC卡充值

## 4007 机械表缴费

## 4008 营业费缴费

## 4009 安检情况上报

# 7 业务工单

## 5001 业务工单创建

## 5002 业务工单状态变更

## 5003 业务工单取消

## 5004 在途工单查询

## 5005 查看工单详情

# 8 圈存厂商

## 6001 圈存机注册接口（企业）

# 9 推送事件

## 7001 地址信息变化

## 7002 供气信息变化

## 7003 气户数据变化

## 7004 表具信息变化

## 7005 工单状态变化

# A 错误码定义

<span id="errorCode">x</span>

# B 组织结构编码定义

# C 用户状态

# D 气表类型

# Introduction

Welcome to the Kittn API! You can use our API to access Kittn API endpoints, which can get information on various cats, kittens, and breeds in our database.

We have language bindings in Shell, Ruby, and Python! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This example API documentation page was created with [Slate](https://github.com/tripit/slate). Feel free to edit it and use it as a base for your own API's documentation.

# Authentication

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint retrieves a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete
