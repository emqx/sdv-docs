# 软开关交互

1. sdv-flow 启动去获取安卓 property：获取不到就读自己记录下来的状态。软件在车上第一次启动前记录的状态为 off。
2. 当用户操作改变状态，app 调jar 包 set  ,通过MQTT 同步状态给大数据；同时修改property
3. sdv-flow 每次收到 set 同步返回 callback 确认收到通知；并记录更新开关状态。

## jar包

public String set(String cmd)

- 调用方式 

```
SoftSwitchLib lib = new SoftSwitchLib(); String result = lib.set("on");
```

- `new SoftSwitchLib()一次或多次均可`，效果一样
- cmd 值可以为on或者off
- result： ok,timeout,invalid_param 或者其他错误原因
  - 若2s后sdv-flow没有响应，则返回timeout
  - 若cmd参数非on或者off，则返回 invalid_param
  - 如果命令下发成功，返回ok

## sdv-flow

读写安卓property

- key：dataSwitch
- value：on或者off