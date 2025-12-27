# Nyanpass API 接口文档

> 版本: 1.0  
> 基础路径: `/api/v1`  
> 认证方式: JWT Token (Header: `Authorization`)

---

## 目录

- [通用说明](#通用说明)
- [认证模块 (Auth)](#认证模块-auth)
- [游客模块 (Guest)](#游客模块-guest)
- [用户模块 (User)](#用户模块-user)
- [转发规则模块 (Forward)](#转发规则模块-forward)
- [管理员模块 (Admin)](#管理员模块-admin)
- [系统模块 (System)](#系统模块-system)
- [附录：常量定义](#附录常量定义)

---

## 通用说明

### 请求格式

所有 POST/PUT/DELETE 请求的 Body 使用 JSON 格式：

```http
Content-Type: application/json
```

### 认证方式

登录成功后，将返回的 JWT Token 存储，后续请求在 Header 中携带：

```http
Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 通用响应格式

```json
{
  "code": 0,           // 0 表示成功，其他值表示错误
  "msg": "success",    // 错误信息（成功时可能为空）
  "data": {}           // 返回数据
}
```

### 分页参数

支持分页的接口可使用以下 Query 参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `page` | int | 页码，从 1 开始 |
| `size` | int | 每页数量 |
| `order` | string | 排序字段 |
| `desc` | int | 1=降序，0=升序 |
| `filter` | string | JSON 格式的过滤条件 |

---

## 认证模块 (Auth)

### 1. 获取验证码

获取人机验证码图片（点选验证码）。

**请求**

```http
GET /api/v1/auth/captcha/get
```

**响应示例**

```json
{
  "code": 0,
  "captcha_key": "captcha_xxx_key",
  "image_base64": "data:image/png;base64,iVBORw0KGgo...",
  "thumb_base64": "data:image/png;base64,iVBORw0KGgo..."
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| captcha_key | string | 验证码唯一标识 |
| image_base64 | string | 主图片 Base64 编码 |
| thumb_base64 | string | 缩略图 Base64 编码 |

---

### 2. 验证验证码

校验用户点选的验证码位置。

**请求**

```http
POST /api/v1/auth/captcha/check
Content-Type: application/x-www-form-urlencoded
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| key | string | 是 | 验证码 key |
| dots | string | 是 | 点击坐标，格式：x1,y1,x2,y2,... |

**请求示例**

```
key=captcha_xxx_key&dots=100,150,200,180,300,120
```

**响应示例**

```json
{
  "code": 0,
  "msg": "success"
}
```

---

### 3. 用户登录

用户登录获取 JWT Token。

**请求**

```http
POST /api/v1/auth/login
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名 |
| password | string | 是 | 密码 |

**请求示例**

```json
{
  "username": "admin",
  "password": "password123"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "success",
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE3MDk4MjQwMDB9.xxx"
}
```

**错误码**

| code | 说明 |
|------|------|
| 0 | 登录成功 |
| 1001 | 用户名或密码错误 |
| 1002 | 账户已被禁用 |

---

### 4. 用户登出

退出登录，使当前 Token 失效。

**请求**

```http
POST /api/v1/auth/logout
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "success"
}
```

---

### 5. 用户注册

新用户注册。

**请求**

```http
POST /api/v1/auth/register
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名 |
| password | string | 是 | 密码 |
| invite_code | string | 否 | 邀请码 |
| captcha_key | string | 否 | 验证码 key（开启验证码时必填） |

**请求示例**

```json
{
  "username": "newuser",
  "password": "password123",
  "invite_code": "ABC123",
  "captcha_key": "captcha_xxx"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "success",
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**错误码**

| code | 说明 |
|------|------|
| 0 | 注册成功 |
| 1003 | 用户名已存在 |
| 1004 | 邀请码无效 |
| 1005 | 验证码错误 |
| 1006 | 注册已关闭 |

---

## 游客模块 (Guest)

### 1. 获取公开 KV 配置

获取面向游客公开的配置信息。

**请求**

```http
GET /api/v1/guest/kv/{key}
GET /api/v1/{type}/kv/{key}  // type 可为 guest/user/admin
```

**路径参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| key | string | 配置键名 |
| type | string | 权限类型 (guest/user/admin) |

**常用 Key 值**

| Key | 权限要求 | 说明 |
|-----|----------|------|
| site_info | guest | 站点信息 |
| site_notice | admin | 站点公告 |
| invite_config | admin | 邀请配置 |
| payment_info | admin | 支付配置 |
| telegram-bot-config | admin | Telegram Bot 配置 |

**响应示例**

```json
{
  "code": 0,
  "data": "{\"title\":\"我的站点\",\"allow_register\":true,\"allow_single_tunnel\":false,\"allow_looking_glass\":true,\"register_policy\":0,\"register_captcha_policy\":0,\"diagnose_hide_ip\":0,\"theme_policy\":0}"
}
```

**site_info 结构说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| title | string | 站点标题 |
| allow_register | boolean | 是否允许注册 |
| allow_single_tunnel | boolean | 是否允许单端隧道 |
| allow_looking_glass | boolean | 是否允许 Looking Glass |
| register_policy | int | 注册策略 (0=无限制, 1=不允许邀请注册, 2=仅开放邀请注册) |
| register_captcha_policy | int | 验证码策略 (0=无, 1=交互认证) |
| diagnose_hide_ip | int | 诊断隐藏IP (0=不隐藏, 1=对非管理员隐藏, 2=对所有用户隐藏) |
| theme_policy | int | 主题策略 (0-3) |

---

### 2. Telegram Webhook

Telegram Bot 的 Webhook 回调地址（由 Telegram 服务器调用）。

**请求**

```http
POST /api/v1/guest/telegram/webhook
Content-Type: application/json
```

**说明**

此接口由 Telegram 服务器自动调用，用于接收用户发送给 Bot 的消息。

需要在管理后台配置 Telegram Bot Token 和 Webhook URL 后才能使用。

---

## 用户模块 (User)

### 1. 获取用户信息

获取当前登录用户的详细信息。

**请求**

```http
GET /api/v1/user/info
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "id": 1,
    "username": "admin",
    "admin": true,
    "balance": 100.00,
    "traffic_used": 1073741824,
    "traffic_limit": 10737418240,
    "expire": 1735689600,
    "max_rules": 100,
    "speed_limit": 0,
    "allow_device": false,
    "group_id": 1,
    "invite_code": "ABC123",
    "inviter_id": 0,
    "telegram_id": 0,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-15T12:00:00Z"
  }
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 用户 ID |
| username | string | 用户名 |
| admin | boolean | 是否管理员 |
| balance | float | 账户余额 |
| traffic_used | int | 已用流量 (bytes) |
| traffic_limit | int | 流量限制 (bytes) |
| expire | int | 过期时间 (Unix 时间戳) |
| max_rules | int | 最大规则数 |
| speed_limit | int | 速度限制 (bytes/s) |
| allow_device | boolean | 是否允许自建设备 |
| group_id | int | 用户组 ID |
| invite_code | string | 邀请码 |
| inviter_id | int | 邀请人 ID |
| telegram_id | int | 绑定的 Telegram ID |

---

### 2. 刷新用户 Token

续期当前 Token。

**请求**

```http
POST /api/v1/user/renew
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": "new_jwt_token..."
}
```

---

### 3. Telegram 绑定/解绑

**请求**

```http
POST /api/v1/user/telegram/bind
POST /api/v1/user/telegram/bind?unbind=1  // 解绑
Authorization: {token}
```

**Query 参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| unbind | int | 1=解绑，不传或0=绑定 |

**绑定响应示例**

```json
{
  "code": 0,
  "msg": "请向 Bot 发送以下命令完成绑定: /bindxxxxxxxx"
}
```

**解绑响应示例**

```json
{
  "code": 0,
  "msg": "解绑成功"
}
```

---

### 4. 重置密码

**请求**

```http
POST /api/v1/user/reset_password
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| current_password | string | 是 | 当前密码 |
| new_password | string | 否 | 新密码（留空则随机生成） |

**请求示例**

```json
{
  "current_password": "current123",
  "new_password": "new456"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "success",
  "data": "random_password_if_empty"
}
```

**说明**

- 如果 `new_password` 留空，系统会随机生成密码并在响应中返回

---

### 5. 更新用户列

更新用户的单个字段。

**请求**

```http
POST /api/v1/user/update_column
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| column | string | 是 | 字段名 |
| value | any | 是 | 新值 |

**请求示例**

```json
{
  "column": "invite_code",
  "value": "NEWCODE"
}
```

---

### 6. 获取用户统计

**请求**

```http
GET /api/v1/user/statistic
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "traffic_today": 1073741824,
    "traffic_yesterday": 2147483648
  }
}
```

---

### 7. 获取设备组列表

获取用户可用的设备组列表。

**请求**

```http
GET /api/v1/user/devicegroup
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "我的出口",
      "type": "DeviceGroupType_OutboundByUser",
      "token": "abc123...",
      "ratio": "1",
      "traffic_used": 1073741824,
      "traffic_limit": 0,
      "display_num": 1,
      "config": "{\"protocol\":\"ws\"}",
      "note": "备注",
      "show_order": 0
    }
  ]
}
```

---

### 8. 创建设备组

**请求**

```http
PUT /api/v1/user/devicegroup
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 设备组名称 |
| config | string | 否 | 配置 JSON |
| note | string | 否 | 备注 |

**请求示例**

```json
{
  "name": "我的出口节点",
  "config": "{\"protocol\":\"ws\"}",
  "note": "个人使用"
}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "id": 5,
    "name": "我的出口节点",
    "type": "DeviceGroupType_OutboundByUser",
    "token": "generated_token_xxx"
  }
}
```

---

### 9. 更新设备组

**请求**

```http
POST /api/v1/user/devicegroup/{id}
Authorization: {token}
Content-Type: application/json
```

**路径参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| id | int | 设备组 ID |

**请求示例**

```json
{
  "name": "新名称",
  "config": "{\"protocol\":\"http\"}",
  "note": "更新备注"
}
```

---

### 10. 删除设备组

**请求**

```http
DELETE /api/v1/user/devicegroup
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ids | int[] | 是 | 要删除的设备组 ID 数组 |

**请求示例**

```json
{
  "ids": [1, 2, 3]
}
```

---

### 11. 重置设备组 Token

**请求**

```http
POST /api/v1/user/devicegroup/{id}/reset_token
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "token": "new_token_xxx"
  }
}
```

---

### 12. 重置设备组流量

**请求**

```http
POST /api/v1/user/devicegroup/reset_traffic
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

### 13. Looking Glass

执行 Looking Glass 网络检测（需要站点启用 Looking Glass 功能）。

**请求**

```http
POST /api/v1/user/devicegroup/looking_glass
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| handle | string | 是 | 设备句柄（从 node_status 获取） |
| method | string | 是 | 检测方法（目前仅支持 ping） |
| target | string | 是 | 目标 IP 或域名 |

**请求示例**

```json
{
  "handle": "server_handle_xxx",
  "method": "ping",
  "target": "8.8.8.8"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.\n64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=1.23 ms\n..."
}
```

**说明**

- `handle` 可从 `/api/v1/system/node/status` 返回的服务器数据中获取
- 检测结果以文本形式返回在 `msg` 字段中

---

### 14. 获取支付信息

**请求**

```http
GET /api/v1/user/shop/payment_info
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "min_deposit": 10,
    "gateways": [
      {
        "name": "epay",
        "type": "epay",
        "enable": true,
        "fee_ratio": 0
      },
      {
        "name": "epusdt",
        "type": "epusdt",
        "enable": true,
        "fee_ratio": 0
      }
    ]
  }
}
```

---

### 15. 查询兑换码

**请求**

```http
GET /api/v1/user/shop/redeem?code={code}
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "code": "REDEEM123",
    "plan_name": "月付套餐",
    "traffic": 10737418240,
    "days": 30,
    "used": false
  }
}
```

---

### 16. 使用兑换码

**请求**

```http
POST /api/v1/user/shop/redeem?code={code}
Authorization: {token}
```

---

### 17. 获取套餐列表

**请求**

```http
GET /api/v1/user/shop/plan
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "月付套餐",
      "type": "PlanType_Month",
      "price": 10.00,
      "traffic": 107374182400,
      "speed_limit": 0,
      "max_rules": 10,
      "multiple": 1,
      "enable_for_gid": "1,2",
      "show_order": 0
    }
  ]
}
```

---

### 18. 购买套餐

**请求**

```http
POST /api/v1/user/shop/purchase
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| plan_id | int | 是 | 套餐 ID |

