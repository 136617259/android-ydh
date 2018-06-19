Android
=======

运动汇，Android客户端代码。


# 运动汇后台 API 文档

[TOC]

## 0. 综述

### 1. API 说明

此 API 按照 RESTful 原则，参照 GitHub 开放 API 的模式设计。

### 2. 参数

本文档所指参数分为两类。

**资源参数**：在 URL 中以 `:` 开头，表示使用时，以实际值代替，如获取指定用户详情，`/users/:userId`，使用时以用户 ID 代替 `:userId`，实际请求为 `/users/1`。

**URL 参数**，是指追加到 URL 上的一个名称/值对，参数以问号 `?` 开始并采用 `name=value` 的格式。如果存在多个 URL 参数，则参数之间用一个 `&` 符隔开。下述参数均指 URL 参数。

### 3. HTTP 方法

RESTful 使用 HTTP 动词表达对资源的操作类型。

|方法    |说明|
|---|---|
|GET    |获取资源|
|POST   |创建资源|
|PUT    |更新资源，提供资源所需的所有属性|
|PATCH  |更新资源，提供一个或多个将要修改的资源属性，在一定情况下可替代PUT方法|
|DELETE |删除资源|

针对不同的资源路径，表达的含义略有不同。

|方法|/users|/users/1|
|---|---|---|
|GET    |获取所有用户|获取某个用户|
|POST   |创建用户||
|PUT    ||修改用户|
|PATCH  ||修改用户|
|DELETE ||删除用户|

RESTful 设计中 HTTP 方法的安全性和幂等性。

|方法|安全性|幂等性|
|---|---|---|
|GET    |是|是|
|POST   |否|否|
|PUT    |否|是|
|PATCH  |否|是|
|DELETE |否|是|

**安全性** 安全的方法是只读的，不会改变资源的状态。

**幂等性** 幂等的方法会修改资源状态，但多次调用不会产生不同的结果。

举例说明：

GET 方法不会对资源造成修改所以是安全的，同时也是幂等的。

DELETE 方法对资源进行删除，不是安全的，但删除后再次调用时已经删除了，不会产生不同后结果，所以是幂等的。

POST 方法会创建一个资源，不是安全的，多次调用会创建多个资源，所以不是幂等的。

### 4. 状态码

本 API 所用状态码如下

|状态码|适用HTTP方法|说明|
|---|---|---|
|200 OK|GET/PUT/PATCH|操作成功|
|201 Created|POST|创建成功|
|204 No Content|DELETE|删除成功|
|400 Bad Request|*|请求URL参数或请求体包含错误参数|
|401 Unauthorized|*|当前请求需要验证身份验证，需要有效`Authorization`头|
|403 Forbidden|*|身份已验证，但没有相应权限|
|404 Not Found|*|URL所指资源不存在|
|405 Method Not Allowed|*|URL不支持当前请求的方法|
|406 Not Acceptable|POST/PUT/PATCH|服务器不支持请求体的`Content-Type`|
|500 Internal Sever Error|*|服务器出现了不可预料的错误|

注：* 所有方法

### 5. 分页与排序

当获取资源合集时，如 `GET /users`，返回资源数量可被分页和排序，分页和排序方式为 URL 参数。

|名称|类型|可选|说明|
|---|---|---|---|
|page|integer|是|页码，起始为0，默认为0|
|size|integer|是|每页数量，最小为1，最大100，默认为10|
|sort|string|是|排序方式，格式为`属性[,顺序]`，顺序可选，默认升序，可以为升序`asc`或降序`desc`，可指定多个`sort`来对多个字段排序

请求示例：以每页100的数量获取第二页，同时按年龄降序、名字升序排序

    GET /users?page=2&size=100&sort=age,desc&sort=nickname,asc

响应结果包含分页等信息。

|名称|类型|说明|
|---|---|---|
|content|array|结果数组|
|first|boolean|是否首页|
|last|boolean|是否尾页|
|size|integer|每页数量|
|number|integer|当前页码，从0开始|
|total_pages|integer|总页数|
|number_of_elements|integer|当前页结果数|
|total_elements|integer|总结果数|

