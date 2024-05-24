# 数据查询与上传

## 概述

本文档介绍了如何通过 NanoMQ 与 eKuiper 的集成来实现数据查询和上传功能。eKuiper 通过 IPC 通信（`ipc:///tmp/nanomq_hook.ipc`）触发文件搜索，搜索到的文件将被上传到云端。具体操作包括落盘搜索命令的格式和搜索结果的上传。

### 1. 落盘搜索命令格式

要触发数据搜索，使用如下格式的命令：

```json
{
  "cmd": "search",
  "id": "EX2NANO",
  "ranges": [
    {"end_key": "1716521545702", "start_key": "33"},
    {"end_key": "1708621545702", "start_key": "1706421545702"}
  ],
  "ruleid": "rule444444444444444444422222221"
}
```

#### 参数说明

- **cmd**: 搜索命令，固定为 `"search"`。
- **id**: 必须为 `"EX2NANO"`，用于标识搜索命令的来源。
- **ranges**: 搜索范围的数组，数组中的每个元素包含 `start_key` 和 `end_key`，目前为时间戳。
- **ruleid**: 用于云端区分和标识不同的搜索规则。

### 2. 搜索范围

数据的搜索范围包括以下两个部分：

1. **Ringbus 中的数据**
2. **Parquet 文件**

### 3. 搜索结果的上传

搜索到的数据会根据其存储位置不同，通过不同的主题上传到云端。

#### 上传主题格式

- **Ringbus 数据上传主题**: `$file/upload/MQ/{ruleid}/{md5sum}/{filename}`
- **Parquet 文件上传主题**: `$file/upload/parquetfile/{ruleid}/{md5sum}/{filename}`

#### 参数说明

- **ruleid**: 对应搜索命令中的 `ruleid`，用于区分云端依赖。
- **md5sum**: 搜索到的文件的 MD5 校验和，用于文件校验和验证。
- **filename**: 搜索到的文件名。

### 示例

假设我们有以下搜索命令：

```json
{
  "cmd": "search",
  "id": "EX2NANO",
  "ranges": [
    {"end_key": "1716521545702", "start_key": "33"},
    {"end_key": "1708621545702", "start_key": "1706421545702"}
  ],
  "ruleid": "rule444444444444444444422222221"
}
```

在执行搜索后，系统会根据提供的 `ranges` 搜索 Ringbus 和 Parquet 文件中的数据。假设搜索到以下文件：

1. `ringbus_data_file`
2. `parquet_data_file`

这些文件将通过以下主题上传到云端：

- **Ringbus 数据文件上传**:
  ```
  $file/upload/MQ/rule444444444444444444422222221/{md5sum}/ringbus_data_file
  ```

- **Parquet 数据文件上传**:
  ```
  $file/upload/parquetfile/rule444444444444444444422222221/{md5sum}/parquet_data_file
  ```

### 注意事项

1. **命令格式**: 搜索命令必须严格按照指定格式，包括所有必要字段。
2. **时间戳范围**: `ranges` 中的 `start_key` 和 `end_key` 必须为有效的时间戳，以确保搜索范围准确。
3. **上传主题**: 上传主题必须包含 `ruleid`、`md5sum` 和 `filename` 以确保文件在云端的唯一性和可验证性。
4. **数据来源**: 搜索范围包括 Ringbus 和 Parquet 文件，确保数据完整性。