**请求示例**

```json
{
  "plan_id": 1
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "购买成功"
}
```

---

### 19. 充值

创建充值订单并获取支付链接。

**请求**

```http
POST /api/v1/user/shop/deposit
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| gateway_name | string | 是 | 支付网关名称 |
| amount | number | 是 | 充值金额 |

**请求示例**

```json
{
  "gateway_name": "epay",
  "amount": 100
}
```

**响应示例（跳转支付）**

```json
{
  "code": 0,
  "data": {
    "qr": false,
    "url": "https://pay.example.com/xxx"
  }
}
```

**响应示例（扫码支付）**

```json
{
  "code": 0,
  "data": {
    "qr": true,
    "url": "weixin://wxpay/bizpayurl?pr=xxx"
  }
}
```

**说明**

- 当 `qr=true` 时，前端应生成二维码供用户扫描
- 当 `qr=false` 时，前端应跳转到 `url` 进行支付
- 充值金额不能低于站点设置的最小充值金额

---

### 20. 获取充值订单支付信息

获取未完成充值订单的支付链接（用于继续支付）。

**请求**

```http
GET /api/v1/user/shop/get_deposit/{orderNo}
Authorization: {token}
```

**路径参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| orderNo | string | 订单号 |

**响应示例（跳转支付）**

```json
{
  "code": 0,
  "data": {
    "qr": false,
    "url": "https://pay.example.com/xxx"
  }
}
```

**响应示例（扫码支付）**

```json
{
  "code": 0,
  "data": {
    "qr": true,
    "url": "weixin://wxpay/bizpayurl?pr=xxx"
  }
}
```

**说明**

- 当 `qr=true` 时，前端应生成二维码供用户扫描
- 当 `qr=false` 时，前端应跳转到 `url` 进行支付

---

### 21. 获取订单列表

**请求**

```http
GET /api/v1/user/shop/order?page=1&size=10
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "count": 50,
  "data": [
    {
      "id": 1,
      "order_no": "ORDER123456",
      "type": "OrderType_DepositToBalance",
      "status": "OrderStatus_Finished",
      "amount": 100.00,
      "created_at": "2024-01-15T12:00:00Z"
    }
  ]
}
```

---

### 22. 获取推广日志

**请求**

```http
GET /api/v1/user/aff/log?page=1&size=10
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "count": 10,
  "data": [
    {
      "id": 1,
      "type": "AffiliateLogType_Commission",
      "amount": 10.00,
      "note": "用户 xxx 消费产生佣金",
      "created_at": "2024-01-15T12:00:00Z"
    }
  ]
}
```

---

### 23. 获取推广配置

**请求**

```http
GET /api/v1/user/aff/config
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "enable": true,
    "commission_rate": "0.1",
    "cycle": false,
    "force_bind_telegram": false
  }
}
```

---

### 24. 推广佣金转余额

**请求**

```http
POST /api/v1/user/aff/deposit
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "amount": 50
}
```

---

### 25. 获取通知设置

获取当前用户的推送通知设置。

**请求**

```http
GET /api/v1/user/notification/settings
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "msg_type": "income",
      "channel": 0,
      "mode": 2,
      "list": ""
    },
    {
      "msg_type": "updown",
      "channel": 0,
      "mode": 1,
      "list": "1,2,3"
    }
  ]
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| msg_type | string | 消息类型 (income=收款信息, updown=设备离线与恢复) |
| channel | int | 推送渠道 (0=Telegram) |
| mode | int | 模式 (0=不接收, 1=白名单, 2=黑名单/接收) |
| list | string | 白名单/黑名单 ID 列表，逗号分隔 |

