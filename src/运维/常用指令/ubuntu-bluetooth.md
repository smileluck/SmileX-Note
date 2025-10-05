- [ubuntu bluez](#ubuntu-bluez)
  - [**一、安装BlueZ**](#一安装bluez)
  - [**二、基本蓝牙操作（命令行）**](#二基本蓝牙操作命令行)
    - [1. 启动/重启蓝牙服务](#1-启动重启蓝牙服务)
    - [2. 进入蓝牙交互模式](#2-进入蓝牙交互模式)
    - [3. 开启蓝牙适配器](#3-开启蓝牙适配器)
    - [4. 扫描并连接设备](#4-扫描并连接设备)
    - [5. 断开连接与移除设备](#5-断开连接与移除设备)
  - [**三、图形界面操作（适合新手）**](#三图形界面操作适合新手)
  - [**四、高级配置（修改配置文件）**](#四高级配置修改配置文件)
  - [**五、常见问题解决**](#五常见问题解决)


## ubuntu bluez

在Ubuntu系统中配置和使用BlueZ（蓝牙协议栈）主要涉及安装、基本操作和高级配置，以下是详细步骤：


### **一、安装BlueZ**
Ubuntu通常预装了BlueZ，若未安装或需要更新，可执行以下命令：
```bash
# 更新软件源
sudo apt update

# 安装BlueZ及其工具
sudo apt install bluez bluez-tools
```

- `bluez`：核心蓝牙协议栈
- `bluez-tools`：提供`bt-adapter`、`bt-device`等命令行工具


### **二、基本蓝牙操作（命令行）**
#### 1. 启动/重启蓝牙服务
```bash
# 启动蓝牙服务
sudo systemctl start bluetooth

# 设置开机自启
sudo systemctl enable bluetooth

# 重启服务（配置修改后生效）
sudo systemctl restart bluetooth
```

#### 2. 进入蓝牙交互模式
使用`bluetoothctl`工具管理蓝牙设备：
```bash
bluetoothctl
```
进入交互模式后，提示符会变为`[bluetooth]#`。

#### 3. 开启蓝牙适配器
```bash
# 列出所有适配器（通常为hci0）
list

# 选择适配器（替换hci0为实际名称）
select hci0

# 开启蓝牙
power on

# 设为可发现模式（持续120秒）
discoverable on

# 设为可配对模式
pairable on
```

#### 4. 扫描并连接设备
```bash
# 扫描附近蓝牙设备（按Ctrl+C停止）
scan on

# 扫描到设备后，记录其MAC地址（如AA:BB:CC:DD:EE:FF）
# 配对设备
pair AA:BB:CC:DD:EE:FF

# 信任设备（可选，避免重复配对）
trust AA:BB:CC:DD:EE:FF

# 连接设备
connect AA:BB:CC:DD:EE:FF
```

#### 5. 断开连接与移除设备
```bash
# 断开连接
disconnect AA:BB:CC:DD:EE:FF

# 移除配对
remove AA:BB:CC:DD:EE:FF
```


### **三、图形界面操作（适合新手）**
若偏好可视化操作，可使用Ubuntu自带的蓝牙管理器：
1. 点击屏幕右上角的系统菜单 → 蓝牙图标 → **蓝牙设置**
2. 在弹出的窗口中：
   - 开启“蓝牙”开关（对应命令行的`power on`）
   - 开启“可发现”（对应`discoverable on`）
   - 扫描到设备后，点击设备名称 → **配对** → 按提示完成连接


### **四、高级配置（修改配置文件）**
BlueZ的主配置文件为`/etc/bluetooth/main.conf`，可通过修改实现自定义功能：
```bash
# 编辑配置文件
sudo nano /etc/bluetooth/main.conf
```

常用配置项：
- `DiscoverableTimeout = 0`：关闭可发现模式超时（默认120秒）
- `AutoEnable = true`：开机自动启用蓝牙适配器
- `Name = YourDeviceName`：自定义设备名称

修改后需重启服务生效：
```bash
sudo systemctl restart bluetooth
```


### **五、常见问题解决**
1. **蓝牙适配器未识别**：
   - 检查硬件是否启用（部分笔记本有物理开关或Fn快捷键）
   - 执行`lsusb`或`lspci`确认适配器是否被系统检测到

2. **配对失败**：
   - 确保设备处于配对模式（参考设备说明书）
   - 尝试删除设备重新配对：`remove <MAC>` → `pair <MAC>`

3. **音频设备无声音**：
   - 安装脉冲音频蓝牙模块：`sudo apt install pulseaudio-module-bluetooth`
   - 重启脉冲音频：`pulseaudio -k && pulseaudio --start`


通过以上步骤，可完成Ubuntu系统中BlueZ的基本配置和设备管理，满足日常蓝牙设备（如耳机、键盘、音箱）的连接需求。