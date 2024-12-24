# syslog server

## syslog server 整体架构

Syslog 服务器的主要功能包括：通过 UDS 按照 [RFC 3164](https://www.rfc-editor.org/rfc/rfc3164.html) 格式收集组件日志，处理日志格式，支持日志的周期性轮转，并可选将日志写入 logcat 或周期性上传。下面是流程图以及说明：

```mermaid
flowchart TD
    %% Modules and Initial Log Transmission
    A1[nanomq Logs] --UDS--> B1[sdv-flow Syslog Server]
    A2[ekuiper Logs] --UDS--> B1
    A3[geely-api Logs] --UDS--> B1


    %% Syslog Server Processing
    B1 --> C1[Format Logs to RFC 3164]
    C1 --> C2[Periodically Rotate Logs]
    C1 -->|Logcat Enabled| D1[Write Logs to Logcat]

    C2 -->|File Transfer Enabled| D2[Compress Logs with Zstd]

    %% Compression and File Transfer
    D2 --> D3[Rename Compressed File]
    D3 --> D4[Nano File Transfer]
    D4 --> D5[Encrypt File]
    D5 --> D6[Transmit File]


    
    %% Subgraph for Modules
    subgraph Log Sources
        A1
        A2
        A3
    end

    %% Subgraph for Syslog Server and File Transfer
    subgraph Syslog server and File Transfer
        B1
        C1
        C2
        subgraph Optional Actions
            D1
            D2
            D3 
        end
        subgraph Nano File Transfer Process
            D4
            D5 
            D6
        end
    end
```
### 流程说明：
1. **日志源**: 
    - nanomq、ekuiper、geely-api 和 sdv-flow 模块通过 UDS 将日志发送到 sdv-flow 的 Syslog 服务器。
2. **Syslog 处理**:
    - 在服务器端，日志格式化为 RFC 3164 格式。
    - 日志进行周期性轮转保存。
3. **可选处理**:
    - 如果开启了 logcat，日志被写入 logcat。
    - 如果开启了文件传输，日志被压缩为 Zstd 格式。
4. **文件压缩和传输**:
    - 压缩的日志文件被重命名。
    - 调用 Nano File Transfer 功能：
        - 对文件进行加密。
        - 通过加密通道传输文件。
5. **成功与失败处理**:
    - 传输处理可参考 [mqtt 消息/文件传输功能](../../communication-databus/file-transfer.md) 。