---

### 26. 更新通知设置

**请求**

```http
PUT /api/v1/user/notification/settings
Authorization: {token}
Content-Type: application/json
```

**请求示例**

```json
[
  {
    "msg_type": "income",
    "channel": 0,
    "mode": 2,
    "list": ""
  },
  {
    "msg_type": "updown",
    "channel": 0,
    "mode": 1,
    "list": "1,2,3"
  }
]
```

**说明**

- 需要绑定 Telegram 后才能接收推送
- `updown` 类型支持白名单/黑名单模式，list 中填写设备组 ID
- `income` 类型仅支持接收/不接收模式

---

## 转发规则模块 (Forward)

> 注意：转发规则接口分为用户接口和管理员接口两套
> - 用户接口：`/api/v1/user/forward/...`
> - 管理员接口：`/api/v1/admin/user/{userId}/forward/...`

### 1. 获取转发规则列表

**请求**

```http
GET /api/v1/user/forward?page=1&size=20
GET /api/v1/admin/user/{userId}/forward?page=1&size=20
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "count": 50,
  "data": [
    {
      "id": 1,
      "name": "我的转发规则",
      "device_group_in": 1,
      "device_group_out": 2,
      "listen_port": 20000,
      "status": "ForwardRuleStatus_Normal",
      "traffic_used": 1073741824,
      "traffic_limit": 0,
      "paused": false,
      "config": "{\"dest\":[\"1.2.3.4:8080\"],\"dest_policy\":\"random\"}",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-15T12:00:00Z"
    }
  ]
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 规则 ID |
| name | string | 规则名称 |
| device_group_in | int | 入口设备组 ID |
| device_group_out | int | 出口设备组 ID (0=直接转发) |
| listen_port | int | 监听端口 |
| status | string | 规则状态 |
| traffic_used | int | 已用流量 (bytes) |
| traffic_limit | int | 流量限制 (bytes) |
| paused | boolean | 是否暂停 |
| config | string | 配置 JSON |

**config 结构**

```json
{
  "dest": ["1.2.3.4:8080", "5.6.7.8:9090"],
  "dest_policy": "random",
  "accept_proxy_protocol": 0,
  "proxy_protocol": 0,
  "speed_limit": 0,
  "ip_limit": 0,
  "connection_limit": 0
}
```

---

### 2. 创建转发规则

**请求**

```http
PUT /api/v1/user/forward
PUT /api/v1/admin/user/{userId}/forward
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 规则名称 |
| device_group_in | int | 是 | 入口设备组 ID |
| device_group_out | int | 是 | 出口设备组 ID (0=直接转发) |
| listen_port | int | 否 | 监听端口，留空随机分配 |
| config | string | 是 | 配置 JSON |

