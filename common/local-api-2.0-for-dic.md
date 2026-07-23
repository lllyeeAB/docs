# Local API V2 使用指南

Local API V2 的调用方式与 V1 一样：先确认本机端口和 Local API Key，再按接口地址发起 HTTP 请求。

V2 接口路径统一以 `/openapi/v2` 开头，例如 `/openapi/v2/profiles`、`/openapi/v2/proxies`、`/openapi/v2/fingerprints`。

## 基础地址

Local API 服务运行在本机端口上，完整请求地址格式为：

```text
http://127.0.0.1:{port}/openapi/v2/...
```

示例：

```http
GET http://127.0.0.1:52100/openapi/v2/profiles
```

`{port}` 为客户端显示或配置的 Local API 端口。示例中的端口、环境 ID、代理 ID 和 API Key 都需要替换为你的实际值。

## 请求头

| 名称           | 类型   | 必填             | 说明                                 |
| -------------- | ------ | ---------------- | ------------------------------------ |
| `X-API-KEY`    | string | 是               | Local API Key。                      |
| `Content-Type` | string | 有请求体时建议传 | 固定为 `application/json`。          |
| `X-Token`      | string | 否               | 可选认证头。普通调用通常不需要填写。 |

说明：

- 请使用 `X-API-KEY` 调用接口。
- 有请求体的接口请传合法 JSON。
- 参数不符合要求时，会返回对应的错误信息。

## 响应格式

V2 使用统一响应包络：

```json
{
  "code": 0,
  "msg": "success",
  "data": {},
  "next": null
}
```

| 名称   | 类型        | 说明                                        |
| ------ | ----------- | ------------------------------------------- |
| `code` | integer     | 业务状态码。`0` 表示成功。                  |
| `msg`  | string/null | 响应消息。                                  |
| `data` | object/null | 响应数据。不同接口结构不同。                |
| `next` | string/null | 游标分页的下一页标记。无下一页时为 `null`。 |

常见错误：

| HTTP |     code | 说明                           |
| ---: | -------: | ------------------------------ |
|  401 | `401000` | 缺少 `X-API-KEY`。             |
|  403 | `401000` | `X-API-KEY` 与本地配置不匹配。 |
|  429 | `429000` | 请求过于频繁。                 |
|  502 | `502000` | 服务暂不可用。                 |
|  502 | `502001` | 服务响应异常。                 |
|  504 | `504000` | 请求超时。                     |

## 分页规则

列表接口支持两种分页方式：

| 方式     | 参数                | 说明                                                 |
| -------- | ------------------- | ---------------------------------------------------- |
| 页码分页 | `page`、`page_size` | `page` 从 `1` 开始。                                 |
| 游标分页 | `cursor`、`limit`   | 第一次请求不传 `cursor`；下一页使用响应中的 `next`。 |

`page/page_size` 与 `cursor/limit` 互斥，不要混用。

分页列表响应示例：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "list": [],
    "total": 0,
    "summary": null
  },
  "next": null
}
```

## 接口范围

当前支持 37 个 V2 接口：

| 模块              | 数量 | 说明                                                  |
| ----------------- | ---: | ----------------------------------------------------- |
| 环境 Profiles     |   19 | 环境增删改查、账号、分组移动、代理绑定、打开/关闭等。 |
| 代理 Proxies      |    7 | 代理增删改查和代理检测。                              |
| Cookie            |    5 | Cookie 查询、导入、清空、导出。                       |
| 指纹 Fingerprints |    6 | 指纹查询、覆盖、刷新、生成和选项查询。                |

# 环境 Profiles

## 查询环境列表

### 接口地址

```http
GET /openapi/v2/profiles
```

用于查询环境列表。

### 查询参数

| 名称               | 类型     | 必填 | 说明                                                      |
| ------------------ | -------- | ---- | --------------------------------------------------------- |
| `page`             | integer  | 否   | 页码分页页码，从 `1` 开始。不能与 `cursor/limit` 混用。   |
| `page_size`        | integer  | 否   | 页码分页每页数量。不能与 `cursor/limit` 混用。            |
| `cursor`           | string   | 否   | 游标分页标记。第一次请求不传。                            |
| `limit`            | integer  | 否   | 游标分页每页数量。不能与 `page/page_size` 混用。          |
| `serial_no`        | integer  | 否   | 环境序号，精准匹配。                                      |
| `name`             | string   | 否   | 环境名称，模糊匹配。                                      |
| `remark`           | string   | 否   | 备注，模糊匹配。                                          |
| `group_ids`        | string[] | 否   | 分组 ID 列表。                                            |
| `tag_ids`          | string[] | 否   | 标签 ID 列表。                                            |
| `proxy_type`       | string   | 否   | 代理类型，例如 `none`、`http`、`https`、`ssh`、`socks5`。 |
| `run_status`       | string   | 否   | 运行状态。可选值：`running`、`stopped`。                  |
| `created_from`     | string   | 否   | 创建开始时间。                                            |
| `created_to`       | string   | 否   | 创建结束时间。                                            |
| `last_opened_from` | string   | 否   | 最近打开开始时间。                                        |
| `last_opened_to`   | string   | 否   | 最近打开结束时间。                                        |

数组查询参数可以重复传入：

```text
group_ids=group-a&group_ids=group-b
```

### 请求示例

```bash
curl -X GET "http://127.0.0.1:52100/openapi/v2/profiles?page=1&page_size=20&run_status=stopped" \
  -H "X-API-KEY: your-local-api-key"
```

### 响应参数

`data.list` 为环境摘要列表，列表项结构见 [环境摘要字段](#环境摘要字段)。

## 创建环境

### 接口地址

```http
POST /openapi/v2/profiles
```

用于创建一个新的浏览器环境。

### 请求参数

请求体使用 `ProfileCreateRequest`，字段见 [环境创建和更新参数](#环境创建和更新参数)。

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/profiles" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "profile-v2-demo",
    "group_ids": [],
    "proxy_binding": {
      "mode": "none"
    },
    "fingerprint": {
      "os": "windows",
      "kernel_version": "142",
      "language": {
        "mode": "ip"
      },
      "timezone": {
        "mode": "ip"
      }
    },
    "advanced": {
      "startup": {
        "urls": [],
        "fixed_urls": [],
        "restore_session_mode": "global"
      },
      "multi_open": "global",
      "remote_inspector": "global",
      "spoofing_video": "default"
    },
    "cookies": [],
    "account_list": [],
    "tag_ids": [],
    "remark": "created by Local API V2"
  }'
```

