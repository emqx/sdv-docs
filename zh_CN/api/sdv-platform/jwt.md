# 认证

在 Platform 中调用 API 时，需先调用登录接口生成token，默认生成的 token 过期时间为4小时，可以通过api 自定义过期时间。

当用户请求 RESTful API 时，请在 http 请求头中按以下格式输入 **Token**：

```
Authorization: Bearer
							XXXXXXXXXXXXXXX
```



如果 **Token** 正确，Platform 将响应结果； 否则，它将返回`401`代码。

## 获取 Token

访问登陆 API

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/account/login' \
--header 'Content-Type: application/json' \
--data-raw '{"username":"admin@emqx.io","password":"admin@emqx.io"}'
```

返回结果中 的`accessToken` 即为token。

```shell
{
    "tokenType": "Bearer",
    "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjFjNjRkZWNlIiwiZXhwIjoxNzE3NTkzMDEwLCJuYmYiOjE3MTc1Nzg2MTAsImlhdCI6MTcxNzU3ODYxMCwianRpIjoiY2RjMjBkZTUtYWI3YS00M2FiLWJhMzAtODhiYmVjYTkzYzU3IiwidHlwIjoiQmVhcmVyIn0.RYzwi4nxRgpsk_XUcMnf8qd0VoBiBDCDonJ5zE0YeP1taTmEAtdxt3l015-q2LHPXClD6dyo7aUJ8aXcfJMwfttNqoEwV3qpU_dTkZ3pjMdfU9L8Cgfauji2x_clxcegVVi_98I1a7c9ttgnkH6MWexbmoavJCg9XCf5zeLQdfzriH4bfCHjlaUdw-HWxWBbN0iZi68Bs4ISD7KapkzGIvcmFqa9lTr2qyL_h0aGCq2zxmVb2fRyAs2WBAO29PLvu7IUPjz_I4ZkNFlHG3obhKqjlRzs2qNSo-CBbGpoIbK6_plgsvGjgL2N1Zmkd9--_YBYsGlxbiCgM7E8Lef05w",
    "expiresIn": 14400,
    "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjFjNjRkZWNlIiwiZXhwIjoxNzE3NjY1MDEwLCJuYmYiOjE3MTc1Nzg2MTAsImlhdCI6MTcxNzU3ODYxMCwianRpIjoiY2FmNTJiMWMtYTQ0NC00YzEwLTg4ZDktYjRiMmUzYWUxZjNkIiwidHlwIjoiUmVmcmVzaCJ9.SLiygX83ybnw8_Ml3myjZNEhV3Xcx4V35w0N_LNAl80aXybVk8V-IubHAi5vDxG3gXK1vAbvyCWJlbghD1nqNtdKYc5zrwj2t3iYul79gaZ4QQ5wKIFit_0YL83r3N2s8IKlnfsatwozTguqynmHTIrgv8g4jXyalpFaxH56XwBNsW6d95l2w0sQuIN4RfELCIR1yDqDxCsKoUxPYRtWbUuaUXgialFwGNOPZzutRo4mdzTYXMhqkeh-Nk68R2eSsz0ZCPf6MmeicwMwImtxqMUFl17FnJt4_YvN8PX2zVEYt__G7qeCFxCPhPO_D8ttYH28UBtgXHrQzVhHf78K9Q",
    "refreshExpiresIn": 86400,
    "admin": true
}
```