**请求示例**

```json
{
  "name": "游戏加速",
  "device_group_in": 1,
  "device_group_out": 2,
  "listen_port": 20000,
  "config": "{\"dest\":[\"game.server.com:7777\"],\"dest_policy\":\"random\"}"
}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "id": 10,
    "name": "游戏加速",
    "listen_port": 20000,
    "status": "ForwardRuleStatus_Unsync"
  }
}
```

---

### 3. 更新转发规则

**请求**

```http
POST /api/v1/user/forward/{id}
POST /api/v1/admin/user/{userId}/forward/{id}
Authorization: {token}
Content-Type: application/json
```

**请求示例**

```json
{
  "name": "新名称",
  "device_group_out": 3,
  "paused": false,
  "config": "{\"dest\":[\"new.server.com:8080\"]}"
}
```

---

### 4. 删除转发规则

**请求**

```http
DELETE /api/v1/user/forward
DELETE /api/v1/admin/user/{userId}/forward
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

### 5. 批量创建规则

**请求**

```http
POST /api/v1/user/forward/batch_create
POST /api/v1/admin/user/{userId}/forward/batch_create
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| device_group_in | int | 是 | 入口设备组 ID |
| device_group_out | int | 否 | 出口设备组 ID |
| content | string | 是 | 批量规则内容，一行一个 |
| config | string | 否 | JSON 配置覆盖（不包含 dest） |

**content 格式说明**

每行一条规则，支持多种格式：
- `监听端口|目标地址:端口` - 如 `20001|1.2.3.4:8080`
- `规则名称|监听端口|目标地址:端口` - 如 `游戏服务器|20001|1.2.3.4:8080`

**请求示例**

```json
{
  "device_group_in": 1,
  "device_group_out": 2,
  "content": "游戏1|20001|server1.com:8080\n游戏2|20002|server2.com:8080\n游戏3|20003|server3.com:8080"
}
```

**使用 JSON 设置覆盖配置**

```json
{
  "device_group_in": 1,
  "device_group_out": 2,
  "content": "20001|server1.com:8080\n20002|server2.com:8080",
  "config": "{\"dest_policy\":\"round\",\"proxy_protocol\":1}"
}
```

---

### 6. 批量修改规则

根据端口匹配已有规则，修改目标地址。

**请求**

```http
POST /api/v1/user/forward/batch_change
POST /api/v1/admin/user/{userId}/forward/batch_change
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| device_group_in | int | 是 | 入口设备组 ID（用于匹配规则） |
| content | string | 是 | 批量修改内容，一行一个 |

**说明**

选定一个入口后，后端将根据批量规则中的端口，匹配该入口已有的规则，并修改其目标地址。

**请求示例**

```json
{
  "device_group_in": 1,
  "content": "20001|new-server1.com:8080\n20002|new-server2.com:8080"
}
```

---

### 7. 批量更新规则

批量更新多条规则的指定字段。

**请求**

```http
POST /api/v1/user/forward/batch_update
POST /api/v1/admin/user/{userId}/forward/batch_update
Authorization: {token}
Content-Type: application/json
```

**请求参数**

数组格式，每个元素包含：

| 参数 | 类型 | 说明 |
|------|------|------|
| ids | int[] | 规则 ID 数组 |
| column | string | 要更新的字段名 |
| value | any | 新值 |

**请求示例**

```json
[
  {
    "ids": [1, 2, 3],
    "column": "device_group_out",
    "value": 5
  },
  {
    "ids": [1, 2, 3],
    "column": "paused",
    "value": true
  }
]
```

---

### 8. 搜索转发规则

**请求**

```http
POST /api/v1/user/forward/search_rules?page=1&size=20
POST /api/v1/admin/user/{userId}/forward/search_rules
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| name | string | 规则名称（模糊匹配） |
| gid_in | int | 入口设备组 ID |
| gid_out | int | 出口设备组 ID |
| listen_port | int | 监听端口（精确匹配） |
| dest | string | 目标地址（模糊匹配） |

**请求示例**

```json
{
  "name": "游戏",
  "gid_in": 1,
  "listen_port": 20000
}
```

---

### 9. 重置规则流量

**请求**

