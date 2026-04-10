# ros2-system-status-pubsub-display

一个基于 ROS 2 的系统状态监控示例项目。

该项目通过自定义消息发布主机的系统状态信息，包括：

- 主机名
- CPU 使用率
- 内存使用率
- 内存总量
- 可用内存
- 网络发送量
- 网络接收量

随后使用一个基于 Qt 的 C++ 图形界面节点订阅这些消息，并实时显示系统状态。

---

## 项目特点

- 使用 **ROS 2 topic 通信**
- 使用 **自定义消息类型**
- 使用 **Python 节点**采集系统状态
- 使用 **C++ + Qt** 实现可视化显示
- 适合练习：
  - ROS 2 自定义 `msg`
  - ROS 2 Python 发布者节点
  - ROS 2 C++ 订阅者节点
  - Qt 与 ROS 2 的简单结合

---

## 项目结构

```text
ros2-system-status-pubsub-display/
├── README.md
└── src/
    ├── status_interfaces/
    │   ├── CMakeLists.txt
    │   ├── package.xml
    │   └── msg/
    │       └── SystemStatus.msg
    ├── status_publisher/
    │   ├── package.xml
    │   ├── setup.py
    │   └── status_publisher/
    │       ├── __init__.py
    │       └── sys_status_pub.py
    └── status_display/
        ├── CMakeLists.txt
        ├── package.xml
        ├── LICENSE
        └── src/
            ├── hello_qt.cpp
            └── sys_status_display.cpp
````

---

## 消息定义

`status_interfaces/msg/SystemStatus.msg`

```msg
builtin_interfaces/Time stamp
string host_name
float32 cpu_percent
float32 memory_percent
float32 memory_total
float32 memory_available
float64 net_sent
float64 net_recv
```

字段说明：

* `stamp`：消息时间戳
* `host_name`：主机名称
* `cpu_percent`：CPU 使用率
* `memory_percent`：内存使用率
* `memory_total`：总内存，单位 MB
* `memory_available`：可用内存，单位 MB
* `net_sent`：累计发送流量，单位 MB
* `net_recv`：累计接收流量，单位 MB

---

## 工作流程

项目运行流程如下：

```text
系统状态采集（Python）
        ↓
封装为 SystemStatus 消息
        ↓
发布到话题 /sys_status
        ↓
C++ 订阅者接收消息
        ↓
Qt 界面实时显示数据
```

---

## 依赖环境

建议准备以下环境：

* ROS 2
* colcon
* Python 3
* Qt5 Widgets
* Python 库 `psutil`

---

## 依赖安装

### 1. 安装 Qt

```bash
sudo apt update
sudo apt install qtbase5-dev
```

### 2. 安装 Python 依赖

```bash
pip install psutil
```

---

## 编译方法

在 ROS 2 工作空间根目录下执行：

```bash
colcon build --packages-select status_interfaces status_publisher status_display
```

编译完成后：

```bash
source install/setup.bash
```

---

## 运行方法

建议开两个终端。

### 终端 1：启动系统状态发布节点

```bash
source install/setup.bash
ros2 run status_publisher sys_status_pub
```

该节点会周期性采集系统状态，并发布到：

```text
/sys_status
```

### 终端 2：启动图形显示节点

```bash
source install/setup.bash
ros2 run status_display sys_status_display
```

运行后会弹出 Qt 窗口，实时显示收到的系统状态数据。

---

## Qt 测试程序

仓库里还包含一个简单的 Qt 测试程序：

```bash
source install/setup.bash
ros2 run status_display hello_qt
```

这个程序可以用来确认 Qt 环境是否配置正常。

---

## 调试方法

你也可以直接查看话题内容：

```bash
source install/setup.bash
ros2 topic echo /sys_status
```

这样可以先确认发布节点是否正常工作，再检查显示节点。

---

## 当前实现说明

### `status_publisher`

该包使用 Python 编写，主要逻辑包括：

* 使用 `psutil` 获取 CPU、内存、网络信息
* 使用 `platform.node()` 获取主机名
* 构造 `SystemStatus` 消息
* 通过 ROS 2 topic 周期性发布

### `status_display`

该包使用 C++ 编写，主要逻辑包括：

* 创建 ROS 2 订阅者
* 订阅 `/sys_status`
* 将收到的消息拼接成文本
* 通过 Qt `QLabel` 显示在窗口中

---

## 可以继续改进的方向

这个项目已经完成了基础的发布-订阅-显示流程，后续还可以继续扩展：

* 把 `QLabel` 改成更完整的 GUI 布局
* 增加 CPU / 内存曲线图
* 增加磁盘使用率监控
* 增加刷新频率配置参数
* 支持多主机状态监控
* 使用 `rqt` 或 `QTableWidget` 做更规范的界面展示

---

## License

Apache-2.0
