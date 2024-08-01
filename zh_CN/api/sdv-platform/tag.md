# 标签管理

## 1. 管理员接口

### 1.1 创建标签

**请求**

- **方法**: POST
- **路径**: /orgs/{orgid}/projects/{projectid}/tags

**请求体**
```json
{
    "name": "example-tag",
    "serviceIds": ["service1", "service2", "service3"]
}
```

**响应**

```json
{
    "id": 1,
    "name": "example-tag",
    "tagged": 5,
    "failedServiceNames": ["service1", "service2"],
    "createdAt": "2024-05-21T14:35:00Z"
}
```

**异常处理**

- 标签名需匹配正则表达式 `"^[\u4e00-\u9fa5A-Za-z0-9-—\\s]{1,10}$"`
- 检测 `orgid` 和 `projectid` 是否存在
- 若为试用版且标签数量达到或超过3，返回错误
- 若非试用版且标签数量达到或超过100，返回错误
- 检测标签名是否已存在
- 检测服务ID是否存在
- 如果服务绑定的标签数量大于等于10，则返回错误

### 1.2 更新标签

**请求**

- **方法**: PUT
- **路径**: /orgs/{orgid}/projects/{projectid}/tags/{tagid}

**请求体**
```json
{
    "name": "updated-tag",
    "serviceIDs": ["service1", "service2", "service3"]
}
```

**响应**

```json
{
    "id": 1,
    "name": "updated-tag",
    "tagged": 5,
    "failedServiceNames": ["edgeService1", "edgeService2"],
    "createdAt": "2024-01-01T12:00:00Z",
    "updatedAt": "2024-05-21T14:35:00Z"
}
```

**异常处理**

- 标签名需匹配正则表达式 `"^[\u4e00-\u9fa5A-Za-z0-9-—\\s]{1,10}$"`
- 检测 `orgid` 和 `projectid` 是否存在
- 根据 `projectid` 和 `tagid` 查找记录，如果不存在则返回错误
- 检查标签名是否已存在

### 1.3 删除标签

**请求**

- **方法**: DELETE
- **路径**: /orgs/{orgid}/projects/{projectid}/tags/{tagid}

**异常处理**

- 检测 `orgid` 和 `projectid` 是否存在
- 如果 `tagid` 不存在，返回错误

## 2. 项目成员接口

### 2.1 分页获取标签列表

**请求**

- **方法**: GET
- **路径**: /orgs/{orgid}/projects/{projectid}/tags
- **查询参数**:
  - `name` (可选): 标签名
  - `size` (可选): 返回结果数量，默认为10
  - `offset` (可选): 偏移量，默认为0

**请求示例**
```
GET /orgs/{orgid}/projects/{projectid}/tags?name=example&size=10&offset=0
```

**响应**

```json
{
    "data": [
        {
            "id": 1,
            "name": "example-tag",
            "tagged": 5,
            "createdAt": "2024-04-21T14:35:00Z",
            "updatedAt": "2024-05-21T14:35:00Z"
        },
        {
            "id": 2,
            "name": "another-tag",
            "tagged": 3,
            "createdAt": "2024-04-30T14:35:00Z",
            "updatedAt": "2024-05-21T14:35:00Z"
        }
    ],
    "total": 2
}
```

**异常处理**

- 检测 `orgid` 和 `projectid` 是否存在
- 如果标签不存在，返回错误

### 2.2 获取与指定标签关联的边缘服务

**请求**

- **方法**: GET
- **路径**: /orgs/{orgid}/projects/{projectid}/tags/{tagid}/edgeservices

**请求示例**
```
GET /orgs/{orgid}/projects/{projectid}/tags/{tagid}/edgeservices
```

**响应**

```json
{
    "data": [
        {
            "id": "ekuiper-ecs-2c63",
            "name": "ekuiper-ecs-2c63"
        }
    ]
}
```

**异常处理**

- 检测 `orgid` 和 `projectid` 是否存在
- 根据 `projectid` 和 `tagid` 查找，如果不存在则返回错误