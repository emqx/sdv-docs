# CANJSON 格式

<span style="background:green;color:white;padding:1px;margin:2px">dynamic schema</span>

CANJSON 格式是一种组合格式，用于读取包含在自定义JSON格式中的CAN帧字节，并将其解码为可读的信号。 它使用 [DBC](./dbc.md) 作为模式。

**注意**: 该格式为自定义的格式，非通用格式。后期可能仍会有变化。

## 数据样例

CANJSON 格式解码自定义的嵌入 CAN 数据的特定JSON。它是一种聚合格式，通常由前置网关接入 CAN 总线后，聚合 CAN 帧后透传而来。其数据结构包含任意数量的 CAN 帧和元数据。数据结构如下示例：

```json
{
  "ts": 1617180010000,
  "frames": [
    {
      "id": 1006,
      "bus": 5,
      "d": 0,
      "t": 1617179000000,
      "data": "5465737400000000546573740000AB00"
    },
    {
      "id": 1414,
      "bus": 6,
      "d": 0,
      "t": 1617179000100,
      "data": "5465737400000000"
    }
  ]
}
```

解析结果如下。请注意，模式是由DBC定义的，不同的DBC设置将产生不同的结果。

```json
{
  "RVB_TVR_Debug2_FO": {
    "id": 1414,
    "bus": 6,
    "d": 0,
    "t": 1617179000100,
    "VBBrkCntlAccel": 0.0,
    "VBTOSLatPstn": 87.125,
    "VBTOSLonPstn": 168.75,
    "VBTOSObjID": 0.0,
    "VBTOSTTC": 46.4
  },
  "raw": {
    "RVB_TVR_Debug2_FO": "1617180010000 6 586 Rx d 8 54 65 73 74 00 00 00 00"
  },
  "ts": 1617180010000
}
```

## CAN ID 白名单

与 [CAN 格式](./can_format.md#can-id-白名单) 的白名单策略相同。