响应示例：

    {
        "content": [
            {
                "id": 1,
                "nickname": "杨大神",
                "gender": "FEMALE",
                "age": 6,
                "occupation": "工程师",
                "created_at": "2015-10-07T19:08:38+08:00"
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 6. 过滤器

过滤器指使用`GET`方法获取资源集合时，对返回资源进行条件过滤的参数，使用 URL 参数来指定。

示例：获取性别为女性，昵称含有`咪咪`的用户

    GET /users?gender=FEMALE&nickname=咪咪

### 7. 日期时间

本文档使用日期采用 ISO 8601 格式，如 `2015-10-07T19:08:38+08:00` 表示 2015 年 10 月 07 日 19 时 08 分 38 秒，时区为 +8 区。

### 8. 认证

用户认证后，会获得 `JWT`，请求接口时，需将其加入 HTTP 请求 Header，Header 名为 `Authorization`。

### 9. 错误响应

请求发生错误时，服务端返回固定格式的响应体，类型为`application/json`，示例如下：

    {
        "path": "/users",
        "message": "users.creating.duplicated_phone",
        "timestamp": "2015-10-07T19:08:38+08:00"
    }

相应体说明

|名称|类型|说明|
|---|---|---|
|path|string|入口点|
|message|string|错误信息|
|timestamp|string|ISO 8601 时间戳|

错误信息对照表

|错误信息|说明|
|---|---|
|authentication.authenticating.failed|用户不存在或密码错误|
|users.creating.duplicated_phone|手机号码已被使用|

## 1. 用户 - User

用户实体属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|phone|string|✓||✓|手机号码|
|password|string|||✓|密码|
|nickname|string|✓|✓|✓|昵称|
|gender|string|✓|✓|✓|性别，"MALE"为男性, "FEMALE"为女性，"OTHER"为其他，null为未指定|
|age|integer|✓|✓|✓|年龄，null为未指定|
|occupation|string|✓|✓|✓|职业，null为未指定|
|created_at|string|✓|||创建时间|

### 1. 获取当前用户

获取当前用户

    GET /user

获得某个用户

	GET /users/:userId

响应头

    Status: 200 OK

响应体

    {
        "id": 1,
        "phone": "13912345678",
        "nickname": "guoqiang",
        "gender": "MALE",
        "age": 18,
        "occupation": "Engineer",
        "created_at": "2015-10-07T19:08:38+08:00"
    }

### 2. 编辑用户信息

    PATCH /user

请求体

    {
        "nickname": "nickname"
    }

响应头

    Status: 200 OK

响应体

    {
        "id": 1,
        "nickname": "nickname",
        "gender": "MALE",
        "age": 18,
        "occupation": "Engineer",
        "created_at": "2015-10-07T19:08:38+08:00"
    }

### 3. 获取用户头像

获取当前用户头像

    GET /user/avatar

获取指定用户头像

    GET /users/:userId/avatar

响应头

    Status: 200 OK

响应体

    <头像图片>

### 4. 编辑用户头像

    PUT /user/avatar

请求体

    <头像图片>

响应头

    Status: 200 OK

响应体

    <头像图片>

### 5. 获得所有用户

    GET /users

过滤器

|名称|类型|说明|
|---|---|---|
|phone|string|搜索手机|
|nickname|string|搜索名称|
|gender|string|按性别过滤|
|age|string|按年龄过滤|
|occupation|string|按职业过滤|

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "nickname": "杨大神",
                "gender": "FEMALE",
                "age": 6,
                "occupation": "工程师",
                "created_at": "2015-10-07T19:08:38+08:00"
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 6. 注册用户

    POST /users?code=<验证码>

请求体

    {
        "phone": "phone",
        "password": "password",
        "nickname": "nickname",
        "gender": "MALE",
        "age": 18,
        "occupation": "Engineer",
    }

响应头

    Status: 201 Created
    Location: http://sportsway.igalaxy.com.cn/users/1
    JWT: 8cddf35e-081d-4d4f-b4df-a0438b271b48

响应体

    {
        "id": 1,
        "nickname": "nickname",
        "gender": "MALE",
        "age": 18,
        "occupation": "Engineer",
        "created_at": "2015-10-07T19:08:38+08:00"
    }
    
### 7. 第三方账号注册/绑定

    PUT /users?code=<验证码>

请求体

    {
        "phone": "phone",
        "provider": "WEIXIN",
        "open_id": "<open_id>",
        "access_token": "<access_token>"
    }

请求体说明

|名称|类型|可选|说明|
|---|---|---|---|
|provider|string|否|第三方登录提供商，可以为"QQ"，"WEIXIN"，"WEIBO"|
|open_id|string|否|第三方用户ID|
|access_token|string|否|第三方用户访问令牌|

相应头

    Status: 200 OK
    Location: http://sportsway.igalaxy.com.cn/users/1
    JWT: 8cddf35e-081d-4d4f-b4df-a0438b271b48

### 8. 登录/第三方登录

    POST /authentication

请求体

    {
        "type": "secret",
        "phone": "phone",
        "password": "password"
    }

请求体说明

|名称|类型|可选|说明|
|---|---|---|---|
|type|string|否|登录类型，"secret"为密码登录，"oauth2"为第三方登录|
|phone|string|是|手机号码，"type"为"secret"时需要|
|password|string|是|密码，"type"为"secret"时需要|
|provider|string|是|第三方登录提供商，"type"为"oauth2"时需要，可以为"QQ"，"WEIXIN"，"WEIBO"|
|open_id|string|是|第三方用户ID，"type"为"oauth2"时需要|
|access_token|string|是|第三方用户访问令牌，"type"为"oauth2"时需要|

响应头

    Status: 200 OK
    JWT: 8cddf35e-081d-4d4f-b4df-a0438b271b48

## 2. 队伍 - Team

队伍属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|name|string|✓|✓|✓|名称|
|description|string|✓|✓|✓|描述|
|location|string|✓|✓|✓|所在地|
|created_at|string|✓|||创建时间|
|members|string|✓|||队伍成员数|

### 1. 获得所有队伍

    GET /teams

过滤器

|名称|类型|说明|
|---|---|---|
|name|string|按名称搜索|
|description|string|按描述搜索|
|location|string|按所在地搜索|

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "name": "银河战队",
                "description": "1",
                "location": "Beijing",
                "created_at": "2015-11-02T16:28:55Z",
                "members": 1
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 2. 获得队伍详情

    GET /teams/:teamId

响应头

    Status: 200 OK

响应体

    {
        "id": 1,
        "name": "Tiny Power",
        "captain": "Guoqiang",
        "description": "A demo team",
        "location": "Beijing",
        "created_at": "2008-01-14T04:33:35Z"
    }

### 3. 编辑队伍详情

    PATCH /teams/:teamId

请求头

    {
        "name": "Tiny Power",
        "captain": 1,
        "description": "A demo team",
        "location": "Beijing",
        "created_at": "2008-01-14T04:33:35Z"
    }

响应头

    Status: 200 OK

响应体

    {
        "id": 1,
        "name": "Tiny Power",
        "captain": "Guoqiang",
        "description": "A demo team",
        "location": "Beijing",
        "created_at": "2008-01-14T04:33:35Z"
    }

### 4. 获取队伍 Logo

    GET /teams/:teamId/logo

响应头

    Status: 200 OK

响应体

    <LOGO图片>

### 5. 编辑队伍 Logo

    PUT /teams/:teamId/logo

请求体

    <LOGO图片>

响应头

    Status: 200 OK

响应体

    <LOGO图片>

### 6. 创建队伍

    POST /teams

请求体

    {
        "name": "Tiny Power",
        "captain": 1,
        "description": "A demo team",
        "location": "Beijing",
    }

响应头

    Status: 201 Created
    Location: http://sportsway.igalaxy.com.cn/teams/1

响应体

    {
        "id": 1,
        "name": "Tiny Power",
        "captain": "Guoqiang",
        "description": "A demo team",
        "location": "Beijing",
        "created_at": "2008-01-14T04:33:35Z"
    }

### 7. 获得热门战队

    GET /hotteams

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "name": "银河战队",
                "description": "1",
                "location": "Beijing",
                "created_at": "2015-11-02T16:28:55Z",
                "members": 1
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

## 3. 成员关系 - Membership

队伍成员关系属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|team|object|✓|||用户信息，参见队伍属性|
|user|object|✓|||用户信息，参见用户属性|
|role|string|✓|||成员角色，"CAPTAIN"为队长，"MEMBER"为队员|
|status|string|✓|||成员状态，"ACTIVE"为有效，"INVITING"为已邀请，"APPLYING"为申请中|

### 1. 获得当前用户的队伍成员关系

    GET /user/teams

过滤器

|名称|类型|说明|
|---|---|---|
|role|string|按角色过滤，参见队伍成员关系|
|status|string|按状态过滤，参队伍成员关系|

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "role": "CAPTAIN",
                "status": "ACTIVE",
                "user": { ... },
                "team": { ... }
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 2. 获得指定用户的队伍成员关系

    GET /users/:userId/teams

过滤器

|名称|类型|说明|
|---|---|---|
|role|string|按角色过滤，参见队伍成员关系|
|status|string|按状态过滤，参见队伍成员关系|

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "role": "CAPTAIN",
                "status": "ACTIVE",
                "user": { ... },
                "team": { ... }
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 3. 获取指定队伍的成员关系

    GET /teams/:teamId/members

过滤器

|名称|类型|说明|
|---|---|---|
|role|string|按角色过滤，参见队伍成员关系|
|status|string|按状态过滤，参见队伍成员关系|

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "role": "CAPTAIN",
                "status": "ACTIVE",
                "user": { ... },
                "team": { ... }
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 4. 获取指定队伍中的指定成员的关系

    GET /teams/:teamId/members/:userId

响应头

    Status: 200 OK

响应体

    {
        "user": { ... },
        "team": { ... },
        "role": "MEMBER",
        "status": "ACTIVE"
    }

### 5. 邀请用户加入队伍（需要当前用户为队长）

    PUT /teams/:teamId/members/:userId

响应头

    Status: 200 OK

响应体

    {
        "user": { ... },
        "team": { ... },
        "role": "MEMBER",
        "status": "INVITING"
    }

### 6. 接受用户加入申请（需要当前用户为队长）

    PATCH /teams/:teamId/members/:userId

响应头

    Status: 200 OK

响应体

    {
        "user": { ... },
        "team": { ... },
        "role": "MEMBER",
        "status": "ACTIVE"
    }

### 7. 拒绝用户加入申请（需要当前用户为队长）

    DELETE /teams/:teamId/members/:userId

响应头

    Status: 204 No Content

### 8. 取消邀请队伍成员（需要当前用户为队长）

同上

### 9. 删除队伍成员（需要当前用户为队长）

同上

### 10. 转让队长（需要当前用户为队长）

    PUT /teams/:teamId/captain/:userId

响应头

    Status: 200 OK

响应体

    {
        "user": {
            "id": 1,
            "nickname": "nickname",
            "gender": "male",
            "age": 18,
            "occupation": "Engineer"
        },
        "role": "CAPTAIN",
        "status": "ACTIVE"
    }

### 11. 申请加入队伍

    PUT user/teams/:teamId

响应头

    Status: 200 OK

响应体

    {
        "user": {
            "id": 1,
            "nickname": "nickname",
            "gender": "male",
            "age": 18,
            "occupation": "Engineer"
        },
        "role": "MEMBER",
        "status": "APPLYING"
    }

### 11. 接受队伍邀请

    PATCH user/teams/:teamId

响应头

    Status: 200 OK

响应体

    {
        "user": {
            "id": 1,
            "nickname": "nickname",
            "gender": "male",
            "age": 18,
            "occupation": "Engineer"
        },
        "role": "MEMBER",
        "status": "ACTIVE"
    }

### 12. 拒绝队伍邀请

    DELETE /user/teams/:teamId

响应头

    Status: 204 No Content

### 13. 取消申请队伍

同上

### 14. 离开队伍

同上

## 4. 商户 - Merchant

商户属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|login|string|||✓|登录名|
|password|string|||✓|密码|
|name|string|✓||✓|名称|
|description|string|✓|✓|✓|简介|
|telephone|string|✓|✓|✓|电话|
|address|string|✓|✓|✓|地址|
|longitude|float|✓|✓|✓|位置纬度|
|latitude|float|✓|✓|✓|位置经度|
|items|array|✓|||商品数组，参见商品属性|
|created_at|string|✓|||创建时间|

### 1. 所有商户

    GET /merchants

排序

|属性|说明|
|---|---|
|price|按价格排序|
|distance|按距离排序，需要指定`longitude`，`latitude`|

响应头

    200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "name": "北环体育馆",
                "description": "",
                "telephone": "",
                "address": "",
                "longitude": "30.1230293848",
                "latitude": "129.0423481295",
                "items": [
                    ...
                ],
                "created_at": "2015-11-02T16:28:55Z"
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 2. 获取商家详情

    GET /merchants/:merchantId

响应头

    200 OK

响应体

    {
        "id": 1,
        "name": "北环体育馆",
        "description": "",
        "telephone": "",
        "address": "",
        "longitude": "30.1230293848",
        "latitude": "129.0423481295",
        "items": [],
        "created_at": "2015-11-02T16:28:55Z"
    }

### 3. 编辑商家详情

    PATCH /merchants/:merchantId

请求头

    {
        "longitude": "30.1230293848",
        "latitude": "129.0423481295",
        "created_at": "2015-11-02T16:28:55Z"
    }

响应头

    Status: 200 OK

响应体

    {
        "id": 1,
        "name": "北环体育馆",
        "description": "",
        "telephone": "",
        "address": "",
        "longitude": "30.1230293848",
        "latitude": "129.0423481295",
        "created_at": "2015-11-02T16:28:55Z"
    }

### 4. 获取商家 Logo

    GET /merchants/:merchantId/logo

响应头

    Status: 200 OK

响应体

    <LOGO图片>

### 5. 编辑商家 Logo

    PUT /merchants/:merchantId/logo

请求体

    <LOGO图片>

响应头

    Status: 200 OK

响应体

    <LOGO图片>

### 6. 注册商家

    POST /merchants

请求体

    {
        "name": "北环体育馆",
        "longitude": "30.1230293848",
        "latitude": "129.0423481295"
    }

响应头

    Status: 201 Created
    Location: http://www.ydh123.com.cn/merchants/1

响应体

    {
        "id": 1,
        "name": "北环体育馆",
        "description": "",
        "telephone": "",
        "address": "",
        "longitude": "30.1230293848",
        "latitude": "129.0423481295",
        "created_at": "2015-11-02T16:28:55Z"
    }

### 7. 商家商品统计

返回该商家近7日的商品统计信息

    GET /merchants/:merchantId/statistics

响应头

    Status: 200 OK

响应体

    [
        {
            "date": "2015-12-15",
            "full": 5,
            "half": 10
        },
        {
            "date": "2015-12-15",
            "full": 5,
            "half": 10
        },
        ...
    ]

响应体说明

|属性|类型|描述|
|---|---|---|
|statistics|array|可用商品数量数组|
|prices|object|最低价格|

### 8. 商家可用商品

返回该商家某日可用商品

    GET /merchants/:merchantId/items?date=2015-12-15

响应头

    Status: 200 OK

响应体

    {
        "content": [
            {
                "date": "2015-12-15",
                "time": 9,
                "items": {
                    "id": 1,
                    "name": "3号场全场",
                    ...
                },
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

响应体说明

|属性|类型|描述|
|---|---|---|
|date|string|日期|
|time|integer|小时|
|item|object|商品，参见商品属性|

## 5. 商品 - Item

商品属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|name|string|✓|✓|✓|名称|
|description|string|✓|✓|✓|描述|
|specification|string|✓|✓|✓|规格，"full"为全场，"half"为半场|
|prices|array|✓|✓|✓|价格数组，参见价格属性|
|sub_items|array|✓|||子商品数组，参见商品属性|
|on_sale|boolean|✓|✓|✓|是否上架|
|created_at|string|✓|||创建时间|
|updated_at|string|✓|||更新时间|

价格属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|time|integer|✓|||时，格式`9`，表示9:00至10:00|
|price|float|✓||✓|价格|

### 1. 所有商品

    GET /items

响应头

    200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "name": "3号场全场",
                "description": "",
                "specification": "FULL",
                "sub_items": [
                    {
                        "id": 2,
                        "name": "3号场半场A",
                        ...
                    },
                    {
                        "id": 3,
                        "name": "3号场半场B",
                        ...
                    }
                ],
                "prices": [
                    {
                        "time": 9,
                        "price": 30.0
                    },
                    {
                        "time": 10,
                        "price": 30.0
                    },
                    {
                        "time": 11,
                        "price": 30.0
                    },
                ],
                "created_at": "2015-11-02T16:28:55Z",
                "updated_at": "2015-11-02T16:28:55Z"
            },
            {
                "id": 2,
                "name": "3号场半场A",
                "description": "",
                "specification": "FULL",
                "sub_items": [],
                "prices": [
                    {
                        "time": 9,
                        "price": 30.0
                    },
                    {
                        "time": 10,
                        "price": 30.0
                    },
                    {
                        "time": 11,
                        "price": 30.0
                    },
                ],
                "created_at": "2015-11-02T16:28:55Z",
                "updated_at": "2015-11-02T16:28:55Z"
            }
        ],
        "total_pages": 1,
        "last": true,
        "total_elements": 1,
        "size": 10,
        "number": 0,
        "first": true,
        "sort": null,
        "number_of_elements": 1
    }

### 2. 获取商品详情

    GET /items/:itemId

响应头

    200 OK

响应体

    {
        "id": 2,
        "name": "3号场半场A",
        "description": "",
        "specification": "FULL",
        "sub_items": [],
        "prices": [
            {
                "time": 9,
                "price": 30.0
            },
            {
                "time": 10,
                "price": 30.0
            },
            {
                "time": 11,
                "price": 30.0
            },
        ],
        "created_at": "2015-11-02T16:28:55Z",
        "updated_at": "2015-11-02T16:28:55Z"
    }


### 3. 编辑商品详情

    PATCH /items/:itemId

请求头

    {
        "name": "3号场"
    }

响应头

    Status: 200 OK

响应体

    {
        "id": 2,
        "name": "3号场半场A",
        "description": "",
        "specification": "FULL",
        "sub_items": [],
        "prices": [
            {
                "time": 9,
                "price": 30.0
            },
            {
                "time": 10,
                "price": 30.0
            },
            {
                "time": 11,
                "price": 30.0
            },
        ],
        "created_at": "2015-11-02T16:28:55Z",
        "updated_at": "2015-11-02T16:28:55Z"
    }

### 4. 获取所有商品图片

    GET /items/:itemId/picture

响应头

    Status: 200 OK

响应体

    <图片>

### 5. 修改指定商品图片

    PUT /items/:itemId/picture

请求体

    <图片>

响应头

    Status: 200 OK

响应体

    <LOGO图片>

### 6. 创建商品

    POST /items

请求体

    {
        "name": "3号场"
    }

响应头

    Status: 201 Created
    Location: http://www.ydh123.com.cn/items/1

响应体

    {
        "id": 2,
        "name": "3号场半场A",
        "description": "",
        "specification": "FULL",
        "sub_items": [],
        "prices": [
            {
                "time": 9,
                "price": 30.0
            },
            {
                "time": 10,
                "price": 30.0
            },
            {
                "time": 11,
                "price": 30.0
            },
        ],
        "created_at": "2015-11-02T16:28:55Z",
        "updated_at": "2015-11-02T16:28:55Z"
    }

## 6. 订单 - Order

订单属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|user|object|✓|||用户信息，参见用户属性|
|items|array|✓||✓|订单项数组，参见订单项属性|
|status|string|✓|||订单状态，"CREATED"为已创建，"PENDING"为处理支付中，"PAID"为已支付, "IN_PROGRESS"为进行中, "COMPLETED"为已完成, "CANCELLED"为已取消, "REFUNDED"为已退款, "EXPIRED"为已过期|
|created_at|string|✓|||创建时间|
|updated_at|string|✓|||更新时间|

订单项属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|item|object|✓||✓|商品，参见商品属性|
|date|string|✓||✓|日期，格式`2015-12-01`|
|time|integer|✓||✓|时间，格式`9`|

### 1. 获得用户订单

    GET /user/orders

响应头

    200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "user": {
                    "id": 2,
                    "nickname": "2b",
                    "gender": "male",
                    "age": 18,
                    "occupation": "Hey man",
                    "created_at": "2015-11-05T16:00:43Z"
                },
                "items": [
                    {
                        "item": {
                            "id": 1,
                            "name": "3号场",
                            "description": "",
                            ...
                        },
                        "date": "2015-11-25",
                        "time": 9
                    }
                ],
                "status": "CREATED",
                "created_at": "2015-10-07T19:08:38+08:00",
                "updated_at": "2015-10-07T19:08:38+08:00"
            }
        ],
        "total_pages": 1,
        "total_elements": 1,
        "last": false,
        "sort": null,
        "first": true,
        "number_of_elements": 10,
        "size": 10,
        "number": 0
    }

