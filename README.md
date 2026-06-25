# 智能应急灯嵌入式系统

基于ESP32的智能应急灯嵌入式系统，集成了多种传感器、LED矩阵显示、MQTT通信和Mesh网络功能。

## 功能特性

### 核心功能
- **LED矩阵显示**：支持HUB75接口的LED矩阵，可显示动态/静态箭头、图标等多种指示图案
- **多传感器集成**：ENS160空气质量传感器、DHT20温湿度传感器
- **MQTT通信**：通过Cat-WH-LTE-7S0 4G模块实现远程数据传输和控制
- **ESP32 Mesh网络**：支持多设备组网，实现分布式应急指示
- **报警系统**：蜂鸣器报警、火焰检测报警、电流检测报警
- **亮度调节**：支持PWM调光控制

### 显示模式
- 动态/静态方向指示（左、右、上、下箭头）
- 多种图标显示（禁止、卫生间等）
- 支持RGB颜色自定义
- 支持亮度调节

### 通信功能
- MQTT协议数据上传
- Mesh网络设备间通信
- 远程控制指令接收
- 实时状态反馈

## 硬件要求

### 主控板
- ESP32开发板（FeatherESP32）

### 传感器
- ENS160空气质量传感器（I2C接口）
- DHT20温湿度传感器（I2C接口）
- 火焰传感器
- 电流检测传感器

### 显示设备
- HUB75接口LED矩阵屏（32x32像素，3个面板级联）

### 通信模块
- Cat-WH-LTE-7S0 4G通信模块

### 其他组件
- 蜂鸣器
- PWM调光电路（场效应管）
- 电源检测电路

## 软件依赖

### PlatformIO库
```ini
robtillaart/DHT20@^0.3.0
sstaub/Ticker@^4.4.0
erropix/ESP32 AnalogWrite@^0.2
painlessmesh/painlessMesh@^1.5.0
bblanchon/ArduinoJson@^7.0.4
```

### 本地库
- Adafruit-GFX-Library
- Adafruit_BusIO
- Adafruit_Circuit_Playground
- FastLED
- RGB-matrix-Panel
- eztime

## 项目结构

```
智能应急灯嵌入式源码/
├── src/
│   ├── main.cpp                    # 主程序
│   ├── ESP32_Mesh.h               # Mesh网络配置
│   ├── Cat-WH-LTE-7S0.h          # 4G模块通信
│   ├── ENS160.h/cpp               # 空气质量传感器驱动
│   ├── DHT20.h                    # 温湿度传感器驱动
│   ├── ESP32-HUB75-MatrixPanel-I2S-DMA.h/cpp  # LED矩阵驱动
│   ├── Adafruit_GFX.h/cpp         # 图形库
│   └── ...                        # 其他驱动文件
├── include/
├── lib/
├── platformio.ini                  # PlatformIO配置文件
└── .gitignore
```

## 快速开始

### 1. 环境准备
安装 [PlatformIO IDE](https://platformio.org/install/ide?install=vscode) 或 PlatformIO CLI。

### 2. 克隆项目
```bash
git clone https://github.com/your-username/smart-emergency-light.git
cd smart-emergency-light
```

### 3. 配置修改
修改 `src/ESP32_Mesh.h` 中的网络配置：
```cpp
#define MESH_PREFIX "your_mesh_name"      # Mesh网络名称
#define MESH_PASSWORD "your_password"     # Mesh网络密码
```

修改 `src/Cat-WH-LTE-7S0.h` 中的MQTT服务器配置：
```cpp
Setting_MQTT_server_parameters("your_mqtt_server_ip");  # MQTT服务器地址
```

### 4. 编译上传
```bash
# 编译项目
pio run

# 上传到ESP32
pio run --target upload

# 监控串口输出
pio device monitor
```

## 配置说明

### 引脚定义
- **PWM引脚**：GPIO 33（LED亮度控制）
- **蜂鸣器引脚**：GPIO 16
- **电流检测引脚**：GPIO 36
- **I2C引脚**：GPIO 21（SDA）、GPIO 22（SCL）
- **串口2**：GPIO 35（RX）、GPIO 32（TX）

### MQTT主题
- 发布主题：`ESP32/{MAC地址}/status`
- 订阅主题：`ESP32/{MAC地址}/public`

### 数据格式
传感器数据以JSON格式上传：
```json
{
  "ESP32_XX:XX:XX:XX:XX:XX": {
    "AQI": 1,
    "TVOC": 100,
    "ECO2": 450,
    "temp": 25.5,
    "hum": 50.0,
    "electricity_Current": true,
    "electricity": 500,
    "ESP32_fires_flag": 0,
    "ESP32_electricity_flag": 0,
    "buzzer": 1
  }
}
```

## 使用说明

### 显示控制
通过MQTT发送控制指令：
```json
{
  "ESP32_XX:XX:XX:XX:XX:XX": {
    "display_left": "Left",      // 左箭头（大写动态，小写静态）
    "display_mid": "motifs",     // 图标
    "display_right": "Right",    // 右箭头
    "RGB_left": [255, 0, 0],    // 左面板颜色
    "RGB_mid": [0, 255, 0],     // 中面板颜色
    "RGB_right": [0, 0, 255],   // 右面板颜色
    "brightness": 50,            // 亮度（0-100）
    "led": 500,                  // LED灯条亮度
    "buzzer": 1,                 // 蜂鸣器（1关/2开）
    "flame": "False",            // 火焰报警
    "electricity_switch": "ON"   // 电流检测开关
  }
}
```

### 显示模式说明
- `Left`/`Right`/`Up`/`Down`：动态箭头
- `left`/`right`/`up`/`down`：静态箭头
- `motifs`：图标
- `style`：样式
- `ban`：禁止图标
- `toilet`：卫生间图标

## 故障排查

### 常见问题

1. **传感器无法初始化**
   - 检查I2C接线是否正确
   - 确认传感器供电正常

2. **MQTT连接失败**
   - 检查4G模块信号
   - 确认MQTT服务器地址和端口正确

3. **LED矩阵不显示**
   - 检查HUB75接口连接
   - 确认电源供电充足

4. **Mesh网络无法组网**
   - 检查WiFi配置
   - 确认Mesh密码一致

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 作者

杨晓涛 - 嘉应学院计算机学院物联网工程专业

## 致谢

- ESP32 Arduino框架
- painlessMesh库
- Adafruit GFX库
- PlatformIO社区