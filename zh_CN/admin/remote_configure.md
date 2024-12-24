# 远程配置

## 数据总线配置文件管理

为了更好地管理和配置车端的 Nanomq 服务，云端提供了 API 接口，支持查询和下发 Nanomq 配置文件。通过这些 API，用户可以轻松地获取当前车端的 Nanomq 配置，并根据需要进行修改和更新。下发的配置文件将在下次车辆启动后生效，确保 Nanomq 服务按照新的配置运行。

#### 1. 查询 Nanomq 配置文件内容
- **接口地址**：`GET /api/edgeservice/proxy/agent/{VIN}/configs/file?category=3`
- **功能描述**：用于查询指定车辆的 Nanomq 配置文件内容。
- **请求示例**：
    ```bash
    curl --location --request GET 'http://123.249.85.169:8082/api/edgeservice/proxy/agent/1111111/configs/file?category=3' \
    --header '_emqx_request_id_: demotest'
    ```
- **返回结果**：返回完整的 Nanomq 配置文件内容，例如：
    ```plaintext
    # NanoMQ Configuration 

    # #============================================================
    # # NanoMQ Broker
    # #============================================================

    mqtt {
        property_size= 80
        max_packet_size = 255MB
        max_mqueue_len = 3333
        retry_interval = 10s
        keepalive_multiplier = 1.25

        # Three of below, unsupported now
        max_inflight_window = 2048
        max_awaiting_rel = 10s
        await_rel_timeout = 10s
    }
    ........
    ```

#### 2. 下发 Nanomq 配置文件内容（单车）
- **接口地址**：`POST /api/edgeservice/proxy/agent/{VIN}/configs/file?category=3`
- **功能描述**：用于向指定车辆下发新的 Nanomq 配置文件内容。
- **请求示例**：
    ```bash
    curl --location --request POST 'http://123.249.85.169:8082/api/edgeservice/proxy/agent/1111111/configs/file?category=3' \
    --header '_emqx_request_id_: demotest' \
    --header 'Content-Type: text/plain' \
    --data-raw '# NanoMQ Configuration 11111111

    # #============================================================
    # # NanoMQ Broker
    # #============================================================

    mqtt {
        property_size= 80
        max_packet_size = 255MB
        max_mqueue_len = 3333
        retry_interval = 10s
        keepalive_multiplier = 1.25

        # Three of below, unsupported now
        max_inflight_window = 2048
        max_awaiting_rel = 10s
        await_rel_timeout = 10s
    }

    listeners.tcp {
        bind = "0.0.0.0:1883"
    }
    ......'
    ```
- **返回结果**：
    ```json
    {
        "code": 0
    }
    ```

#### 3. 批量下发 Nanomq 配置文件内容
- **接口地址**：`POST /api/edgeservices/config`
- **功能描述**：用于批量向多辆车下发新的 Nanomq 配置文件内容。
- **请求示例**：
    ```bash
    curl --location --request POST 'http://123.249.85.169:8082/api/edgeservices/config' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "ids": [
            "1111111"
        ],
        "method": "POST",
        "path": "/configs/file",
        "query":"category=3",
        "category":2,
        "body": "# NanoMQ Configuration 3333333\n\n# #============================================================\n# # NanoMQ Broker\n# #============================================================\n\nmqtt {\n\tproperty_size= 80\n\tmax_packet_size = 255MB\n\tmax_mqueue_len = 3333\n\tretry_interval = 10s\n\tkeepalive_multiplier = 1.25\n\n\t# Three of below, unsupported now\n\tmax_inflight_window = 2048\n\tmax_awaiting_rel = 10s\n\tawait_rel_timeout = 10s\n}\n\nlisteners.tcp {\n\tbind = \"0.0.0.0:1883\"\n}\n\n# listeners.ssl {\n#\tbind = \"0.0.0.0:8883\"\n#\tkeyfile = \"/etc/certs/key.pem\"\n#\tcertfile = \"/etc/certs/cert.pem\"\n#\tcacertfile = \"/etc/certs/cacert.pem\"\n#\tverify_peer = false\n#\tfail_if_no_peer_cert = false\n# }\n\nhttp_server {\n\tport = 8081\n\tlimit_conn = 2\n\tusername = admin\n\tpassword = public\n\tauth_type = no_auth\n}\n\nlog {\n\tto = [file, console]\n\tlevel = warn\n\tdir = \"./log\"\n\tfile = \"nanomq.log\"\n\trotation {\n\t\tsize = 10MB\n\t\tcount = 5\n\t}\n}\n\nauth {\n\tallow_anonymous = true\n\tno_match = allow\n\tdeny_action = ignore\n\n\tcache = {\n\t\tmax_size = 32\n\t\tttl = 1m\n\t}\n\n}\n# ......'
    }'
    ```
- **返回结果**：
    ```json
    {
        "id": "4b6e02a5"
    }
    ```

#### 注意事项
- **接口调用限制**：请合理使用这些 API 接口，避免频繁调用导致系统负载过高。
- **配置文件格式**：下发的 Nanomq 配置文件内容必须符合 Nanomq 的配置文件格式，否则可能导致配置下发失败或 Nanomq 服务运行异常。
- **配置生效时间**：下发的配置文件将在下次车辆启动后生效，请确保在下发配置后及时重启车辆。
- **网络连接要求**：调用 API 接口时，需要确保车端与云端的网络连接稳定，以免因网络问题导致配置下发失败。

通过以上 API 接口，用户可以方便地查询和管理车端的 Nanomq 配置文件，确保 Nanomq 服务按照预期的配置运行，满足物联网通信的需求。