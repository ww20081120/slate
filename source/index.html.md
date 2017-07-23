---
title: VCC API

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

# 范围

本标准规定了VCC业务系统与第三方业务系统的通信协议，供港华集团内部和厂商共同使用，用于在业务开展、设备开发方面为集团公司和各地市公司提供技术依据。
## 规范性引用文件

下列文件中的条款通过本标准的引用而成为本标准的条款。凡是注日期的引用文件，其随后所有的修改单(不包括勘误的内容)或修订版均不适用于本标准，然而，鼓励根据本标准达成协议的各方研究是否可使用这些文件的最新版本。凡是不注日期的引用文件，其最新版本适用于本标准。

序号|版本|参考文件|归属
----|----|------|----
[1]|V2.0|《TCIS3.0系统与VCC系统接口规范》|港华投资有限公司
[2]|V1.0.20160420|《TCIS2.0系统与VCC系统接口规范》|港华投资有限公司
[3]|V1.6|《港华集团网上客户系统与圈存机通信协议》|港华投资有限公司

## 简介

本章节主要讲述https和ftps两种协议，https用于实时交互接口，例如账户信息查询、圈存机提气及结果上报等接口，sftp用于文件同步类接口，如按日对账接口。[]代表可选参数，{}代表必须参数。

本章节主要讲述https和ftps两种协议，https用于实时交互接口，例如账户信息查询、圈存机提气及结果上报等接口，sftp用于文件同步类接口，如按日对账接口。[]代表可选参数，{}代表必须参数。

### HTTP Request

`GET https://api.towngasvcc.com/vcc-openapi/{resource}?{query_string}`

#### 说明
1. resource为操作业务资源名；
2. query_string由通用参数部分和具体API调用参数部分组成；
3. query_string中的key/value对都必须经过urlencode处理，而且必须是UTF-8编码；
4. 对于GET请求，query_string必须放在QUERY参数中传递，即放在“？”后面；

# 安全机制
由于VCC系统支持多种通信方式作为承载手段，而在某些特定环境下，一些通信方式本身就存在着一定安全风险。因此，本协议的安全机制主要是在应用协议层解决圈存机被非法使用和圈存机与VCC平台的数据交互安全问题，同时尽可能的不依赖于具体的通信方式。

## 数据交互安全

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