### 响应示例

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "id": "1876881021063852034",
    "serial_no": 1,
    "name": "profile-v2-demo"
  },
  "next": null
}
```

响应中的 `data` 使用环境摘要结构，字段见 [环境摘要字段](#环境摘要字段)。

## 获取环境详情

### 接口地址

```http
GET /openapi/v2/profiles/{profileId}
```

用于获取单个环境详情。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求示例

```bash
curl -X GET "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034" \
  -H "X-API-KEY: your-local-api-key"
```

响应中的 `data` 使用环境详情结构，字段见 [环境详情字段](#环境详情字段)。

## 部分更新环境

### 接口地址

```http
PATCH /openapi/v2/profiles/{profileId}
```

用于更新已有环境。未传入的字段保持不变。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

请求体使用 `ProfilePatchRequest`，字段与 `ProfileCreateRequest` 基本一致，见 [环境创建和更新参数](#环境创建和更新参数)。

### 请求示例

```bash
curl -X PATCH "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "profile-v2-demo-updated",
    "remark": "updated by Local API V2"
  }'
```

## 删除环境

### 接口地址

```http
DELETE /openapi/v2/profiles/{profileId}
```

用于删除环境。通常表示移入回收站。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求示例

```bash
curl -X DELETE "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034" \
  -H "X-API-KEY: your-local-api-key"
```

## 彻底删除环境

### 接口地址

```http
DELETE /openapi/v2/profiles/{profileId}/permanent
```

用于彻底删除环境。该操作不可恢复，请谨慎使用。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求示例

```bash
curl -X DELETE "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/permanent" \
  -H "X-API-KEY: your-local-api-key"
```

## 恢复环境

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/restore
```

用于恢复已删除的环境。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/restore" \
  -H "X-API-KEY: your-local-api-key"
```

## 移动环境分组

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/move
```

用于移动环境所属分组。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称        | 类型     | 必填 | 说明               |
| ----------- | -------- | ---- | ------------------ |
| `group_ids` | string[] | 否   | 目标分组 ID 列表。 |

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/move" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "group_ids": ["1876881021063852033"]
  }'
```

## 克隆环境

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/clone
```

用于克隆环境。

### 路径参数

| 名称        | 类型   | 必填 | 说明              |
| ----------- | ------ | ---- | ----------------- |
| `profileId` | string | 是   | 被克隆的环境 ID。 |

### 请求参数

| 名称            | 类型     | 必填 | 说明                                                       |
| --------------- | -------- | ---- | ---------------------------------------------------------- |
| `copies`        | integer  | 否   | 克隆份数，表示要复制出几个新环境。                         |
| `group_ids`     | string[] | 否   | 目标分组 ID 列表。留空则沿用原环境分组。                   |
| `inherit_items` | string[] | 否   | 需要继承的数据项。留空默认继承指纹、代理、账号和云端数据。 |
| `remark`        | string   | 否   | 克隆备注。留空则沿用原环境备注。                           |

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/clone" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "copies": 1,
    "group_ids": [],
    "inherit_items": ["fingerprint", "proxy", "account"],
    "remark": "clone by Local API V2"
  }'
```

## 打开环境

### 接口地址

```http
PATCH /openapi/v2/profiles/{profileId}/open
```

用于打开环境。成功后可从响应 `data` 中获取调试端口或 WebSocket 地址。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称                           | 类型    | 必填 | 说明                                     |
| ------------------------------ | ------- | ---- | ---------------------------------------- |
| `client_ip`                    | string  | 否   | 客户端 IP。                              |
| `headless`                     | boolean | 否   | 是否无头打开。桌面客户端通常传 `false`。 |
| `skip_proxy_check`             | boolean | 否   | 是否跳过打开前代理检测。                 |
| `skip_cookie_sync_before_open` | boolean | 否   | 是否跳过打开前 Cookie 同步。             |
| `skip_data_sync_before_open`   | boolean | 否   | 是否跳过打开前数据同步。                 |
| `skip_extension_data_sync`     | boolean | 否   | 是否跳过打开前扩展数据同步。             |

可选查询参数：

| 名称   | 类型   | 必填 | 说明                                                      |
| ------ | ------ | ---- | --------------------------------------------------------- |
| `sync` | string | 否   | 传 `false` 时可返回打开进度，最终结果仍使用 V2 响应格式。 |

### 请求示例

```bash
curl -X PATCH "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/open" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "client_ip": "127.0.0.1",
    "headless": false,
    "skip_proxy_check": true
  }'
```

### 响应示例

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "id": "1876881021063852034",
    "pid": "12345",
    "serial_number": 1,
    "debug_port": 9222,
    "web_socket_url": "ws://127.0.0.1:9222/devtools/browser/xxxx",
    "request_id": "request-id"
  },
  "next": null
}
```

## 关闭环境

### 接口地址

```http
PATCH /openapi/v2/profiles/{profileId}/close
```

用于关闭环境。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称                              | 类型    | 必填 | 说明                         |
| --------------------------------- | ------- | ---- | ---------------------------- |
| `skip_cookie_sync_after_close`    | boolean | 否   | 是否跳过关闭后 Cookie 同步。 |
| `skip_data_sync_after_close`      | boolean | 否   | 是否跳过关闭后数据同步。     |
| `skip_extension_sync_after_close` | boolean | 否   | 是否跳过关闭后扩展数据同步。 |

### 请求示例

```bash
curl -X PATCH "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/close" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "skip_cookie_sync_after_close": true,
    "skip_data_sync_after_close": true,
    "skip_extension_sync_after_close": true
  }'
```

## 查询环境账号

### 接口地址

```http
GET /openapi/v2/profiles/{profileId}/accounts
```

用于查询环境账号列表。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

