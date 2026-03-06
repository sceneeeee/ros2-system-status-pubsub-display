# ros2-system-status-pubsub-display

一个基于 ROS 2 的系统状态发布与可视化显示小项目。  
项目通过 **发布-订阅（Pub/Sub）** 的方式采集并展示主机系统状态：

- 时间戳
- 主机名
- CPU 使用率
- 内存使用率
- 内存总量
- 可用内存
- 网络发送量
- 网络接收量

其中：

- `status_publisher`：负责采集系统状态并发布消息
- `status_interfaces`：负责定义自定义消息 `SystemStatus`
- `status_display`：负责订阅消息并使用 Qt 进行界面显示

---

## 项目效果

运行后，发布节点会持续获取系统状态并发布到 `sys_status` 话题；显示节点订阅该话题后，会通过 Qt 窗口实时显示系统信息。

显示内容示例：

- 数据时间
- 主机名字
- CPU 使用率
- 内存使用率
- 内存总大小
- 剩余有效内存
- 网络发送量
- 网络接收量

---

## 项目结构

```text
ros2-system-status-pubsub-display/
├── .gitignore
├── README.md
└── src/
    ├── status_display/
    │   ├── CMakeLists.txt
    │   ├── package.xml
    │   └── src/
    │       ├── hello_qt.cpp
    │       └── sys_status_display.cpp
    ├── status_interfaces/
    │   ├── CMakeLists.txt
    │   ├── package.xml
    │   └── msg/
    │       └── SystemStatus.msg
    └── status_publisher/
        ├── package.xml
        ├── setup.cfg
        ├── setup.py
        ├── resource/
        │   └── status_publisher
        ├── status_publisher/
        │   ├── __init__.py
        │   └── sys_status_pub.py
        └── test/
            ├── test_copyright.py
            ├── test_flake8.py
            └── test_pep257.py
````

---

## 功能说明

### 1. `status_interfaces`

该功能包用于定义项目中的自定义消息类型 `SystemStatus.msg`。

消息字段如下：

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

该消息用于统一传输系统状态数据，方便发布节点和显示节点进行解耦。

---

### 2. `status_publisher`

该功能包是一个 Python 节点，用于周期性采集系统状态并发布消息。

主要功能：

* 获取当前时间戳
* 获取主机名称
* 获取 CPU 使用率
* 获取内存占用情况
* 获取网络发送/接收流量
* 将上述信息封装为 `SystemStatus` 消息
* 以固定周期发布到话题 `sys_status`

核心实现依赖：

* `rclpy`
* `psutil`
* `platform`

默认发布周期为 **1 秒一次**。

---

### 3. `status_display`

该功能包是一个 C++ 节点，用于订阅 `sys_status` 话题，并用 Qt 界面显示收到的数据。

主要功能：

* 订阅 `sys_status` 话题
* 接收 `SystemStatus` 消息
* 将消息内容格式化为字符串
* 使用 `QLabel` 在 Qt 窗口中实时显示系统状态

该包中包含两个可执行程序：

* `hello_qt`：一个简单的 Qt 测试程序，用于验证 Qt 环境是否正常
* `sys_status_display`：系统状态显示主程序

核心依赖：

* `rclcpp`
* `status_interfaces`
* `Qt5 Widgets`

---

## 运行环境

建议环境：

* Ubuntu 22.04
* ROS 2 Humble
* Python 3
* Qt5
* colcon

---

## 依赖安装

### 1. 安装 ROS 2

请先确保已经正确安装 ROS 2（如 Humble 版本）。

### 2. 安装 Python 依赖

`status_publisher` 使用了 `psutil` 获取系统信息，因此需要安装：

```bash
pip install psutil
```

如果你的系统限制全局安装 Python 包，也可以使用虚拟环境安装。

### 3. 安装 Qt 开发环境

`status_display` 依赖 Qt5 Widgets。若未安装，可执行：

```bash
sudo apt update
sudo apt install qtbase5-dev
```

---

## 编译方法

在工作空间根目录下执行：

```bash
colcon build
```

编译完成后，加载环境：

```bash
source install/setup.bash
```

---

## 运行方法

建议开启两个终端。

### 终端 1：运行系统状态发布节点

```bash
source install/setup.bash
ros2 run status_publisher sys_status_pub
```

### 终端 2：运行 Qt 显示节点

```bash
source install/setup.bash
ros2 run status_display sys_status_display
```

运行后会弹出一个 Qt 窗口，用于显示当前系统状态。

---

## Qt 测试程序

如果你只想先验证 Qt 是否可用，可以运行：

```bash
source install/setup.bash
ros2 run status_display hello_qt
```

如果成功弹出显示 `Hello Qt!` 的窗口，说明 Qt 环境配置正常。

---

## 节点与话题关系

### 节点

* `sys_status_pub`
* `sys_status_display`

### 话题

* `/sys_status`

通信关系如下：

```text
sys_status_pub  ----publish---->  /sys_status  ----subscribe---->  sys_status_display
```

---

## 代码思路

整个项目采用 ROS 2 的典型分层思路：

1. **接口层**
   使用 `status_interfaces` 统一定义消息格式。

2. **数据发布层**
   使用 `status_publisher` 周期性读取本机系统状态，并发布成标准 ROS 2 消息。

3. **显示层**
   使用 `status_display` 订阅消息，并通过 Qt 图形界面进行展示。

这种设计方式将“数据采集”和“数据显示”分离，结构清晰，便于后续扩展，例如：

* 改成表格界面
* 增加磁盘占用监控
* 增加温度监控
* 增加多主机状态显示
* 将数据显示改为图表

---

## 可改进的地方

目前项目已经具备完整的发布-订阅-显示流程，但还有一些可以继续完善的方向：

* 补全 `package.xml` 和 `setup.py` 中的项目描述信息
* 在 README 中加入运行截图
* 使用更美观的 Qt 界面组件，而不仅是 `QLabel`
* 将时间显示改为更直观的日期时间格式
* 增加磁盘、进程、温度等更多监控项
* 增加异常处理和节点退出清理逻辑
* 对显示界面进行布局优化

---

## 学习点

这个项目适合用于练习和展示以下内容：

* ROS 2 自定义消息
* ROS 2 Python 发布者节点
* ROS 2 C++ 订阅者节点
* Qt 与 ROS 2 的结合使用
* 系统状态采集
* 跨语言通信（Python + C++）

---

## License

Apache-2.0

---

## 作者

该项目为 ROS 2 学习过程中的实践项目，主要用于练习：

* 自定义消息
* 发布订阅通信
* Python 与 C++ 混合开发
* Qt 可视化显示

[4]: https://github.com/sceneeeee/ros2-system-status-pubsub-display/blob/main/src/status_publisher/package.xml "ros2-system-status-pubsub-display/src/status_publisher/package.xml at main · sceneeeee/ros2-system-status-pubsub-display · GitHub"