```http
POST /api/v1/user/forward/reset_traffic
POST /api/v1/admin/user/{userId}/forward/reset_traffic
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

### 10. 诊断转发规则

检测转发规则的连通性。

**请求**

```http
POST /api/v1/user/forward/{id}/diagnose
POST /api/v1/admin/user/{userId}/forward/{id}/diagnose
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "inbound": {
      "name": "香港入口",
      "status": "online",
      "ip": "1.2.3.4"
    },
    "outbound": {
      "name": "日本出口",
      "status": "online",
      "ip": "5.6.7.8"
    },
    "target": {
      "address": "target.server.com:8080",
      "reachable": true,
      "latency": 50
    }
  }
}
```

---

## 管理员模块 (Admin)

> 所有管理员接口需要管理员权限 (`userInfo.admin === true`)

### KV 配置管理

#### 1. 设置 KV 配置

**请求**

```http
PUT /api/v1/admin/kv/{key}
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "value": "{\"title\":\"站点名称\",\"allow_register\":true}"
}
```

**常用 Key 及数据结构**

##### site_info - 站点信息

```json
{
  "title": "站点名称",
  "allow_register": true,
  "allow_single_tunnel": false,
  "allow_looking_glass": true,
  "register_policy": 0,
  "register_captcha_policy": 0,
  "diagnose_hide_ip": 0,
  "theme_policy": 0,
  "transparent_theme_bg_desktop": "",
  "transparent_theme_bg_mobile": ""
}
```

##### site_notice - 站点公告

```json
"公告内容，以 < 开头则显示为 HTML"
```

##### invite_config - 邀请配置

```json
{
  "enable": true,
  "commission_rate": "0.1",
  "cycle": false,
  "force_bind_telegram": false
}
```

##### payment_info - 支付配置

```json
{
  "min_deposit": 10,
  "gateways": [
    {
      "name": "支付宝",
      "type": "epay",
      "enable": true,
      "url": "https://pay.example.com/submit.php",
      "pid": "1001",
      "secret": "your_secret_key",
      "callback_host": "",
      "fee_ratio": 0
    }
  ]
}
```

**支付网关类型**

| type | 说明 |
|------|------|
| epay | 易支付 |
| epusdt | USDT 支付 |
| tokenpay | TokenPay |
| cyber | Cyber 支付 |
| cryptomus | Cryptomus |

##### telegram-bot-config - Telegram Bot 配置

```json
{
  "enable": true,
  "token": "123456:ABC-DEF...",
  "webhook_url": "https://your-domain.com/api/v1/guest/telegram/webhook"
}
```

---

### 用户管理

#### 2. 获取用户列表

**请求**

```http
GET /api/v1/admin/user?page=1&size=20
Authorization: {token}
```

**Query 参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| page | int | 页码 |
| size | int | 每页数量 |
| order | string | 排序字段 |
| desc | int | 1=降序 |
| filter | string | 过滤条件 JSON |

**响应示例**

```json
{
  "code": 0,
  "count": 100,
  "data": [
    {
      "id": 1,
      "username": "user1",
      "admin": false,
      "balance": 50.00,
      "traffic_used": 1073741824,
      "traffic_limit": 10737418240,
      "expire": 1735689600,
      "max_rules": 10,
      "group_id": 1,
      "disabled": false,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

---

#### 3. 创建用户

管理员创建新用户，密码由系统随机生成。

**请求**

```http
PUT /api/v1/admin/user
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 用户名 |

**请求示例**

```json
{
  "username": "newuser"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "添加用户成功，密码: Ab3xY9zK"
}
```

**说明**

- 创建成功后，初始密码会在响应消息中返回
- 用户创建后默认 group_id=0，无套餐

---

#### 4. 更新用户

**请求**

```http
POST /api/v1/admin/user/{id}
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| banned | boolean | 是否封禁 |
| admin | boolean | 是否管理员 |
| group_id | int | 用户组 ID（0=无套餐） |
| traffic_used | int | 已用流量 (bytes) |
| traffic_enable | int | 可用流量 (bytes) |
| expire | int | 过期时间 (Unix 时间戳) |
| max_rules | int | 最大规则数 |
| speed_limit | int | 速度限制 (bytes/s) |
| ip_limit | int | IP 限制 |
| connection_limit | int | 连接数限制 |
| balance | float | 钱包余额 |
| note | string | 备注 |
| update_traffic | boolean | 是否更新流量字段 |
| update_balance | boolean | 是否更新余额字段 |
| plan_id | int | 套餐 ID |
| calc_expire | boolean | 是否根据套餐计算过期时间 |
| invite_config | string | 邀请配置 JSON（空=跟随站点） |

**请求示例**

```json
{
  "banned": false,
  "admin": false,
  "group_id": 1,
  "traffic_used": 0,
  "traffic_enable": 107374182400,
  "update_traffic": true,
  "expire": 1767225600,
  "max_rules": 50,
  "speed_limit": 0,
  "ip_limit": 0,
  "connection_limit": 0,
  "balance": 100,
  "update_balance": true,
  "note": "VIP用户"
}
```

**说明**

- 更新流量时需设置 `update_traffic: true`
- 更新余额时需设置 `update_balance: true`
- 更改套餐并重新计算到期时间时设置 `calc_expire: true`

---

#### 5. 删除用户

**请求**

```http
DELETE /api/v1/admin/user
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

#### 6. 删除未使用用户

删除从未创建过规则的用户。

**请求**

```http
DELETE /api/v1/admin/user/delete_unused
Authorization: {token}
```

---

#### 7. 删除未使用规则

删除无效的转发规则。

**请求**

```http
DELETE /api/v1/admin/user/delete_unused_rules
Authorization: {token}
```

---

#### 8. 重置用户密码 (管理员)

管理员可以直接重置用户密码，无需原密码。

**请求**

```http
POST /api/v1/admin/user/{userId}/reset_password
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| new_password | string | 否 | 新密码（留空则随机生成） |

**请求示例**

```json
{}
```

或指定新密码：

```json
{
  "new_password": "newpassword123"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "success",
  "data": "generated_random_password"
}
```

---

### 设备组管理

#### 9. 获取设备组列表

**请求**

```http
GET /api/v1/admin/devicegroup
GET /api/v1/admin/devicegroup?uid={userId}  // 获取指定用户的设备组
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "香港入口",
      "type": "DeviceGroupType_Inbound",
      "token": "abc123...",
      "enable_for_gid": "1,2,3",
      "ratio": "1",
      "port_range": "10000-50000",
      "connect_host": "hk.example.com",
      "traffic_used": 10737418240,
      "display_num": 2,
      "display_protocol": "ws",
      "hide_status": 0,
      "allowed_out": "",
      "allowed_in": "",
      "config": "{}",
      "note": "备注",
      "show_order": 0
    }
  ]
}
```

---

#### 10. 创建设备组

**请求**

```http
PUT /api/v1/admin/devicegroup
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 名称 |
| type | string | 是 | 类型 |
| enable_for_gid | string | 否 | 允许使用的用户组 ID，逗号分隔 |
| ratio | string | 否 | 倍率 |
| port_range | string | 否 | 端口范围 (入口) |
| connect_host | string | 否 | 连接地址 (入口展示用) |
| allowed_out | string | 否 | 限制出口 (入口) |
| allowed_in | string | 否 | 限制入口 (出口) |
| down_sec | int | 否 | 负载下线时间 (出口) |
| fallback_group | int | 否 | 故障转移组 (出口) |
| hide_status | int | 否 | 在探针中隐藏 |
| config | string | 否 | 配置 JSON |
| note | string | 否 | 备注 |

**入口设备组请求示例**

```json
{
  "name": "香港入口",
  "type": "DeviceGroupType_Inbound",
  "enable_for_gid": "1,2,3",
  "ratio": "1",
  "port_range": "10000-50000",
  "connect_host": "hk.example.com\n1.2.3.4",
  "config": "{\"direct_policy\":0,\"udp_smart_bind\":false}",
  "note": "香港入口节点"
}
```

**出口设备组请求示例**

```json
{
  "name": "日本出口",
  "type": "DeviceGroupType_OutboundBySite",
  "enable_for_gid": "1,2,3",
  "ratio": "1.5",
  "down_sec": 60,
  "config": "{\"protocol\":\"ws\"}",
  "note": "日本出口节点"
}
```

**链式出口请求示例**

```json
{
  "name": "香港->日本",
  "type": "DeviceGroupType_ChainOutbound",
  "enable_for_gid": "1,2,3",
  "ratio": "2",
  "config": "{\"chain\":[{\"group_id\":1},{\"group_id\":2}]}",
  "note": "链式转发"
}
```

---

#### 11. 更新设备组

**请求**

```http
POST /api/v1/admin/devicegroup/{id}
Authorization: {token}
Content-Type: application/json
```

---

#### 12. 删除设备组

**请求**

```http
DELETE /api/v1/admin/devicegroup
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

#### 13. 重置设备组 Token

**请求**

```http
POST /api/v1/admin/devicegroup/{id}/reset_token
Authorization: {token}
```

---

#### 14. 重置设备组流量

**请求**

```http
POST /api/v1/admin/devicegroup/reset_traffic
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

#### 15. 管理员设备组排序

**请求**

```http
POST /api/v1/admin/devicegroup/reorder
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [3, 1, 2],
  "show_order": [0, 1, 2]
}
```

---

#### 16. 用户设备组排序

用户对自己的单端隧道设备组进行排序。

**请求**

```http
POST /api/v1/user/devicegroup/reorder
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [3, 1, 2],
  "show_order": [0, 1, 2]
}
```

---

### 用户组管理

#### 16. 获取用户组列表

**请求**

```http
GET /api/v1/admin/usergroup
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "id": 0,
      "name": "",
      "user_count": 50,
      "show_order": 0
    },
    {
      "id": 1,
      "name": "普通用户",
      "user_count": 100,
      "show_order": 1
    },
    {
      "id": 2,
      "name": "VIP用户",
      "user_count": 20,
      "show_order": 2
    }
  ]
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 用户组 ID |
| name | string | 用户组名称 |
| user_count | int | 该组用户数量 |
| show_order | int | 显示顺序 |

