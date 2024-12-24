# 软开关

在整个 sdv-flow 的生命周期内关于软开关的操作如下：
## 软开关主流程
```mermaid
flowchart TB 
    A["Start Sdvflow"] --> B["Lots of operations......"]
    B --> Q{"Data Switch Enabled?"}
    Q -- Yes --> R["Load and Save Data Switch Property"]
    Q -- No --> T
    R --> T["Lots of operations......"]
    T --> AG{"Data Switch Enabled?"}
    AG -- Yes --> AH["Manage Data Switch"]
    AG -- No --> AI["Lots of operations......"]
    AH --> AI
    AI --> AJ{"Data Switch Enabled?"}
    AJ --Yes--> AK["Run DataSwitchController"]
    AJ --No--> AP
    AK --> AP["Wait for Signal Handling"]
    AP --> AQ["Capture signal"]
    AQ --> AR["Save dataSwitch status to file"]
    AR --> AS["Exit Successfully"] 
```

### 流程描述
- 启动 Sdvflow
    - 启动程序并执行初始设置和预处理步骤。
- 执行大量初始化操作
    - 包括日志系统配置、插件管理器加载、全局环境变量设置、HTTP 服务启动等。
- 检查数据开关是否启用
    - 如果启用：
        - 加载并保存数据开关属性。
    - 如果未启用：
        - 跳过数据开关相关的操作。
- 继续执行剩余初始化操作
        - 包括插件的初始化逻辑、子进程（如 eKuiper、NanoMQ）的触发和启动。
- 再次检查数据开关是否启用
    - 如果启用：
        - 管理数据开关状态。
    - 如果未启用：
        - 跳过数据开关管理。
- 处理数据开关控制逻辑
    - 再次检查数据开关是否启用：
        - 如果启用，运行 DataSwitchController 以处理数据开关的动态控制。
        - 如果未启用，跳过该步骤。
- 等待信号处理
    - 主线程进入等待状态，监听外部信号（如 SIGTERM、SIGINT 等）。
- 捕获退出信号
    - 当收到退出信号时，执行清理操作。
- 保存数据开关状态到文件
    - 将当前数据开关的状态保存到持久化文件中，确保下次启动时状态一致。
- 成功退出
    - 记录日志并安全退出程序。

## 软开关上传判断条件
```mermaid
flowchart TD
    A["Start"] --> B["Log vehicle info"]
    B --> C{"Is VIN valid?"}
    C -- No --> D["Return false, error: invalid VIN"]
    C -- Yes --> E{"Check car type (CS)"}
    E -- CS == E245 --> F{"Check market code (NC)"}
    E -- CS != E245 --> G["Return false, error: unknown car configuration"]
    F -- NC in [42, 43, A8, A7, A6] --> H{"Check vehicle mode"}
    F -- Other NC --> I["Return false, error: unknown NC"]
    H -- VehicleMode in [Normal, Crash, Dyno] --> J{"Check power mode"}
    H -- Other VehicleMode --> K["Return false, error: unknown vehicle mode"]
    J -- PowerMode != abandoned --> L["Return DataSwitch, nil"]
    J -- PowerMode == abandoned --> M["Return false, error: unknown power mode"]
    D --> L
    G --> L
    I --> L
    K --> L
    M --> L
```
## 流程图描述：
- 开始
    - 从函数 ShouldUploadData() 开始执行。
- 日志打印信息
    - 打印车辆相关信息：nc、cs、vehicleMode 和 powerMode。
- 检查 VIN 是否有效
    - 如果 VIN 是无效的INVALIDVIN(00000000000000000)，返回 false 和错误信息 "invalid VIN"。
    - 如果 VIN 是有效的，继续执行下一步。
- 检查车辆类型 (CS)
    - 如果 CS 是 E245(E4)，则进入下一步。
    - 如果 CS 不是 E245，返回 false 和错误信息 "unknown car configuration"。
- 检查市场代码 (NC)
    - 如果 NC 是 42、43、A8、A7 或 A6（表示特定的市场，如澳大利亚、新西兰、泰国、印度尼西亚和马来西亚），进入下一步。
    - 如果 NC 不是这几个值，返回 false 和错误信息 "unknown NC"。
- 检查车辆模式 (VehicleMode)
    - 如果 VehicleMode 是 VehModNorm（正常模式）、VehModCrash（碰撞模式）或 VehModDyno（台架模式），进入下一步。
    - 如果 process.VehicleMode 不是这些值，返回 false 和错误信息 "unknown vehicle mode"。
- 检查电源模式 (PowerMode)
    - 如果 PowerMode 不是 PowerModAbdnd（废弃模式），则返回 DataSwitch 的值和 nil（表示数据上传可用）。
    - 如果 PowerMode 是 PowerModAbdnd，返回 false 和错误信息 "unknown power mode"。

## 加载并存储软开关

```mermaid
flowchart TD
    A["Start"] --> B{"Is EnableDataSwitch enabled?"}
    B -- Yes --> C["Log: sdv-flow data switch is enabled"]
    C --> F["Get Android Property:  persist.sys.dataSwitch"]
    F --> G{"Is property empty?"}
    G -- Yes --> H{"Check if file dataSwitch Not exists"}
    H -- No --> I["Read property from file"]
    I --> M{"Set DataSwitch"}
    H -- Yes --> J@{ label: "Set default property: 'off'" }
    J --> K@{ label: "Create file and write 'off'" }
    K --> O["DataSwitch = false"]
    L["Save property to file dataSwitch"] --> M
    M -- on --> N["DataSwitch = true"]
    M -- off --> O
    M -- Invalid --> P["Error: Invalid property"]
    P --> Q["Return error"]
    N --> R["Return nil"]
    O --> R
    G -- No --> L
    N --> n1["Untitled Node"]

    J@{ shape: rect}
    K@{ shape: rect}

```
### 流程描述
1. EnableDataSwitch 检查
- 条件：如果 EnableDataSwitch 为 true，则继续执行。
- 操作：打印日志 "sdv-flow data switch is enabled"。
2. 加载并保存 persist.sys.dataSwitch 属性
- 操作：调用 agent.LoadAndSaveProperty("persist.sys.dataSwitch")。
- 成功：继续执行。
- 失败：如果 err != nil，则打印错误日志："LoadAndSaveProperty: 错误信息"。
3. LoadAndSaveProperty 函数处理
- 输入：传入 key（persist.sys.dataSwitch）到 LoadAndSaveProperty。
- 操作：
    - 获取 Android 属性：调用 getAndroidProperty(key) 获取属性。
    - 如果成功，打印获取到的属性值。
    - 如果失败，打印错误信息。
4. 检查属性是否为空
- 条件：如果属性为空，执行以下操作：
    - 文件检查：检查 filePath 指定的文件是否存在。
        - 如果文件不存在：
            - 设置默认值 "off"。
            - 创建文件并写入 "off"。
            - 打印日志："Set default property off." 和 "Write property to file."。
        - 如果文件存在：
            - 读取文件内容。
            - 打印日志："Read property from file."。
5. 写入属性
- 操作：如果属性不为空，将其保存到文件中，并打印日志："Save the retrieved property to the file"。
6. 设置 DataSwitch
- 条件：根据属性值设置 DataSwitch：
    - 如果值为 "on"，设置 DataSwitch = true。
    - 如果值为 "off"，设置 DataSwitch = false。
    - 如果属性值无效，打印错误并返回。

7. 返回成功或失败
- 如果设置成功，返回 nil。
- 如果出现错误，返回相应的错误。