### 2. 创建订单

    POST /orders

请求体

    {
        "items": [
            "item": {
                "id": 1
            },
            "date": "2015-11-25",
            "time": 9
        ]
    }

响应头

    201 Created

响应体

    {
        "id": 1,
        "user": {
            "id": 2,
            "nickname": "2b",
            "gender": "male",
            "age": 18,
            "occupation": "Hey man",
            "created_at": "2015-11-05T16:00:43Z"
        },
        "items": [
            {
                "item": {
                    "id": 1,
                    "name": "3号场",
                    "description": ""
                },
                "date": "2015-11-25",
                "time": 9
            }
        ],
        "status": "CREATED",
        "created_at": "2015-10-07T19:08:38+08:00",
        "updated_at": "2015-10-07T19:08:38+08:00"
    }

## 7. 评论 - Review (TODO)

## 10. 通知 - Notification

通知属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|id|integer|✓|||唯一 ID|
|created_at|integer|✓|||创建时间|
|unread|boolean|✓|||未读|
|type|string|✓|||通知类型|
|payload|object|✓|||通知负载，实体属性取决于通知类型|

通知类型说明

|通知类型|负载类型|说明|
|---|---|---|
|INVITED|队伍成员属性|你被邀请加入某个队伍|
|APPLIED|队伍成员属性|（队长通知）有用户申请加入你的队伍|
|ACCEPTED|队伍成员属性|你申请的队伍接受了你的申请|
|AGREED|队伍成员属性|（队长通知）你邀请的用户接受了你的邀请|
|REJECTED|队伍成员属性|你申请的队伍拒绝了你的申请|
|REFUSED|队伍成员属性|（队长通知）你邀请的用户拒绝了你的邀请|
|KICKED|队伍成员属性|你被踢出了某个队伍|
|LEFT|队伍成员属性|（队长通知）你的队伍成员离开了你的队伍|

