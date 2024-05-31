# CAN 格式

<span style="background:green;color:white;padding:1px;margin:2px">动态 Schema</span>

CAN 格式读取 CAN 帧字节并将其解码为可读的信号。它使用 [DBC](./dbc.md) 作为模式。

## 数据示例

CAN 格式支持读取 CAN 帧或者 CAN FD 帧。CAN 帧请参考 [socket can](https://www.kernel.org/doc/html/next/networking/can.html) 的定义，如下所示：

```c
struct can_frame {
        canid_t can_id;  /* 32 bit CAN_ID + EFF/RTR/ERR flags */
        union {
                /* CAN frame payload length in byte (0 .. CAN_MAX_DLEN)
                 * was previously named can_dlc so we need to carry that
                 * name for legacy support
                 */
                __u8 len;
                __u8 can_dlc; /* deprecated */
        };
        __u8    __pad;   /* padding */
        __u8    __res0;  /* reserved / padding */
        __u8    len8_dlc; /* optional DLC for 8 byte payload length (9 .. 15) */
        __u8    data[8] __attribute__((aligned(8)));
};

struct canfd_frame {
        canid_t can_id;  /* 32 bit CAN_ID + EFF/RTR/ERR flags */
        __u8    len;     /* frame payload length in byte (0 .. 64) */
        __u8    flags;   /* additional flags for CAN FD */
        __u8    __res0;  /* reserved / padding */
        __u8    __res1;  /* reserved / padding */
        __u8    data[64] __attribute__((aligned(8)));
};
```

示例 CAN 帧字节输入如下:

```
0x2b020000080000005465737400000000
```

解析结果如下。请注意，模式是由DBC定义的，不同的DBC设置将产生不同的结果。

```json
{
  "Mess0": {
    "Mess0_Sig1": 100,
    "Mess0_Sig2": 1000,
    "Mess0_Sig3": 0
  }
}
```

## CAN ID 白名单

CAN 总线上包含各种类型的数据。不同类型的消息通过 CAN ID 来区分。

在某些情况下，用户可能只关心某些信号。解析所有信号将会是很大的浪费。CAN 格式支持只解析所需的消息。所需的消息（白名单）将根据用户的规则进行自动计算，无需额外配置。影响白名单计算的因素包括：

- 规则的信号使用情况。例如，如果流只被一条规则使用，SQL 语句为 `SELECT Mess0.Mess0_Sig1 FROM canStream` 。那么它将只解码 Mess0 消息。
- 如果流被多个规则共享，白名单将根据规则的启动/停止动态更新。
- DBC 文件可视为信号级别的白名单。在 DBC 中未定义的信号/消息将被忽略。