---

#### 17. 创建用户组

用户组主要用于命名和分组展示，实际权限由套餐和设备组配置决定。

**请求**

```http
PUT /api/v1/admin/usergroup
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 用户组名称 |

**请求示例**

```json
{
  "name": "VIP用户"
}
```

**说明**

- 用户组 #0 是基础用户组，代表未购买套餐
- 删除用户组不会影响已分配该组的用户
- 用户可以看到自己所在分组的名称

---

#### 18. 更新用户组

**请求**

```http
POST /api/v1/admin/usergroup/{id}
Authorization: {token}
Content-Type: application/json
```

---

#### 19. 删除用户组

**请求**

```http
DELETE /api/v1/admin/usergroup
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2]
}
```

---

#### 20. 用户组排序

**请求**

```http
POST /api/v1/admin/usergroup/reorder
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [3, 1, 2],
  "show_order": [0, 1, 2]
}
```

---

### 套餐管理

#### 20. 获取套餐列表

**请求**

```http
GET /api/v1/admin/shop/plan
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "月付套餐",
      "type": "PlanType_Month",
      "price": 10.00,
      "traffic": 107374182400,
      "speed_limit": 0,
      "max_rules": 10,
      "multiple": 1,
      "enable_for_gid": "1,2",
      "show_order": 0
    }
  ]
}
```

---

#### 21. 创建套餐

**请求**

```http
PUT /api/v1/admin/shop/plan
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 套餐名称 |
| type | string | 是 | 套餐类型 |
| price | float | 是 | 价格 |
| traffic | int | 是 | 流量 (bytes) |
| group_id | int | 否 | 分配用户组 ID（默认1） |
| max_rules | int | 否 | 最大规则数 |
| speed_limit | int | 否 | 速度限制 (bytes/s) |
| ip_limit | int | 否 | IP 限制 |
| connection_limit | int | 否 | 连接数限制 |
| multiple | int | 否 | 时长倍数（天/月，默认1） |
| hide | boolean | 否 | 是否隐藏 |
| desc | string | 否 | 说明文字 |

**请求示例**

```json
{
  "name": "月付 100G",
  "type": "PlanType_Month",
  "price": 10.00,
  "traffic": 107374182400,
  "group_id": 1,
  "max_rules": 10,
  "speed_limit": 0,
  "ip_limit": 0,
  "connection_limit": 0,
  "multiple": 1,
  "hide": false,
  "desc": "适合轻度用户"
}
```

