# 历史采集最佳实践

## 预准备

- 准备2台服务器，安装 sdv-low，sdv-platform ，以下sdv-low的vin码用vin1指带。
- 准备信号定义文件dbc.json，放在software/ekuiper/data/uploads/dbc 下。

- 云端创建历史采集长期运行的2个流和1个触发规则

  - POST /api/edgeservices/config

    - ```
      {
          "ids": ["vin1"],
          "method": "POST",
          "path": "/streams",
          "body": "{\"sql\": \"CREATE STREAM triggerStream () WITH (TYPE=\\\"mqtt\\\", FORMAT=\\\"json\\\", DATASOURCE=\\\"trigger\\\", SHARED=\\\"TRUE\\\")\"}"}
      ```

  - POST /api/edgeservices/config

    - ```
      {
          "ids": ["vin1"],
          "method": "POST",
          "path": "/rules",
          "body": "{\"id\":\"ruleTrigger\",\"sql\":\"SELECT \\\"async\\\" as kind, ts1, ts2 FROM triggerStream\",\"actions\":[{\"nanoquery\":{\"url\":\"tcp://127.0.0.1:10000\",\"format\": \"delimited\",\"delimiter\":\"-\",\"fields\": [\"kind\",\"ts1\",\"ts2\"],\"sendSingle\":true}}]}"}
      ```

  - POST /api/edgeservices/config

    - ```
      {
          "ids": ["vin1"],
          "method": "POST",
          "path": "/streams",
          "body": "{\"sql\": \"CREATE STREAM queryStream () WITH (TYPE=\\\"nanoquery\\\", FORMAT=\\\"spi\\\", SCHEMAID=\\\"data/uploads/dbc/gee.json\\\", SHARED=\\\"TRUE\\\")\"}"}
      ```

## 云端下发历史采集

1. 调platform api，获取当前历史数据可用时间范围

   ```
   GET /api/edgeservice/proxy/message_bus/${vin}/can_data_span
   ```

   返回结果

   ```json
   {
       "code": 0,
       "data": {
           "begin": 1722411442963,
           "end": 1722412705492
       }
   }
   ```

2. 下发历史查询算法

   ```
   POST /api/edgeservice/proxy/history/${vin}
   
   {
       "id": "history_rule"
       "sql": "SELECT ts, `PropulsionCAN.0x342.HvCellUOverFlt`,`PropulsionCAN.0x346.HvBattCooltLvl` FROM queryStream",
       "actions": [{"file":{"path": "data/history_rule1_1111.zstd","format":"delimited","fileType": "csv", "hasHeader":true,"sendSingle":true,"compression":"zstd"}}],
       "options": "{"sendError":false}",
       "start": 1723017850443,
       "end": 1723018779225,
       "extras": {"xids":"13421346"}
   }
   ```

   - id  规则id，确保用在不同的历史采集任务中的规则 id 都是不重复的。
   - sql 中，select 后替换为实际用户选择的信号（用逗号分隔）
   - actions 规则算法的actions，上面示例中action表示采集结果存储在 data/history_rule1_1111.zstd
   - options 规则算法的options
   - start 和 end 分别为需要查询的起始、结束时间戳（毫秒为单位），参考上一步的返回结果
   - extras 字典包含用户选择的信号的 busId 拼接的结果，使用 `xids` 为键，值的拼接方式为：**每个 busId 4位原始16进制的拼接在一起**。例如，上面示例中选择了 2 个信号，PropulsionCAN.0x342.HvCellUOverFlt 的 十六进制 busId 为 0x1342，PropulsionCAN.0x346.HvBattCooltLvl 的 十六进制 busId 为 0x1346，则拼接后的值为 13421346，或者 13461342，信号拼接的顺序不作要求。（busId 的真实值根据客户提供的映射表来确定）
   - 下发成功，返回 204 code

3. 根据下发的内容，按时间范围查询到的信号数据将会存储在action中指定的路径下或者传输到指定的mqtt topic中。历史查询算法查询结束后，算法的状态会自动变成stopped，可以通过/api/metric/vin获取算法状态及相关数据（如查询到了多少条数据）。查询完成后建议适时删除已完成的历史查询算法，以节省资源。