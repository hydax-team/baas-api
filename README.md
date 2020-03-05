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
```
GET /api/v1/address/unused/count
```
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
当检查到剩余地址小于特定值，比如1000时，重新生成一批地址，并调用此接口添加充值地址，建议每次添加的充值地址不超过100个， 如果需要导入大量地址，可以多次调用此接口。 对于使用memo的公链，客户端可只调用此服务添加一个充值地址即可，由服务端给来分配memo供充值使用。
```
----
## 3 充值到账通知
HTTP Request：
```
POST /api/v1/notify/deposit
```

请求参数：

| 参数 | 类型 | 必须 | 说明 |
| ---- | ---- | ---- | ---- |
| token_id | string | 是 | 币种ID |
| from | string | 是 | 从哪个地址转出来 |
| to | string | 是 | 转给哪个地址 |
| memo | string | 可选 | memo标识 |
| amount | string | 是 | 充值金额 |
| tx_hash | string | 是 | 交易hash |
| index | string | 是 | 该充值所在交易中的位置 |
| block_height | string | 是 | 区块高度 |
| block_time | string | 是 | 区块时间（秒） |

响应结果
| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| code | int | 详情见返回类型表 |
| msg | string | 返回内容，失效时为错误信息 |
```
当有用户充值时，调用此接口，为保证充值可靠性，充值需要逐笔执行，超过1条的话可以多次调用此接口。 客户端必须保证充币的真实可靠，因客户端通知错误充值带来的损失，由客户端自行承担。
```
----
## 4 获取待处理提现请求
HTTP Request：
```
GET /api/v1/withdrawal/orders?chain=ABC
```

请求参数：

| 参数 | 类型 | 必须 | 说明 |
| ---- | ---- | ---- | ---- |
| chain | string | 是 | 链，使用主网代币 |

响应结果
| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| code | int | 详情见返回类型表 |
| msg | string | 返回内容，失效时为错误信息 |
| data | order[] | 待处理提现订单列表 |

---
order 信息

| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| order_id | string | 订单id |
| token_id | string | 提现币种 |
| to | string | 提现给哪个地址 |
| memo | string | memo标记 |
| amount | string | 提现金额 |
```
应定期轮询是否有用户提现需求 ，轮询周期建议不大于出块间隔的十分之一。 如出块时间为15s，建议每1s轮询一次，出块时间为15min的话，建议15s轮询一次。 为对账方便，服务端返回提币金额为净提币金额，链处理所需的手续费由客户端管理。 该接口每次最多返回50个未处理订单。
```
----

## 5 提现处理完成通知
HTTP Request：
```
POST /api/v1/notify/withdrawal
```

请求参数：

| 参数 | 类型 | 必须 | 说明 |
| ---- | ---- | ---- | ---- |
| order_id | string | 是 | 订单id |
| token_id | string | 是 | 币种ID |
| to | string | 是 | 提现给哪个地址 |
| memo | string | 可选 | memo标识 |
| amount | string | 是 | 充值金额 |
| tx_hash | string | 是 | 交易hash |
| index | string | 是 | 该充值所在交易中的位置 |
| block_height | string | 是 | 区块高度 |
| block_time | string | 是 | 区块时间（秒） |

响应结果
| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| code | int | 详情见返回类型表 |
| msg | string | 返回内容，失效时为错误信息 |
```
提币成功（提币交易已被区块链打包并确认执行成功）后调用此接口，为保证提币结果反馈可靠性，每次仅通知一笔提现处理结果。 客服端负责保证提币执行后才调用此接口
```
----

## 6 对账
HTTP Request：
```
POST /api/v1/asset/verify
```

请求参数：

| 参数 | 类型 | 必须 | 说明 |
| ---- | ---- | ---- | ---- |
| token_id | string | 是 | 提现币种 |
| total_deposit_amount | string | 是 | 总充值金额 |
| total_withdrawal_amount | string | 是 | 总提现金额 |
| last_block_height | string | 是 | 对账最高区块高度 |

响应结果
| 参数 | 类型 | 说明 |
| ---- | ---- | ---- |
| code | int | 详情见返回类型表 |
| msg | string | 返回内容，失效时为错误信息 |
```
定期进行资产对账，客户端定期（每小时或者每天进行对账，客户端根据链的出块时间，交易数量合理确定）向服务端反馈特定链上资产的充提情况（截止到asset_info中指定的区块高度）。 如果有未处理完成的提现订单，建议处理完成后进行对账。当服务端发现客户端反馈的资产信息与服务端不一致，将返回错误，并暂停该币种的充值提现。
```
----

## 7 返回值列表
| 返回值 | 类型 | 说明 |
| ---- | ---- | ---- |
| 10000 | SUCCESS | 成功 |
| 10001 | INVALID_SIGN | 无效签名 |
| 10002 | INVALID_APIKEY | 无效的api_key |
| 10004 | INVALID_CHAIN | 无效的chain |
| 10005 | INVALID_TOKEN_ID | 无效的token_id |
| 10006 | INVALID_PARAMS | 无效的参数 |
| 10007 | INVALID_TO_ADDRESS | 无效的充币地址 |
| 10008 | INVALID_ORDER_ID | 无效的订单id |
| 10009 | INVALID_AMOUNT | 无效的amount值 |
| 10010 | INVALID_DECIMALS | 无效的精度 |
| 10011 | INVALID_BLOCK_HEIGHT | 无效的区块高度 |
| 10012 | INVALID_BLOCK_TIME | 无效的区块时间 |
| 10013 | INVALID_TXHASH | 无效的tx_hash |
| 10014 | INVALID_INDEX | 无效的交易index |
| 10015 | NETWORK_ERROR | 网络错误 |
| 10016 | REPEAT_DEPOSIT | 重复充值 |
| 10017 | ASSET_VERIFY_FAILED | 资产校验失败 |
| 10018 | DEPOSIT_SUSPENDED | 充值暂停 |
| 10019 | WITHDRAWAL_SUSPENDED | 提现暂停 |
| 10020 | TIMESTAMP_EXPIRED | 时间戳过期 |