---

#### 22. 更新套餐

**请求**

```http
POST /api/v1/admin/shop/plan/{id}
Authorization: {token}
Content-Type: application/json
```

---

#### 23. 推送套餐更新给用户

将套餐属性推送给持有此套餐的所有用户（批量更新用户属性）。

**请求**

```http
POST /api/v1/admin/shop/plan/{id}/push
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| update_traffic | boolean | 否 | 更新可用流量 |
| update_group | boolean | 否 | 更新用户组 |
| update_max_rules | boolean | 否 | 更新最大规则数 |
| update_limits | boolean | 否 | 更新限速与连接限制 |

**请求示例**

```json
{
  "update_traffic": true,
  "update_group": false,
  "update_max_rules": true,
  "update_limits": true
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "更新了 50 个用户"
}
```

---

#### 24. 删除套餐

**请求**

```http
DELETE /api/v1/admin/shop/plan
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2]
}
```

---

#### 25. 套餐排序

**请求**

```http
POST /api/v1/admin/shop/plan/reorder
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [3, 1, 2],
  "show_order": [0, 1, 2]
}
```

---

### 订单管理

#### 25. 获取订单列表

**请求**

```http
GET /api/v1/admin/shop/order?page=1&size=20
Authorization: {token}
```

---

#### 26. 订单手动记账

手动为用户记账（余额充值或扣除）。

**请求**

```http
POST /api/v1/admin/shop/order/accounting
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| uid | int | 是 | 用户 ID |
| amount | float | 是 | 金额（正数=充值，负数=扣除） |
| message | string | 否 | 订单信息/备注 |

**请求示例**

```json
{
  "uid": 1,
  "amount": 100.00,
  "message": "手动充值"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "记账成功"
}
```

**说明**

- 金额会直接计入用户钱包余额
- 负数金额代表扣除余额
- 若扣除后余额为负数，操作将失败

---

#### 27. 删除订单

**请求**

```http
DELETE /api/v1/admin/shop/order
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

#### 28. 手动回调订单

手动触发订单支付回调。

**请求**

```http
POST /api/v1/admin/shop/order/{id}/manual_callback
Authorization: {token}
```

---

### 兑换码管理

#### 29. 获取兑换码列表

**请求**

```http
GET /api/v1/admin/shop/redeem?page=1&size=20
Authorization: {token}
```

---

#### 30. 导入兑换码

批量导入兑换码（自定义代码）。

**请求**

```http
POST /api/v1/admin/shop/redeem/import
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| plan_id | int | 是 | 关联的套餐 ID |
| discount_ratio | float | 是 | 折扣比例（0=免费，0.9=9折） |
| count | int | 是 | 每个兑换码的可用次数 |
| codes | string[] | 是 | 兑换码数组 |

**请求示例**

```json
{
  "plan_id": 1,
  "discount_ratio": 0,
  "count": 1,
  "codes": ["NEWYEAR2024", "SPRING2024", "VIP001"]
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "导入成功"
}
```

---

#### 31. 删除兑换码

**请求**

