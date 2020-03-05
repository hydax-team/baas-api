# Hydax BAAS

Hydax BAAS 提供REST风格的API（HTTPS + JSON)，方便第三方公链接入hydax。

在请求API接口之前，需要申请APIKEY, 使用ED25519算法生成公私钥对，用户自己保存私钥，公钥在上币申请时进行提交，得到APIKEY。

# API签名验证
签名前准备的数据如下： HTTP_METHOD + | + HTTP_REQUEST_PATH + | + TIMESTAMP + | + PARAMS 连接完成后，对数据进行 ED25519 签名，签名后的 bytes 进行 Hex 编码。

## HTTP方法
GET　POST

## TIMESTAMP
访问 API 时的 UNIX EPOCH 时间戳 (精确到毫秒), 过期时间120000ms。

# 接口列表
## 1 获取剩余地址数量
HTTP Request：

GET /api/v1/address/unused/count

请求参数：

| 参数 | 类型 | 必须 | 说明 |
| ---- | ---- | ---- | ---- |
| chain | string | 是 | 链，使用主网代币 |

响应结果
| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| code | int | 详情见返回类型表 |
| msg | string | 返回内容，失效时为错误信息 |
| data | int | 剩余地址数量 |
```
检查是否需要重新生成地址时使用，需要定期调用，视新增用户速度决定。建议每小时调用一次。
```
----
## 2 添加充值地址
HTTP Request：
```
POST /api/v1/address/add
```

请求参数：

| 参数 | 类型 | 必须 | 说明 |
| ---- | ---- | ---- | ---- |
| chain | string | 是 | 链，使用主网代币 |
| addr_list | string[] | 是 | 地址列表 |

响应结果
| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| code | int | 详情见返回类型表 |
| msg | string | 返回内容，失效时为错误信息 |
```
当检查到剩余地址小于特定值，比如1000时，重新生成一批地址，并调用此接口添加充值地址，建议每次添加的充值地址不超过100个， 如果需要导入大量地址，可以多次调用此接口。 对于使用memo的公链，客户端可只调用此服务添加一个充值地址即可，由服务端给来分配memo供充值使用。```
----