响应中的 `data.list` 为账号列表，字段见 [账号参数](#账号参数)。

## 添加环境账号

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/accounts
```

用于给环境添加账号。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

请求体使用账号参数，见 [账号参数](#账号参数)。

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/accounts" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "platform": "other",
    "username": "account@example.com",
    "password": "password",
    "secret": "",
    "url": "https://example.com",
    "remark": "account remark"
  }'
```

## 修改环境账号

### 接口地址

```http
PATCH /openapi/v2/profiles/{profileId}/accounts/{accountId}
```

用于修改环境账号。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |
| `accountId` | string | 是   | 账号 ID。 |

### 请求参数

请求体使用账号参数，见 [账号参数](#账号参数)。

## 删除环境账号

### 接口地址

```http
DELETE /openapi/v2/profiles/{profileId}/accounts/{accountId}
```

用于删除环境账号。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |
| `accountId` | string | 是   | 账号 ID。 |

## 更新环境代理绑定

### 接口地址

```http
PATCH /openapi/v2/profiles/{profileId}/proxy-binding
```

用于更新单个环境的代理绑定。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称            | 类型   | 必填 | 说明                                             |
| --------------- | ------ | ---- | ------------------------------------------------ |
| `proxy_binding` | object | 否   | 代理绑定配置，见 [代理绑定参数](#代理绑定参数)。 |

### 请求示例

```bash
curl -X PATCH "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/proxy-binding" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "proxy_binding": {
      "mode": "linked",
      "proxy_id": "1876881021063852999"
    }
  }'
```

## 批量更新环境代理绑定

### 接口地址

```http
PATCH /openapi/v2/profiles/proxy-binding/batch
```

用于批量更新环境代理绑定。

### 请求参数

| 名称            | 类型     | 必填 | 说明                                             |
| --------------- | -------- | ---- | ------------------------------------------------ |
| `profile_ids`   | string[] | 否   | 环境 ID 列表，不能为空。                         |
| `proxy_binding` | object   | 否   | 代理绑定配置，见 [代理绑定参数](#代理绑定参数)。 |

## 获取局部已打开环境汇总

### 接口地址

```http
POST /openapi/v2/profiles/summary
```

用于获取指定环境集合中的已打开环境汇总。

### 请求参数

| 名称  | 类型     | 必填 | 说明           |
| ----- | -------- | ---- | -------------- |
| `ids` | string[] | 是   | 环境 ID 列表。 |

## 清除环境 Storage

### 接口地址

```http
DELETE /openapi/v2/profiles/{profileId}/storage
```

用于清除环境 Storage。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称    | 类型     | 必填 | 说明                                                                      |
| ------- | -------- | ---- | ------------------------------------------------------------------------- |
| `types` | string[] | 否   | 需要清理的 Storage 类型。不传时默认清理 `local_storage` 和 `indexed_db`。 |

### 请求示例

```bash
curl -X DELETE "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/storage" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "types": ["local_storage", "indexed_db"]
  }'
```

# 代理 Proxies

## 查询代理列表

### 接口地址

```http
GET /openapi/v2/proxies
```

用于查询代理列表。

### 查询参数

| 名称             | 类型    | 必填 | 说明                        |
| ---------------- | ------- | ---- | --------------------------- |
| `page`           | integer | 否   | 页码分页页码，从 `1` 开始。 |
| `page_size`      | integer | 否   | 页码分页每页数量。          |
| `cursor`         | string  | 否   | 游标分页标记。              |
| `limit`          | integer | 否   | 游标分页每页数量。          |
| `serial_no`      | integer | 否   | 代理序号。                  |
| `id`             | string  | 否   | 代理 ID。                   |
| `type`           | string  | 否   | 代理类型。                  |
| `host`           | string  | 否   | 代理主机。                  |
| `remark`         | string  | 否   | 备注。                      |
| `proxy_group_id` | string  | 否   | 代理分组 ID。               |

响应中的 `data.list` 为代理列表项，字段见 [代理响应字段](#代理响应字段)。

## 创建代理

### 接口地址

```http
POST /openapi/v2/proxies
```

用于创建代理。

### 请求参数

请求体使用代理新增或修改参数，见 [代理参数](#代理参数)。

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/proxies" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "http",
    "host": "127.0.0.1",
    "port": 8080,
    "user": "",
    "password": "",
    "ipchecker": "ipapi",
    "ip_version": "ipv4",
    "remark": "proxy created by Local API V2"
  }'
```

## 获取代理详情

### 接口地址

```http
GET /openapi/v2/proxies/{proxyId}
```

### 路径参数

| 名称      | 类型   | 必填 | 说明      |
| --------- | ------ | ---- | --------- |
| `proxyId` | string | 是   | 代理 ID。 |

## 修改代理

### 接口地址

```http
PATCH /openapi/v2/proxies/{proxyId}
```

### 路径参数

| 名称      | 类型   | 必填 | 说明      |
| --------- | ------ | ---- | --------- |
| `proxyId` | string | 是   | 代理 ID。 |

### 请求参数

请求体使用代理新增或修改参数，见 [代理参数](#代理参数)。

## 删除代理

### 接口地址

```http
DELETE /openapi/v2/proxies/{proxyId}
```

### 路径参数

| 名称      | 类型   | 必填 | 说明      |
| --------- | ------ | ---- | --------- |
| `proxyId` | string | 是   | 代理 ID。 |

## 检测已保存代理

### 接口地址

```http
POST /openapi/v2/proxies/{proxyId}/check
```

用于检测已保存代理。

### 路径参数

| 名称      | 类型   | 必填 | 说明      |
| --------- | ------ | ---- | --------- |
| `proxyId` | string | 是   | 代理 ID。 |

## 检测临时代理

### 接口地址

```http
POST /openapi/v2/proxies/check
```

用于检测未保存的临时代理。

### 请求参数

请求体使用代理检测参数，见 [代理检测参数](#代理检测参数)。

### 响应示例

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "ok": true,
    "out_ip": "1.2.3.4",
    "country": "United States",
    "country_code": "US",
    "region": "California",
    "city": "Los Angeles",
    "timezone": "America/Los_Angeles",
    "status": "success",
    "error_message": ""
  },
  "next": null
}
```

# Cookie

## 查询环境 Cookie

### 接口地址

```http
GET /openapi/v2/profiles/{profileId}/cookies
```

用于查询环境 Cookie。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 查询参数

| 名称     | 类型   | 必填 | 说明                                       |
| -------- | ------ | ---- | ------------------------------------------ |
| `domain` | string | 否   | 按 Cookie 域名过滤。不传表示返回全部域名。 |

## 覆盖导入环境 Cookie

### 接口地址

```http
PUT /openapi/v2/profiles/{profileId}/cookies
```

覆盖导入 Cookie，会用请求体内容替换当前环境 Cookie。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

请求体使用 Cookie 导入参数，见 [Cookie 参数](#cookie-参数)。

## 增量导入环境 Cookie

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/cookies
```

增量导入 Cookie，会在当前环境 Cookie 基础上合并写入请求体内容。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

请求体使用 Cookie 导入参数，见 [Cookie 参数](#cookie-参数)。

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/profiles/1876881021063852034/cookies" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "merge",
    "cookies": [
      {
        "domain": ".example.com",
        "path": "/",
        "name": "session",
        "value": "cookie-value",
        "expirationDate": 1893456000,
        "httpOnly": true,
        "secure": true,
        "sameSite": "lax"
      }
    ]
  }'
```

## 清空环境 Cookie

### 接口地址

```http
DELETE /openapi/v2/profiles/{profileId}/cookies
```

用于清空环境 Cookie。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

## 导出环境 Cookie

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/cookies/export
```

用于导出环境 Cookie。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称                | 类型     | 必填 | 说明                                             |
| ------------------- | -------- | ---- | ------------------------------------------------ |
| `format`            | string   | 否   | 导出格式，例如 `json`。                          |
| `domains`           | string[] | 否   | 需要导出的域名列表。不传或传空表示导出全部域名。 |
| `include_http_only` | boolean  | 否   | 是否包含 HttpOnly Cookie。                       |

# 指纹 Fingerprints

## 获取环境指纹

### 接口地址

```http
GET /openapi/v2/profiles/{profileId}/fingerprint
```

用于获取单个环境的指纹配置。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

## 覆盖设置环境指纹

### 接口地址

```http
PUT /openapi/v2/profiles/{profileId}/fingerprint
```

用于覆盖设置单个环境指纹。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称          | 类型   | 必填 | 说明                                 |
| ------------- | ------ | ---- | ------------------------------------ |
| `fingerprint` | object | 否   | 指纹配置，见 [指纹参数](#指纹参数)。 |

## 刷新单个环境指纹

### 接口地址

```http
POST /openapi/v2/profiles/{profileId}/fingerprint/refresh
```

用于刷新单个环境指纹。

### 路径参数

| 名称        | 类型   | 必填 | 说明      |
| ----------- | ------ | ---- | --------- |
| `profileId` | string | 是   | 环境 ID。 |

### 请求参数

| 名称          | 类型   | 必填 | 说明                                 |
| ------------- | ------ | ---- | ------------------------------------ |
| `fingerprint` | object | 否   | 指纹配置，见 [指纹参数](#指纹参数)。 |

## 生成指纹

### 接口地址

```http
POST /openapi/v2/fingerprints/generate
```

用于生成一份临时指纹。

### 请求参数

| 名称          | 类型   | 必填 | 说明                                 |
| ------------- | ------ | ---- | ------------------------------------ |
| `fingerprint` | object | 否   | 指纹配置，见 [指纹参数](#指纹参数)。 |

### 请求示例

```bash
curl -X POST "http://127.0.0.1:52100/openapi/v2/fingerprints/generate" \
  -H "X-API-KEY: your-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "fingerprint": {
      "os": "windows",
      "kernel_version": "142",
      "language": {
        "mode": "ip"
      },
      "timezone": {
        "mode": "ip"
      },
      "webrtc": {
        "mode": "replace",
        "ip_source": "proxy",
        "keep_random_internal_ip": false
      }
    }
  }'
```

## 批量刷新环境指纹

### 接口地址

```http
PATCH /openapi/v2/profiles/fingerprints
```

用于批量刷新环境指纹。

### 请求参数

| 名称          | 类型     | 必填 | 说明                                 |
| ------------- | -------- | ---- | ------------------------------------ |
| `profile_ids` | string[] | 否   | 环境 ID 列表。                       |
| `fingerprint` | object   | 否   | 指纹配置，见 [指纹参数](#指纹参数)。 |

## 获取指纹可选项

### 接口地址

```http
GET /openapi/v2/fingerprints/options
```

用于获取系统支持的指纹枚举选项，例如系统类型、内核版本、代理类型、WebRTC 模式等。

# 公共参数说明

## 环境创建和更新参数

`ProfileCreateRequest` 和 `ProfilePatchRequest` 使用相同的主要字段：

| 名称            | 类型     | 必填 | 说明                                                                            |
| --------------- | -------- | ---- | ------------------------------------------------------------------------------- |
| `name`          | string   | 否   | 环境名称。不传时按系统规则自动生成。                                            |
| `group_ids`     | string[] | 否   | 分组 ID 列表。仅保留一个有效值；不传或传空时自动使用默认分组。                  |
| `proxy_binding` | object   | 否   | 环境代理绑定配置，见 [代理绑定参数](#代理绑定参数)。                            |
| `fingerprint`   | object   | 否   | 指纹配置，见 [指纹参数](#指纹参数)。                                            |
| `advanced`      | object   | 否   | 高级配置，见 [高级配置参数](#高级配置参数)。                                    |
| `cookies`       | object[] | 否   | 创建或更新环境时写入的 Cookie 列表。不传表示不写入，传空数组表示写入空 Cookie。 |
| `account_list`  | object[] | 否   | 创建或更新环境时绑定的平台账号列表，见 [账号参数](#账号参数)。                  |
| `tag_ids`       | string[] | 否   | 标签 ID 列表。                                                                  |
| `remark`        | string   | 否   | 备注。                                                                          |

## 代理绑定参数

`proxy_binding` 字段用于配置环境使用的代理。

| 名称                | 类型    | 必填 | 说明                                                                                                |
| ------------------- | ------- | ---- | --------------------------------------------------------------------------------------------------- |
| `mode`              | string  | 否   | 代理绑定模式。可选值：`none`、`manual`、`linked`。                                                  |
| `proxy_id`          | string  | 否   | 已保存代理 ID。`mode=linked` 时必填。                                                               |
| `type`              | string  | 否   | 代理类型。`mode=manual` 时必填，且不能为 `none`。可选值：`none`、`http`、`https`、`ssh`、`socks5`。 |
| `host`              | string  | 否   | 代理主机。`mode=manual` 时必填。                                                                    |
| `port`              | integer | 否   | 代理端口。`mode=manual` 时必填，取值范围 `1-65535`。                                                |
| `username`          | string  | 否   | 代理账号。`mode=manual` 时使用。                                                                    |
| `password`          | string  | 否   | 代理密码。`mode=manual` 时使用。                                                                    |
| `ip_check_provider` | string  | 否   | IP 检测渠道。可选值：`ip2location`、`ipapi`。                                                       |
| `ip_version`        | string  | 否   | IP 地址类型。可选值：`ipv4`、`ipv6`。                                                               |

示例：不使用代理。

```json
{
  "proxy_binding": {
    "mode": "none"
  }
}
```

示例：绑定已有代理。

```json
{
  "proxy_binding": {
    "mode": "linked",
    "proxy_id": "1876881021063852999"
  }
}
```

示例：手动代理。

```json
{
  "proxy_binding": {
    "mode": "manual",
    "type": "http",
    "host": "127.0.0.1",
    "port": 8080,
    "username": "",
    "password": "",
    "ip_check_provider": "ipapi",
    "ip_version": "ipv4"
  }
}
```

## 指纹参数

`fingerprint` 用于配置环境指纹。所有字段均为可选；不传时按系统规则生成或沿用已有配置。

| 名称             | 类型   | 说明                                                                        |
| ---------------- | ------ | --------------------------------------------------------------------------- |
| `os`             | string | 系统类型。可选值：`random`、`windows`、`macos`、`linux`、`android`、`ios`。 |
| `kernel_version` | string | 浏览器内核版本。可选值：`120`、`134`、`142`、`143`、`147`。                 |
| `ua`             | string | User Agent。留空时系统随机生成。                                            |
| `language`       | object | 浏览器语言配置。                                                            |
| `ui_language`    | object | 界面语言配置。                                                              |
| `timezone`       | object | 时区配置。                                                                  |
| `geolocation`    | object | 地理位置配置。                                                              |
| `fonts`          | object | 字体配置。                                                                  |
| `webrtc`         | object | WebRTC 配置。                                                               |
| `screen`         | object | 屏幕配置。                                                                  |
| `webgl`          | object | WebGL 配置。                                                                |
| `webgpu`         | object | 通用指纹模块配置。                                                          |
| `audio_context`  | object | 通用指纹模块配置。                                                          |
| `client_rects`   | object | 通用指纹模块配置。                                                          |
| `speech_voices`  | object | 通用指纹模块配置。                                                          |
| `media_devices`  | object | 通用指纹模块配置。                                                          |
| `hardware`       | object | 硬件配置。                                                                  |
| `privacy`        | object | 隐私配置。                                                                  |
| `launch`         | object | 启动参数配置。                                                              |

### `language`

| 名称        | 类型     | 说明                                                           |
| ----------- | -------- | -------------------------------------------------------------- |
| `mode`      | string   | 浏览器语言模式。可选值：`ip`、`custom`。                       |
| `languages` | string[] | 浏览器语言列表。`mode=custom` 时必填，例如 `["en-US", "en"]`。 |

### `ui_language`

| 名称    | 类型   | 说明                                                                                   |
| ------- | ------ | -------------------------------------------------------------------------------------- |
| `mode`  | string | 界面语言模式。可选值：`follow_browser_language`、`current_device_language`、`custom`。 |
| `value` | string | 自定义界面语言。`mode=custom` 时必填，例如 `en-US`。                                   |

### `timezone`

| 名称    | 类型   | 说明                                       |
| ------- | ------ | ------------------------------------------ |
| `mode`  | string | 时区来源。可选值：`ip`、`custom`、`real`。 |
| `value` | string | 时区值。仅 `mode=custom` 时生效。          |

### `geolocation`

| 名称                    | 类型   | 说明                                            |
| ----------------------- | ------ | ----------------------------------------------- |
| `permission`            | string | 地理位置权限。可选值：`ask`、`allow`、`block`。 |
| `source`                | string | 坐标来源。可选值：`ip`、`custom`。              |
| `coordinates.longitude` | number | 经度。                                          |
| `coordinates.latitude`  | number | 纬度。                                          |
| `coordinates.accuracy`  | number | 精度。                                          |

### `fonts`

| 名称     | 类型     | 说明                                                     |
| -------- | -------- | -------------------------------------------------------- |
| `mode`   | string   | 字体模式。可选值：`real`、`mask`、`custom`、`disabled`。 |
| `values` | string[] | 字体列表。仅 `mode=custom` 时生效。                      |

### `webrtc`

| 名称                      | 类型    | 说明                                                                          |
| ------------------------- | ------- | ----------------------------------------------------------------------------- |
| `mode`                    | string  | WebRTC 模式。可选值：`real`、`replace`、`forward`、`disabled`。               |
| `ip_source`               | string  | 替换模式来源。仅 `mode=replace` 时生效。可选值：`manual`、`proxy`、`random`。 |
| `ip`                      | string  | 手动指定内网 IP。仅 `ip_source=manual` 时生效。                               |
| `keep_random_internal_ip` | boolean | 随机内网 IP 是否保持不变。仅 `ip_source=random` 时生效。                      |

### `screen`

| 名称                 | 类型    | 说明                                                            |
| -------------------- | ------- | --------------------------------------------------------------- |
| `resolution_mode`    | string  | 分辨率模式。可选值：`recommended`、`random`、`custom`、`real`。 |
| `resolution.width`   | integer | 分辨率宽度。                                                    |
| `resolution.height`  | integer | 分辨率高度。                                                    |
| `window_size_mode`   | string  | 窗口大小模式。可选值：`recommended`、`custom`。                 |
| `window_size.width`  | integer | 窗口宽度。                                                      |
| `window_size.height` | integer | 窗口高度。                                                      |

### `webgl`

| 名称            | 类型   | 说明                                                             |
| --------------- | ------ | ---------------------------------------------------------------- |
| `image_mode`    | string | WebGL 图像模式。可选值：`real`、`mask`、`custom`、`disabled`。   |
| `metadata_mode` | string | WebGL 元数据模式。可选值：`real`、`mask`、`custom`、`disabled`。 |
| `manufacturer`  | string | WebGL 厂商。仅 `metadata_mode=custom` 时生效。                   |
| `renderer`      | string | WebGL 渲染器。仅 `metadata_mode=custom` 时生效。                 |

### 通用指纹模块

`webgpu`、`audio_context`、`client_rects`、`speech_voices`、`media_devices` 使用同一结构：

| 名称   | 类型   | 说明                                                     |
| ------ | ------ | -------------------------------------------------------- |
| `mode` | string | 模块模式。可选值：`real`、`mask`、`custom`、`disabled`。 |

### `hardware`

| 名称               | 类型   | 说明                                                         |
| ------------------ | ------ | ------------------------------------------------------------ |
| `cpu_cores.mode`   | string | CPU 核心数取值模式。可选值：`random`、`custom`、`real`。     |
| `cpu_cores.value`  | string | 自定义 CPU 核心数。仅 `mode=custom` 时生效。                 |
| `memory_gb.mode`   | string | 内存取值模式。可选值：`random`、`custom`、`real`。           |
| `memory_gb.value`  | string | 自定义内存 GB。仅 `mode=custom` 时生效。                     |
| `device_name_mode` | string | 设备名模式。可选值：`real`、`mask`、`custom`、`disabled`。   |
| `device_name`      | string | 设备名。仅 `device_name_mode=custom` 时生效。                |
| `mac_address_mode` | string | MAC 地址模式。可选值：`real`、`mask`、`custom`、`disabled`。 |
| `mac_address`      | string | MAC 地址。仅 `mac_address_mode=custom` 时生效。              |

### `privacy`

| 名称                           | 类型    | 说明                                                     |
| ------------------------------ | ------- | -------------------------------------------------------- |
| `do_not_track_mode`            | string  | Do Not Track。可选值：`default`、`enabled`、`disabled`。 |
| `battery_mode`                 | string  | 电池模式。可选值：`real`、`mask`、`custom`、`disabled`。 |
| `port_scan_protection_enabled` | boolean | 是否开启端口扫描保护。                                   |
| `hardware_acceleration_mode`   | string  | 硬件加速。可选值：`default`、`enabled`、`disabled`。     |

### `launch`

| 名称           | 类型   | 说明                                     |
| -------------- | ------ | ---------------------------------------- |
| `start_params` | string | 浏览器启动参数，多个参数用英文逗号分隔。 |

## 高级配置参数

`advanced` 用于配置启动、浏览器行为、同步、缓存、书签、访问限制和扩展。

| 名称               | 类型   | 说明                                                     |
| ------------------ | ------ | -------------------------------------------------------- |
| `startup`          | object | 启动配置。                                               |
| `browser_settings` | object | 浏览器设置。                                             |
| `data_sync`        | object | 环境数据同步设置。                                       |
| `local_cache`      | object | 本地缓存清理设置。                                       |
| `bookmarks`        | object | 书签设置。                                               |
| `access_limit`     | object | 访问限制设置。                                           |
| `extensions`       | object | 扩展设置。                                               |
| `multi_open`       | string | 多开模式。可选值：`global`、`allow`、`ban`。             |
| `remote_inspector` | string | 远程调试模式。可选值：`global`、`allow`、`ban`。         |
| `spoofing_video`   | string | 视频伪装模式。可选值：`default`、`enabled`、`disabled`。 |

### `startup`

| 名称                   | 类型     | 说明                                                       |
| ---------------------- | -------- | ---------------------------------------------------------- |
| `urls`                 | string[] | 启动时打开的网址列表。                                     |
| `fixed_urls`           | string[] | 固定网址列表。                                             |
| `restore_session_mode` | string   | 恢复会话模式。可选值：`global`、`restore`、`not_restore`。 |

### `browser_settings`

| 名称                                  | 类型    | 说明                                   |
| ------------------------------------- | ------- | -------------------------------------- |
| `scope`                               | string  | 应用方式。可选值：`global`、`custom`。 |
| `restore_last_page`                   | boolean | 是否恢复上次页面。                     |
| `block_images`                        | boolean | 是否屏蔽图片。                         |
| `block_video`                         | boolean | 是否屏蔽视频。                         |
| `mute_audio`                          | boolean | 是否静音。                             |
| `block_notifications`                 | boolean | 是否屏蔽网页通知。                     |
| `block_open_on_proxy_failure`         | boolean | 代理检测失败时是否阻止打开。           |
| `block_save_password_prompt`          | boolean | 是否禁止保存密码弹窗。                 |
| `disable_developer_tools`             | boolean | 是否禁用开发者工具。                   |
| `ignore_https_errors`                 | boolean | 是否忽略 HTTPS 证书错误。              |
| `disable_extension_management`        | boolean | 是否禁止管理扩展。                     |
| `random_fingerprint_on_launch`        | boolean | 是否每次启动随机指纹。                 |
| `disable_incognito`                   | boolean | 是否禁用无痕模式。                     |
| `hide_homepage`                       | boolean | 是否隐藏首页。                         |
| `extension_security`                  | boolean | 是否启用扩展安全。                     |
| `block_extension_store`               | boolean | 是否禁止访问扩展商店。                 |
| `disable_view_password`               | boolean | 是否禁止查看网站密码。                 |
| `block_on_proxy_country_change`       | boolean | 代理国家变化时是否阻止打开。           |
| `block_on_extension_download_failure` | boolean | 扩展下载失败时是否阻止打开。           |
| `disable_disk_write`                  | boolean | 是否禁止写盘。                         |

### `data_sync`

| 名称                  | 类型     | 说明                                                                                            |
| --------------------- | -------- | ----------------------------------------------------------------------------------------------- |
| `scope`               | string   | 应用方式。可选值：`global`、`custom`。                                                          |
| `items`               | string[] | 同步项列表，例如 `cookie`、`bookmark`、`account`、`local_storage`、`indexed_db`、`extensions`。 |
| `permission.enabled`  | boolean  | 是否开启同步权限控制。                                                                          |
| `permission.role_ids` | string[] | 允许同步的角色 ID 列表。                                                                        |

### `local_cache`

| 名称               | 类型     | 说明                                              |
| ------------------ | -------- | ------------------------------------------------- |
| `scope`            | string   | 应用方式。可选值：`global`、`custom`。            |
| `clear_mode`       | string   | 清理方式。可选值：`none`、`default`、`custom`。   |
| `items`            | string[] | 需要清理的项目列表。                              |
| `sync_after_clear` | boolean  | 清理后是否同步保存。                              |
| `frequency`        | string   | 清理频率。可选值：`every_open`、`custom_days`。   |
| `interval`         | integer  | 清理间隔天数。仅 `frequency=custom_days` 时生效。 |

### `bookmarks`

| 名称          | 类型    | 说明                                     |
| ------------- | ------- | ---------------------------------------- |
| `scope`       | string  | 应用方式。可选值：`global`、`custom`。   |
| `enabled`     | boolean | 是否启用书签设置。                       |
| `import_mode` | string  | 导入方式。可选值：`append`、`cover`。    |
| `cover_rule`  | string  | 覆盖规则。可选值：`overwrite`、`clear`。 |
| `file_name`   | string  | 书签文件名。创建和更新时通常无需填写。   |
| `content`     | object  | 书签内容，建议传浏览器导出的书签树结构。 |

### `access_limit`

| 名称              | 类型     | 说明                                      |
| ----------------- | -------- | ----------------------------------------- |
| `scope`           | string   | 应用方式。可选值：`global`、`custom`。    |
| `enabled`         | boolean  | 是否启用访问限制。                        |
| `policy`          | string   | 限制策略。可选值：`block`、`allow_only`。 |
| `quick_selection` | string[] | 快捷选择项，例如 `google_play`。          |
| `urls`            | string   | 网址列表，多行文本。                      |

### `extensions`

| 名称       | 类型   | 说明                               |
| ---------- | ------ | ---------------------------------- |
| `mode`     | string | 扩展模式。可选值：`allow`、`ban`。 |
| `group_id` | string | 扩展分组 ID。                      |

## 账号参数

| 名称       | 类型   | 必填 | 说明                                                                                                                                                                                 |
| ---------- | ------ | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `platform` | string | 否   | 账号平台。常用值：`other`、`facebook.com`、`amazon.com`、`linkedin.com`、`x.com`、`paypal.com`、`accounts.google.com`、`youtube.com`、`ebay.com`、`tiktok.com`、`instagram.com` 等。 |
| `username` | string | 否   | 登录账号。                                                                                                                                                                           |
| `password` | string | 否   | 登录密码。                                                                                                                                                                           |
| `secret`   | string | 否   | 2FA 密钥或账号密钥。                                                                                                                                                                 |
| `url`      | string | 否   | 自定义网站 URL。`platform=other` 时必填，且必须以 `http://` 或 `https://` 开头。                                                                                                     |
| `remark`   | string | 否   | 账号备注。                                                                                                                                                                           |

## Cookie 参数

`CookieImportRequest`：

| 名称      | 类型     | 必填 | 说明                                                                         |
| --------- | -------- | ---- | ---------------------------------------------------------------------------- |
| `mode`    | string   | 否   | 导入模式。可选值：`replace`、`merge`。                                       |
| `cookies` | object[] | 否   | Cookie 列表。单次最多 5000 个；导入时每项的 `domain`、`name`、`value` 必填。 |

`CookieItem`：

| 名称             | 类型    | 必填 | 说明                                                                      |
| ---------------- | ------- | ---- | ------------------------------------------------------------------------- |
| `domain`         | string  | 否   | 域名。导入 Cookie 时必填。                                                |
| `path`           | string  | 否   | 路径。                                                                    |
| `name`           | string  | 否   | Cookie 名称。导入 Cookie 时必填。                                         |
| `value`          | string  | 否   | Cookie 值。导入 Cookie 时必填，可为空字符串但不能为 `null`。              |
| `expirationDate` | integer | 否   | 过期时间，Unix 时间戳。                                                   |
| `httpOnly`       | boolean | 否   | 是否 HttpOnly。                                                           |
| `secure`         | boolean | 否   | 是否 Secure。                                                             |
| `sameSite`       | string  | 否   | SameSite 策略。可选值：`unspecified`、`no_restriction`、`strict`、`lax`。 |
| `hostOnly`       | boolean | 否   | 是否 HostOnly，兼容扩展 Cookie 格式。                                     |
| `session`        | boolean | 否   | 是否会话 Cookie，兼容扩展 Cookie 格式。                                   |
| `storeId`        | string  | 否   | Cookie Store ID，兼容扩展 Cookie 格式。                                   |
| `hasExpires`     | boolean | 否   | 是否显式设置过期时间，兼容扩展 Cookie 格式。                              |
| `priority`       | string  | 否   | 优先级，兼容扩展 Cookie 格式。                                            |
| `isSameParty`    | boolean | 否   | 是否 SameParty。                                                          |

## 代理参数

`ProxyUpsertRequest` 用于创建和修改代理。

| 名称             | 类型    | 必填 | 说明                                                         |
| ---------------- | ------- | ---- | ------------------------------------------------------------ |
| `type`           | string  | 否   | 代理类型。可选值：`none`、`http`、`https`、`ssh`、`socks5`。 |
| `host`           | string  | 否   | 代理主机。                                                   |
| `port`           | integer | 否   | 代理端口。                                                   |
| `user`           | string  | 否   | 代理账号。                                                   |
| `password`       | string  | 否   | 代理密码。                                                   |
| `ipchecker`      | string  | 否   | IP 检测渠道。可选值：`ip2location`、`ipapi`。                |
| `ip_version`     | string  | 否   | IP 地址类型。可选值：`ipv4`、`ipv6`。                        |
| `remark`         | string  | 否   | 备注。                                                       |
| `proxy_group_id` | string  | 否   | 代理分组 ID。                                                |

## 代理检测参数

`ProxyCheckRequest` 用于检测临时代理。

| 名称         | 类型    | 必填 | 说明                                                         |
| ------------ | ------- | ---- | ------------------------------------------------------------ |
| `type`       | string  | 否   | 代理类型。可选值：`none`、`http`、`https`、`ssh`、`socks5`。 |
| `host`       | string  | 否   | 代理主机。                                                   |
| `port`       | integer | 否   | 代理端口。                                                   |
| `user`       | string  | 否   | 代理账号。                                                   |
| `password`   | string  | 否   | 代理密码。                                                   |
| `ipchecker`  | string  | 否   | IP 检测渠道。可选值：`ip2location`、`ipapi`。                |
| `ip_version` | string  | 否   | IP 地址类型。可选值：`ipv4`、`ipv6`。                        |
| `timeout_ms` | integer | 否   | 超时时间，单位毫秒。                                         |

## 环境摘要字段

| 名称             | 类型     | 说明                                               |
| ---------------- | -------- | -------------------------------------------------- |
| `id`             | string   | 环境 ID。                                          |
| `serial_no`      | integer  | 环境序号。                                         |
| `name`           | string   | 环境名称。                                         |
| `status`         | string   | 环境状态。可选值：`enabled`、`disabled`。          |
| `run_status`     | string   | 运行状态。可选值：`stopped`、`running`、`locked`。 |
| `browser`        | object   | 浏览器配置。                                       |
| `os`             | string   | 操作系统。                                         |
| `groups`         | object[] | 分组列表。                                         |
| `tags`           | object[] | 标签列表。                                         |
| `proxy_binding`  | object   | 环境代理绑定配置。                                 |
| `proxy_summary`  | object   | 代理摘要。                                         |
| `created_at`     | string   | 创建时间。                                         |
| `updated_at`     | string   | 更新时间。                                         |
| `last_opened_at` | string   | 最近打开时间。                                     |
| `remark`         | string   | 备注。                                             |

## 环境详情字段

环境详情包含环境摘要字段，并额外包含：

| 名称          | 类型     | 说明           |
| ------------- | -------- | -------------- |
| `group_ids`   | string[] | 分组 ID 列表。 |
| `tag_ids`     | string[] | 标签 ID 列表。 |
| `fingerprint` | object   | 指纹配置。     |
| `advanced`    | object   | 高级配置。     |

## 代理响应字段

| 名称             | 类型    | 说明                                                         |
| ---------------- | ------- | ------------------------------------------------------------ |
| `proxy_id`       | string  | 代理 ID。                                                    |
| `serial_no`      | integer | 代理序号。                                                   |
| `type`           | string  | 代理类型。可选值：`none`、`http`、`https`、`ssh`、`socks5`。 |
| `host`           | string  | 代理主机。                                                   |
| `port`           | integer | 代理端口。                                                   |
| `user`           | string  | 代理账号。                                                   |
| `password`       | string  | 代理密码。                                                   |
| `ipchecker`      | string  | IP 检测渠道。可选值：`ip2location`、`ipapi`。                |
| `ip_version`     | string  | IP 地址类型。可选值：`ipv4`、`ipv6`。                        |
| `proxy_group_id` | string  | 代理分组 ID。                                                |
| `remark`         | string  | 备注。                                                       |

## 指纹可选项响应字段

`GET /openapi/v2/fingerprints/options` 的 `data` 可能包含：

| 名称                                   | 类型     | 说明                   |
| -------------------------------------- | -------- | ---------------------- |
| `os_options`                           | string[] | 系统类型选项。         |
| `kernel_version_options`               | string[] | 浏览器内核版本选项。   |
| `proxy_type_options`                   | string[] | 代理类型选项。         |
| `ip_check_provider_options`            | string[] | IP 检测渠道选项。      |
| `restore_session_options`              | string[] | 恢复会话选项。         |
| `fingerprint_language_mode_options`    | string[] | 浏览器语言模式选项。   |
| `fingerprint_ui_language_mode_options` | string[] | 界面语言模式选项。     |
| `timezone_mode_options`                | string[] | 时区模式选项。         |
| `geolocation_permission_options`       | string[] | 地理位置权限选项。     |
| `geolocation_source_options`           | string[] | 地理位置来源选项。     |
| `fingerprint_control_mode_options`     | string[] | 通用指纹控制模式选项。 |
| `fingerprint_value_mode_options`       | string[] | 指纹取值模式选项。     |
| `toggle_mode_options`                  | string[] | 开关继承模式选项。     |
| `web_rtc_mode_options`                 | string[] | WebRTC 模式选项。      |
| `web_rtc_ip_source_options`            | string[] | WebRTC IP 来源选项。   |
| `resolution_mode_options`              | string[] | 屏幕分辨率模式选项。   |
| `window_size_mode_options`             | string[] | 窗口尺寸模式选项。     |

# 常见调用顺序

## 创建并打开一个环境

1. 创建环境：

```http
POST /openapi/v2/profiles
```

2. 从响应 `data.id` 取得环境 ID。

3. 打开环境：

```http
PATCH /openapi/v2/profiles/{profileId}/open
```

4. 使用响应中的 `debug_port` 或 `web_socket_url` 连接浏览器调试协议。

5. 关闭环境：

```http
PATCH /openapi/v2/profiles/{profileId}/close
```

## 给环境绑定已有代理

1. 创建代理：

```http
POST /openapi/v2/proxies
```

2. 从响应 `data.proxy_id` 取得代理 ID。

3. 绑定到环境：

```http
PATCH /openapi/v2/profiles/{profileId}/proxy-binding
```

请求体：

```json
{
  "proxy_binding": {
    "mode": "linked",
    "proxy_id": "proxy-id"
  }
}
```

# 调用提示

- 所有 V2 接口路径都需要包含 `/openapi/v2`。
- 所有请求都需要携带正确的 `X-API-KEY`。
- 有请求体的接口请使用 JSON 格式，并设置 `Content-Type: application/json`。
- 创建、更新、导入类接口建议先使用少量数据验证，再批量调用。
- 打开环境成功后，可以使用响应中的 `debug_port` 或 `web_socket_url` 连接浏览器。
- 遇到 429 时请降低请求频率，并参考响应头中的 `Retry-After`、`X-RateLimit-*`。
