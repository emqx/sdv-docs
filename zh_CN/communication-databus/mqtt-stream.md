# mqtt stream
## 1. 背景
对于同一topic的MQTT消息，可以看做一条数据流，并且这个数据流是可以进行落盘存储以及查询操作的，对于一些网络较差的环境下，为数据的完整性和可靠性提供了解决方案.

## 2. 内部数据通路
数据流入
1. 当消息进入nanomq的时候，会先将消息打上一个时间戳作为msg的key。
2. 然后消息会进入exchange_client。
3. 然后exchange_client把消息传入exchange中。
4. exchange如果发现消息是指定topic时，会将消息放入ringbus中。
5. 如果ringbus满了则会执行ringbus对应的fullOp，比如将消息落盘。

数据流出
1. nanomq收到触发搜索上传的请求
2. nanomq去ringbus中检索数据，将搜索到的数据上传到MQfile topic
3. nanomq检索parquet，将范围内命中的parquet文件上传到ParquetFile topic

## 3. mqtt-stream与文件上传
mqtt-stream中会对外提供一个数据查询/上传接口，可以通过这个接口触发搜索mqtt-stream中的数据
- Ringbus中的数据：由于Ringbus中的数据还未进行落盘成为文件，所以nanomq搜索出来后自行组装成mqtt消息发送到本地nanomq的1883端口
- Parquet文件：nanomq将命中的parquet组装成文件传输请求发送给文件传输客户端

Ringbus中的数据会直接上传至nanomq的1883端口，parquet文件则会先发送文件传输请求给文件传输客户端，文件传输客户端收到请求后会上传文件到nanomq的1883端口