### 1. 获取通知

    GET /user/notifications

过滤器

|名称|类型|说明|
|---|---|---|
|unread|boolean|按未读过滤|

响应头

    200 OK

响应体

    {
        "content": [
            {
                "id": 1,
                "created_at": "2015-10-07T19:08:38+08:00",
                "unread": true,
                "type": "INVITING",
                "payload": {
                    "role": "MEMBER",
                    "status": "INVITING",
                    "user":{
                        "id": 2,
                        "nickname": "2b",
                        "gender": "male",
                        "age": 18,
                        "occupation": "Hey man",
                        "created_at": "2015-11-05T16:00:43Z"
                    },
                    "team":{
                        "id": 42,
                        "name": "法拉利战队",
                        "description": "WIN",
                        "location": "北京",
                        "created_at": "2015-11-05T16:00:43Z",
                        "members": 2
                    }
                }
            }
        ],
        "total_pages": 1,
        "total_elements": 1,
        "last": false,
        "sort": null,
        "first": true,
        "number_of_elements": 10,
        "size": 10,
        "number": 0
    }

### 2. 获取某个通知

    GET /user/notifications/:notificationsId

响应头

    200 OK

响应体

    {
        "id": 1,
        "created_at": "2015-10-07T19:08:38+08:00",
        "unread": true,
        "type": "INVITING",
        "payload": {
            "role": "MEMBER",
            "status": "INVITING",
            "user":{
                "id": 2,
                "nickname": "2b",
                "gender": "male",
                "age": 18,
                "occupation": "Hey man",
                "created_at": "2015-11-05T16:00:43Z"
            },
            "team":{
                "id": 42,
                "name": "法拉利战队",
                "description": "WIN",
                "location": "北京",
                "created_at": "2015-11-05T16:00:43Z",
                "members": 2
            }
        }
    }

### 3. 标记已读

    PATCH /user/notifications/:notificationsId

响应头

    200 OK

## 11. 软件更新 - Update

软件更新信息属性

|属性|类型|可读|可写|创建时可写|描述|
|---|---|---|---|---|---|
|update|boolean|✓|||是否需要更新|
|forced|boolean|✓|||是否强制更新|
|version|string|✓|||最新版本号|
|size|string|✓|||软件大小|
|content|string|✓|||更新内容|
|url|string|✓|||下载链接|

### 1. 检测更新

    GET /ydh?system="ANDROID"&version="7"

请求参数说明

|属性|类型|描述|
|---|---|---|
|system|string|系统类型`ANDROID``IOS`|
|version|string|当前版本号，Android发送version code|

响应头

    200 OK

响应体

    {
        "update": True,
        "forced": True,
        "version": "2.0.0",
        "size": "13M",
        "content": "1.界面大更新\n2.添加赛事功能",
        "url": "http://www.ydh123.com/ydh.apk"
    }