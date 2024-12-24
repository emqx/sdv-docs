# 可灵活配置的流模组，实现数据灵活落盘

## 1. 背景

### 1.1 Parquet 文件特点

Parquet 是一种列式存储文件格式，由 Apache 开发。它广泛应用于大数据处理和分析领域，特别是在 Apache Hadoop 和 Apache Spark 生态系统中。Parquet 文件具有以下特点：

1. **列式存储**：Parquet 文件将数据按列而非行进行存储。这种存储方式使得查询性能得到显著提升，因为只需读取必要的列而不是整行数据。特别适合于读取某些列的数据时。

2. **压缩效率高**：由于列式存储的特性，Parquet 文件在压缩方面表现优异。相同类型的数据存储在一起，有助于提高压缩比，从而减少存储空间。

3. **支持嵌套数据**：Parquet 文件可以处理复杂的嵌套数据结构，如嵌套的记录、数组和映射。这使得它在处理复杂数据模式时非常灵活。

4. **模式演进**：Parquet 支持模式演进，允许在数据文件中更改列的类型或添加新列，而不会破坏现有的数据。这对于数据架构的变化非常有用。

5. **高效的读写**：Parquet 文件采用了分块存储的方式，使得在大数据集中的随机访问和批量读取变得高效。

6. **兼容性强**：Parquet 文件格式被广泛支持，包括在 Apache Hive、Apache Impala、Apache Drill、Apache Spark、Presto 等数据处理工具中。

### 1.2 Parquet 文件存储格式

Parquet 文件的存储格式设计旨在优化数据存储和读取性能。其主要特点包括：

1. **文件结构**：
   - **文件头**：包括 magic number 和文件的元数据。
   - **Row Groups**：文件由多个 Row Groups 组成。每个 Row Group 包含一个或多个数据列的块，并且 Row Group 是 Parquet 文件中的存储单元。
   - **Column Chunks**：Row Group 内的每一列数据都会被分成多个 Column Chunks。Column Chunk 是实际存储数据的地方，支持并行读写。
   - **Page**：每个 Column Chunk 内的数据被组织为多个 Page。Page 分为数据页、字典页、索引页等不同类型，用于优化数据存储和访问。

2. **压缩和编码**：
   - **压缩算法**：Parquet 支持多种压缩算法，如 Snappy、Gzip 和 LZO，用户可以根据需要选择合适的算法来减少文件大小。
   - **编码方式**：Parquet 使用了多种编码技术（如字典编码、游程编码）来进一步提高数据压缩率和读取效率。

3. **数据类型和模式**：
   - **强类型**：每个列都有明确的数据类型，支持整数、浮点数、字符串、日期等多种数据类型。
   - **复杂数据类型**：支持嵌套的数据结构，如结构体（Struct）、数组（Array）和映射（Map），能够表示复杂的层次结构。

4. **读取优化**：
   - **列裁剪**：在读取数据时，只需读取感兴趣的列，从而减少不必要的数据读取。
   - **行组分割**：数据按 Row Group 存储，支持对大数据集进行分块处理，优化数据访问和读取性能。

由于 Parquet 文件的这些特性，如何将数据合理地存储到 Parquet 文件中是一个关键问题。合理的数据存储方式能够显著提高数据处理的性能，并减少文件的存储空间。例如，通过优化 Row Group 的大小和选择合适的压缩算法，可以在保持高性能的同时减少存储开销。



## 2. 可配置流管理系统

随着大数据处理需求的不断增加，各种数据流的格式和存储需求变得越来越复杂。为了高效地管理和优化这些数据流，流管理系统应运而生。流管理系统通过为不同类型的数据流提供专门的处理和转换机制，从而实现对数据的高效处理和存储。特别是在处理大规模数据和高并发数据时，流管理系统的作用尤为重要。它不仅能够确保数据的高效存储和读取，还能够在数据存储到 Parquet 文件中前和从 Parquet 文件中读取后，对数据进行相应的编码和解码，从而提升系统的整体性能。

### 2.1 流管理系统初始化

在系统启动时，流管理系统会进行必要的初始化操作。这一过程包括加载和配置流模块，以及准备处理数据流所需的各种资源。初始化过程中，系统会自动加载原始流 (raw_stream) 模块。原始流模块负责将数据流以基础格式存储到 Parquet 文件中。这个模块的设计重点在于确保数据的基本格式正确存储，并不会进行额外的性能优化。因此，它适用于数据流格式简单且对性能要求不是特别高的场景。

### 2.2 原始流(raw_stream)模块

原始流模块的设计简单直观，其主要任务是将数据流按照基础格式存储到 Parquet 文件中。数据被存储为时间戳和数据体的形式，其中时间戳用于标记数据的时间点，数据体则是实际的数据内容。以下是原始流模块的数据存储格式示例：

```
|    ts(uint64_t)   |        data(data_len)        |
|       ts1         |             data1            |
|       ts2         |             data2            |
|       ...         |             ...              |
|       tsn         |             datan            |

```

这种存储方式的优点在于实现简单，但对于需要高性能或复杂数据处理的场景可能不够高效。

### 2.3 插件形式的流模块

