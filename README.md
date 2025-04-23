# 基于小智AI的ESP32机器狗

## 项目概述

本项目旨在基于 ESP32-S3 芯片打造一款集语音识别、远程通信和图形显示于一体的智能机器狗。该设备结合语音交互、OLED 显示和网络通信功能，具备一定的“AI”属性，能够实现简单的语音指令识别并作出反应，同时通过 MQTT 或 UDP 接收远程指令，具备良好的扩展性与交互性。

## 技术参数

主控芯片 ：ESP32S3
开发框架 ：ESP-IDF 5.4
显示 ： ssd1306 OLED (128x64) i2c 通信
服务通信接口 ：MQTT + UDP

## 项目代码结构

```c
.
├── attachment                  # 存放项目附件资料
├── CMakeLists.txt              # 顶层 CMake 配置文件，定义构建流程
├── components/                 # 自定义组件模块
│   ├── background_task         # 后台任务处理模块（如定时器、系统循环任务）
│   ├── settings                # 系统设置存储与管理（如NVS参数、配置项）
│   └── system_info             # 系统信息获取（如芯片信息、版本信息等）
├── dependencies.lock           # ESP-IDF 的依赖锁定文件，确保组件版本一致性
├── main/                       # 主程序代码入口
│   ├── assets                  # 静态资源（如图像、音效等）
│   ├── fonts                   # 显示用字体资源
│   ├── audio_codecs            # 音频编解码相关逻辑
│   ├── audio_processing        # 音频处理模块
│   ├── boards                  # 板级支持包（如 pin 脚配置、硬件抽象层）
│   ├── protocols               # 协议解析与打包（自定义通信协议处理）
│   ├── iot                     # 物联网操作逻辑
│   ├── led                     # LED 控制模块（如状态指示灯、炫彩灯效）
│   ├── display                 # OLED 显示相关逻辑（如屏幕绘图、UI）
│   ├── dog                     # 机器狗行为逻辑模块（如动作、反馈、响应）
│   ├── application.cc/h        # 应用主逻辑控制入口，初始化与状态管理
│   ├── idf_component.yml       # main 模块的组件元信息定义
│   ├── CMakeLists.txt          # main 目录专用构建配置
│   ├── Kconfig.projbuild       # 项目级配置菜单（Kconfig）扩展
│   └─── main.cc                 # 主函数入口
├── managed_components          # ESP-IDF 管理的第三方组件依赖
├── partitions.csv              # Flash 分区表配置
├── README.md                   # 项目说明文件（你正在看的这个）
├── sdkconfig                   # 当前配置生成文件
├── sdkconfig.defaults          # 默认配置（通用）
└── sdkconfig.defaults.esp32s3  # 针对 ESP32-S3 的默认配置覆盖
```

其中 `main.cc `中的` Application::GetInstance().Start();` 是整个系统的启动入口，承担了系统初始化与主逻辑调度的核心功能。该函数完成了以下几个关键步骤，是系统运行的起点：

`Application::Start() `功能详解如下：

1. **初始化硬件资源**
2. **音频处理准备**
3. **启动音频模块与主循环**
4. **网络和协议初始化**
5. **音频处理和唤醒词模块**（仅启用 CONFIG_USE_AUDIO_PROCESSING）
6. **设置初始状态并启动时钟**

## 硬件部分

##### 主控芯片：ESP32-S3

- 支持 AI 指令集，适用于语音识别与音频处理应用
- 支持 Wi-Fi 和 BLE，满足物联网通信需求
- 丰富的 GPIO，适合多外设控制（如舵机、OLED）
##### 显示屏：SSD1306 OLED（128x64，I2C 通信）

- 显示系统状态、响应信息、可用于 UI 界面展示
- 低功耗、对比度高，适合嵌入式小设备
##### 数字麦克风：INMP441（I2S 接口）

- 高灵敏度数字麦克风，适用于本地语音采集
- I2S 接口可直接接入 ESP32-S3 用于语音识别处理
##### 音频输出：MAX98357A I2S 数字功放 + 腔体喇叭

- 将语音合成/提示音输出至扬声器，实现语音反馈
- I2S 接口支持直接播放 PCM 数据
##### 动作执行器：SG90 舵机 x 4

- 控制狗狗的“头部”与“腿部”动作
- 通过 PWM 控制，支持多动作组合执行

硬件效果如下图：

![651](attachments/Pasted%20image%2020250423183709.png)

## 使用

##### 1. **编译并烧录 ESP32S3 固件**

 构建项目并将其烧录到板子上，然后运行监控工具以查看串行输出：
- 运行 `idf.py set-target esp32s3` 以设置目标芯片
- 运行` idf.py -p PORT flash `来构建并烧录项目
##### 2. 为开发板配置WiFi网络

  - 连接开发板的热点并等待跳转至后台网页，填写待连接的网络信息
  - 网络配置成功后开发板重启、成功后效果如下：

![582](attachments/Pasted%20image%2020250423184648.png)
##### 3. 使用控制指令控制小狗执行动作

| 指令英文（Method Name） | 中文动作说明       | 动作状态常量（SetActionState 参数） |
|--------------------------|--------------------|-------------------------------------|
| `Walk`                   | 前进（forward）     | `kActionStateWalk`                  |
| `Walk back`              | 后退（back）        | `kActionStateWalkBack`              |
| `sitdown`                | 坐下（sit down）    | `kActionStateSitdown`               |
| `stand`                  | 站立（stand up）    | `kActionStateStand`                 |
| `sleep`                  | 睡觉（sleep）       | `kActionStateSleep`                 |
| `turn left`              | 左转（left）        | `kActionStateTurnLeft`              |
| `turn right`             | 右转（right）       | `kActionStateTurnRight`             |
| `wave`                   | 挥手（挥挥手）      | `kActionStateWave`                  |
| `stop`                   | 停止（stop）        | `kActionStateStop`                  |

## 参考：

9. https://github.com/78/xiaozhi-esp32
10. https://ccnphfhqs21z.feishu.cn/wiki/F5krwD16viZoF0kKkvDcrZNYnhb
