# 灵活更新规则

SDV Platform 提供了一种灵活的方法，通过 REST API 远程更新车端 SDV Flow 的规则。本文档将指导您如何使用这些 API 来实现云端对车端规则的管理和更新。API的调用需要鉴权，可参考[文档](./jwt.md)。

## 管理单个车端

要通过云端更新车端的 SDV Flow 规则，您需要构造特定的 HTTP 请求。以下是请求的构成要素：

1. **基本 URL 结构**：
   
- `http://localhost:8082/api/edgeservice/proxy/${vin}`
   
其中 `${vin}` 是车辆识别码（VIN），用于指定目标车辆。
   
2. **SDV Flow API 路径**：
   
- 将 SDV Flow 的 API 路径附加到基本 URL 之后。
   
3. **HTTP 方法**：
   
- 使用相应的 HTTP 方法，如 POST、GET、PUT、DELETE 等，与 SDV Flow 的 API 方法相对应。
   
4. **头部信息**：
   
   - 相应的 SDV Flow 的 API 需要的请求头，如 `Content-Type: application/json`
   - `_emqx_request_id_`:request_id,可通过 订阅 emqx 的 topic `agent/{vin}/proxy/ack/{request_id}`  的方式来获取下发结果, 若未设置，则会随机生成。
   
5. **请求正文（Body）**：

   - 根据 SDV Flow 的 API 要求，提供相应的 JSON 格式的请求正文。

### 示例

- **创建规则**

```
POST http://localhost:8082/api/edgeservice/proxy/${vin}/rules
```

请求正文示例：
```json
{
  "id": "rule1",
  "sql": "SELECT * FROM demo",
  "actions": [
    {
      "log": {}
    }
  ]
}
```

- **获取所有规则及其状态**

```
GET http://localhost:8082/api/edgeservice/proxy/${vin}/rules
```

- **获取规则状态**

```
GET http://localhost:8082/api/edgeservice/proxy/${vin}/rules/{id}/status
```

其中 `{id}` 是您想要查询状态的规则的 ID。

- **更新规则**

```
PUT http://localhost:8082/api/edgeservice/proxy/${vin}/rules/{id}
```

请求正文示例与创建规则类似。

- **删除规则**

要删除一个规则，发送一个 DELETE 请求到：

```
DELETE http://localhost:8082/api/edgeservice/proxy/${vin}/rules/{id}
```

- **启动、停止和重启规则**

要分别启动、停止或重启规则，发送 POST 请求到：

```
POST http://localhost:8082/api/edgeservice/proxy/${vin}/rules/{id}/start
POST http://localhost:8082/api/edgeservice/proxy/${vin}/rules/{id}/stop
POST http://localhost:8082/api/edgeservice/proxy/${vin}/rules/{id}/restart
```

### 注意事项
- 确保替换 `${vin}` 为实际车辆的 VIN 码。
- 请求正文（Body）需要遵循 SDV Flow API 所要求的 JSON 格式。
- 检查 API 文档以获取更多关于请求参数和响应格式的详细信息。

通过遵循本上面的方法，您可以轻松地通过云端 SDV Platform 对车端 SDV Flow 规则进行灵活管理和更新。其他车端支持的API,可以查看[文档](https://ekuiper.org/docs/zh/latest/api/restapi/overview.html)，您可根据上面的方法灵活调用所有的SDV Flow 支持的的APi。

## 批量管理车端规则

### 批量下发API

- **URL**：`http://localhost:8082/api/edgeservices/config`

- **请求方法**：`POST`

- **头部信息**：

  - `Content-Type: application/json`
  - `_emqx_request_id_prefix_`: request_id 前缀，可选，若指定前缀，下发成功后，可以通过订阅 emqx 的 topic ecp/{vin}/proxy/ack/{request_id} （或者使用通配符ecp/{vin}/proxy/ack/+）来获取部署结果。其中{request_id}为`_emqx_request_id_prefix_`的值加序号生成（如果未设置`_emqx_request_id_prefix_`，则随机生成{request_id}）

- **请求正文**：包含车辆 VIN 列表、SDV Flow API 方法、路径和正文。

  - **ids**：车辆识别码（VIN）列表，指定要管理的目标车辆列表。
  - **method**：SDV Flow 的 API 方法，例如 `POST`、`GET`、`PUT`、`DELETE` 等。
  - **path**：SDV Flow 的 API 路径，例如 `/rules`。
  - **body**：SDV Flow 的 API 请求正文的 JSON 字符串。
  - **category**: 服务类型 1:流处理引擎 3:消息总线，默认为1

- **响应结果**：例如

  ```
  {"id": "7527e427"}
  ```

  id 本次下发的任务id，可用于后续获取批量下发结果

  ### 示例

  - ##### 批量创建规则

  ```json
  curl --location --request POST 'http://localhost:8082/api/edgeservices/config' \
  --header 'Content-Type: application/json' \
  --data-raw '{
      "ids": ["vin1", "vin2", "vin3"],
      "method": "POST",
      "path": "/rules",
      "body": "{\"id\":\"rule1\",\"sql\":\"SELECT * FROM candata\",\"actions\":[{\"mqtt\":{\"sendSingle\":true,\"server\":\"tcp://127.0.0.1:1883\",\"topic\":\"results/rule1\"}}]}"
   }'
  ```

  - ##### 批量创建流

  ```json
  curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
  --header 'Content-Type: application/json' \
  --data-raw '{
      "ids": [
          "DV_PV_TASK_ENABLE"
      ],
      "method": "POST",
      "path": "/streams",
      "body": "{\"sql\": \"CREATE STREAM triggerStream() WITH (TYPE=\\\"memory\\\", FORMAT=\\\"json\\\", DATASOURCE=\\\"trigger\\\")\"}"
  }'
  ```

### 获取批量下发结果API

- **URL**：`http://localhost:8082/api/edgeservices/config/result?id=`

  - **id**：任务 ID，用于查询特定批量任务的结果。

- **请求方法**：`GET`

- **响应结果**：例如

  ```json
  {
    "processing": [
      "vin1",
      "vin2"
    ],
    "success": [
      "vin3",
      "vin4"
    ],
    "failed": [
      "vin5"
    ],
    "failed_reasons": {
      "vin5": "更新超时"
    }
  }
  ```

  服务器将返回一个 JSON 对象，包含以下字段：

  - **processing**：仍在处理中的车辆 VIN 列表。
  - **success**：已成功更新规则的车辆 VIN 列表。
  - **failed**：更新规则失败的车辆 VIN 列表。
  - **failed_reasons**：失败原因的详细描述，以属性名称为键，失败原因为值。

### **示例**

```shell
curl --location --request GET 'http://127.0.0.1:8082/api/edgeservices/config/result?id=d7aff271' \
```