```http
DELETE /api/v1/admin/shop/redeem
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

### 推广管理

#### 32. 获取推广日志

**请求**

```http
GET /api/v1/admin/aff/log?page=1&size=20
Authorization: {token}
```

---

#### 33. 删除推广日志

**请求**

```http
DELETE /api/v1/admin/aff/log
Authorization: {token}
Content-Type: application/json
```

**请求参数**

```json
{
  "ids": [1, 2, 3]
}
```

---

#### 34. 推广手动记账

手动调整用户的推广佣金余额。

**请求**

```http
POST /api/v1/admin/aff/log/accounting
Authorization: {token}
Content-Type: application/json
```

**请求参数**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| uid | int | 是 | 用户 ID |
| amount | float | 是 | 金额（正数=增加，负数=扣除） |
| message | string | 否 | 订单信息/备注 |

**请求示例**

```json
{
  "uid": 1,
  "amount": 50.00,
  "message": "手动调整佣金"
}
```

**响应示例**

```json
{
  "code": 0,
  "msg": "记账成功"
}
```

**说明**

- 金额会计入用户的推广佣金余额（aff_balance）
- 负数金额代表扣除
- 若扣除后余额为负数，操作将失败

---

### 统计

#### 35. 获取统计数据

**请求**

```http
GET /api/v1/admin/statistic?top_users=10
Authorization: {token}
```

**Query 参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| top_users | int | 返回 TOP N 用户 |

**响应示例**

```json
{
  "code": 0,
  "data": {
    "total_users": 1000,
    "active_users": 500,
    "total_traffic": 10995116277760,
    "today_traffic": 107374182400,
    "top_users": [
      {
        "user_id": 1,
        "username": "user1",
        "traffic": 10737418240
      }
    ]
  }
}
```

---

## 系统模块 (System)

### 1. 获取后端信息

**请求**

```http
GET /api/v1/system/info
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "version": "1.0.0",
    "build_time": "2024-01-01",
    "go_version": "1.21"
  }
}
```

---

### 2. 获取队列信息

**请求**

```http
GET /api/v1/system/info/queue
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": {
    "pending": 10,
    "processing": 2
  }
}
```

---

### 3. 获取节点状态

获取所有设备组及其在线服务器的状态（用于 Looking Glass 等功能）。

**请求**

```http
GET /api/v1/system/node/status
Authorization: {token}
```

**响应示例**

```json
{
  "code": 0,
  "data": [
    {
      "gid": 1,
      "name": "香港入口",
      "gType": "DeviceGroupType_Inbound",
      "servers": [
        {
          "handle": "server_handle_xxx",
          "name": "HK-01"
        },
        {
          "handle": "server_handle_yyy",
          "name": "HK-02"
        }
      ]
    },
    {
      "gid": 2,
      "name": "日本出口",
      "gType": "DeviceGroupType_OutboundBySite",
      "servers": [
        {
          "handle": "server_handle_zzz",
          "name": "JP-01"
        }
      ]
    }
  ]
}
```

**字段说明**

| 字段 | 类型 | 说明 |
|------|------|------|
| gid | int | 设备组 ID |
| name | string | 设备组名称 |
| gType | string | 设备组类型 |
| servers | array | 在线服务器列表 |
| servers[].handle | string | 服务器句柄（用于 Looking Glass） |
| servers[].name | string | 服务器名称 |

---

### 4. 获取节点配置

节点客户端对接时调用。

**请求**

```http
GET /api/v1/client/config_v2?token={token}
```

**Query 参数**

| 参数 | 类型 | 说明 |
|------|------|------|
| token | string | 设备组 Token |

---

## 附录：常量定义

### 设备组类型 (DeviceGroupType)

| 值 | 说明 |
|----|------|
| DeviceGroupType_AgentOnly | 仅监控 |
| DeviceGroupType_Inbound | 入口 |
| DeviceGroupType_OutboundBySite | 出口 |
| DeviceGroupType_OutboundByUser | 单端出口 |
| DeviceGroupType_ChainOutbound | 链式出口 |

### 套餐类型 (PlanType)

| 值 | 说明 |
|----|------|
| PlanType_TrafficPack | 不限时流量包 |
| PlanType_TrafficPack_CanStacked | 不限时流量包（可叠加） |
| PlanType_Month | 月付 |
| PlanType_Day | 日付 |

### 订单类型 (OrderType)

| 值 | 说明 |
|----|------|
| OrderType_DepositToBalance | 充值到余额 |
| OrderType_PurchaseByBalance | 余额消费 |
| OrderType_Accounting | 手动记账 |

### 订单状态 (OrderStatus)

| 值 | 说明 |
|----|------|
| OrderStatus_Open | 待支付 |
| OrderStatus_Closed | 交易关闭 |
| OrderStatus_Finished | 交易完成 |

### 转发规则状态 (ForwardRuleStatus)

| 值 | 说明 |
|----|------|
| ForwardRuleStatus_Unsync | 未同步 |
| ForwardRuleStatus_Normal | 正常 |
| ForwardRuleStatus_Failed | 同步失败 |
| ForwardRuleStatus_Disabled | 已禁用 |

### 推广日志类型 (AffLogType)

| 值 | 说明 |
|----|------|
| AffiliateLogType_Commission | 产生佣金 |
| AffiliateLogType_Withdraw | 佣金提现 |
| AffiliateLogType_Deposit | 佣金转余额 |
| AffiliateLogType_Accounting | 手动记账 |

### 负载均衡策略 (SelectorType)

| 值 | 说明 |
|----|------|
| random | 随机 |
| round | 轮询 |
| ip_hash | IP Hash |
| least_load | 最小连接数 |
| failover | 故障转移 |

### 注册策略 (RegisterPolicy)

| 值 | 说明 |
|----|------|
| 0 | 无限制 |
| 1 | 不允许邀请注册 |
| 2 | 仅开放邀请注册 |

### 直接转发策略 (DirectPolicy)

| 值 | 说明 |
|----|------|
| 0 | 禁止直接转发 |
| 1 | 可选直接转发 |
| 2 | 强制直接转发 |

### 探针隐藏策略 (HideInServerStatus)

| 值 | 说明 |
|----|------|
| 0 | 不隐藏 |
| 1 | 对非管理员用户隐藏 |
| 2 | 对所有用户隐藏 |

---

## 完整操作流程示例

### 管理员登录并创建转发规则

```bash
# 1. 登录
curl -X POST https://your-domain.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password123"}'

# 响应: {"code":0,"data":"eyJhbGci..."}
# 保存 Token: TOKEN=eyJhbGci...

# 2. 获取用户信息，确认是管理员
curl https://your-domain.com/api/v1/user/info \
  -H "Authorization: $TOKEN"

# 3. 创建入口设备组
curl -X PUT https://your-domain.com/api/v1/admin/devicegroup \
  -H "Authorization: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "香港入口",
    "type": "DeviceGroupType_Inbound",
    "enable_for_gid": "1",
    "ratio": "1",
    "port_range": "10000-50000",
    "connect_host": "hk.example.com"
  }'

# 响应: {"code":0,"data":{"id":1,"token":"入口token..."}}

# 4. 创建出口设备组
curl -X PUT https://your-domain.com/api/v1/admin/devicegroup \
  -H "Authorization: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "日本出口",
    "type": "DeviceGroupType_OutboundBySite",
    "enable_for_gid": "1",
    "ratio": "1.5",
    "config": "{\"protocol\":\"ws\"}"
  }'

# 响应: {"code":0,"data":{"id":2,"token":"出口token..."}}

# 5. 在服务器上部署节点 (入口)
# bash <(curl -fLSs https://xxx/install.sh) rel_nodeclient "-t 入口token -u https://your-domain.com"

# 6. 在服务器上部署节点 (出口)
# bash <(curl -fLSs https://xxx/install.sh) rel_nodeclient "-o -t 出口token -u https://your-domain.com"

# 7. 创建转发规则
curl -X PUT https://your-domain.com/api/v1/user/forward \
  -H "Authorization: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "我的转发",
    "device_group_in": 1,
    "device_group_out": 2,
    "listen_port": 20000,
    "config": "{\"dest\":[\"target.server.com:8080\"]}"
  }'

# 响应: {"code":0,"data":{"id":1,"status":"ForwardRuleStatus_Unsync"}}

# 8. 查看规则状态
curl "https://your-domain.com/api/v1/user/forward?page=1&size=10" \
  -H "Authorization: $TOKEN"

# 等待 status 变为 ForwardRuleStatus_Normal 即可使用
```

---

## 更新日志

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0 | 2025-12-27 | 初始版本 |

---

> 文档生成时间: 2025-12-27  
> 基于前端代码分析生成