为了满足不同场景下的性能需求，流管理系统允许用户使用自定义插件形式的流模块。这些插件模块可以对数据流进行编码和解码，从而优化性能和减少内存占用。用户可以根据实际需求实现特定的编码解码逻辑，以提高数据处理效率。 

以下是一个插件形式的流模块必须实现的接口示例：

```C
#include "include/plugin.h"
#include "nng/exchange/stream/stream.h"
#define USER_STREAM_NAME "user"
#define USER_STREAM_ID   0x2

void *user_encode(void *data)
{
	struct stream_data_out *output_stream = NULL;
	...
	return output_stream;
}
void *user_decode(struct parquet_data_ret *parquet_data)
{
    struct stream_decoded_data *decoded_data = NULL;
    ...
    return decoded_data;
}

void *user_cmd_parser(void *data)
{
    struct cmd_data *cmd_data = NULL;
    ...
    return cmd_data;
}

int nano_plugin_init()
{
	int ret = 0;
	char *name = NULL;

	name = (char *)malloc(strlen(USER_STREAM_NAME) + 1); 
	if (name == NULL) {
		return -1; 
	}

	strcpy(name, USER_STREAM_NAME);

	ret = stream_register(name, USER_STREAM_ID, user_decode, user_encode, user_cmd_parser);
	if (ret != 0) {
		log_error("stream_register %s failed", name);
		free(name);

		return -1; 
	}

	return ret;
}
```

结构体定义示例：

```C
struct stream_data_out {
    uint32_t col_len;
    uint32_t row_len;
    uint64_t *ts;
    char **schema;
    parquet_data_packet ***payload_arr;
};

struct stream_data_in {
    void **datas;
    uint64_t *keys;
    uint32_t *lens;
    uint32_t len;
};

struct stream_decoded_data {
    void *data;
    uint32_t len;
};

struct cmd_data {
    bool is_sync;
    uint64_t start_key;
    uint64_t end_key;
    uint32_t schema_len;
    char **schema;
};
```

## 3. 流系统是如何工作的

流系统的工作主要涉及两个方面：

- 数据在落盘前的编码

- 数据在落盘后的解码。

	这两个过程分别在数据存储到 Parquet 文件之前和从 Parquet 文件中读取之后进行。

### 3.1 落盘前对数据进行encode

在数据被存储到 Parquet 文件之前，流管理系统会先对数据进行编码。编码过程包括：

1. **获取数据流**：从数据源中获取需要处理的数据流。
2. **编码数据流**：调用流模块中的编码函数对数据进行编码，将数据转换成适合存储的格式。
3. **流系统处理**：流系统接收编码后的数据，并进行必要的处理和转发。
4. **存储数据**：将编码后的数据存储到 Parquet 文件中。

数据流编码流程示意图：

```
                         4.encoded data
  +------------------------------------------+
  |                                          v
+----------------------+  3.StreamType     +-----------------------+
| Stream Module encode | <---------------- |     Stream system     | -+
+----------------------+                   +-----------------------+  |
                                             ^                        |
                                             | 2.StreamType           | 5.encoded data
                                             |                        |
+----------------------+  1.FULL_RETURN    +-----------------------+  |
|       Ringbus        | ----------------> |        Webhook        | <+
+----------------------+                   +-----------------------+
                                             |
                                             | 6.flush encoded data
                                             v
                                           +-----------------------+
                                           |        Parquet        |
                                           +-----------------------+
```



### 3.2 落盘后对数据进行decode

在从 Parquet 文件中读取数据后，流管理系统会对数据进行解码。解码过程包括：

1. **查询数据**：通过查询命令从 Parquet 文件中查找数据。
2. **查询命令解析**：调用流模块中的命令解析函数对查询命令进行解析，返回解析后的查询命令。
3. **数据提取**：从 Parquet 文件中提取数据。
4. **解码数据流**：调用流模块中的解码函数对数据进行解码，将数据转换回原始格式。
5. **数据返回**：将解码后的数据返回给查询者或进一步处理。

数据流解码流程示意图：

```
                                +-------------------------+
                                |         Parquet         | -+
                                +-------------------------+  |
                                  ^                          |                         5.parsed command
                                  | 6.find data in paruqet   | 7.data in parquet   +-------------------------+
                                  |                          v                     v                         |
+----------+  1.query command   +----------------------------------------------------+  8.StreamType       +------------------------------------------------+  9.StreamType      +----------------------+
| consumer | -----------------> |                                                    | ------------------> |                                                | -----------------> | Stream Module decode |
+----------+                    |                                                    |                     |                                                |                    +----------------------+
  ^          12.decoded data    |                                                    |  2.parse command    |                                                |  10.decoded data     |
  +---------------------------- |                  exchange_server                   | ------------------> |                 Stream system                  | <--------------------+
                                |                                                    |                     |                                                |
                                |                                                    |  11.decoded data    |                                                |
                                |                                                    | <------------------ |                                                |
                                +----------------------------------------------------+                     +------------------------------------------------+
                                                                                                             |                           ^
                                                                                                             | 3.StreamType              | 4. parsed command
                                                                                                             v                           |
                                                                                                           +--------------------------+  |
                                                                                                           | Stream Module cmd parser | -+
                                                                                                           +--------------------------+
```

