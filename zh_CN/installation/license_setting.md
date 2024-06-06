# 安装许可证

许可证可以多次安装，重新安装后，旧的许可证将会被删除。

## 一次性激活模式

重新安装许可证后，之前的许可证剩余可用的实例数量将被清空。

申请许可证后，可通过 [api](https://docs.emqx.com/zh/sdv-flow/latest/api/api-docs.html#tag/license) 的方式，上传许可证。

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/license' \
--header 'Content-Type: application/json' \
--data-raw '{
	"license":"b25lLXRpb******"
}'
```

返回结果

```json
{
    "licenseNumber": "22573eaa1e",
    "version": 1,
    "mode": "one-time-activation",
    "licenseType": "OFFICIAL",
    "customerName": "customer",
    "customerContact": "support@emqx.io",
    "validFrom": "20240517",
    "totalInstances": 100000,
    "availableInstances": 100000,
    "sumUsedInstances": 0
}
```

上传成功后，可通过 [api](https://docs.emqx.com/zh/sdv-flow/latest/api/api-docs.html#tag/license) 查看详细信息。

```shell
curl --location --request GET 'http://127.0.0.1:8082/api/license' 
```

返回结果

```json
{
    "licenseNumber": "22573eaa1e",
    "version": 1,
    "mode": "one-time-activation",
    "licenseType": "OFFICIAL",
    "customerName": "customer",
    "customerContact": "support@emqx.io",
    "validFrom": "20240517",
    "totalInstances": 100000,
    "availableInstances": 100000,
    "sumUsedInstances": 0
}
```

