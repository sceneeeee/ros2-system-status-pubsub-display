# ros2-face-recognition

一个基于 **ROS 2 Python Service** 的人脸检测项目。  
项目使用 `face_recognition` 和 `OpenCV` 完成人脸位置检测，并通过 **自定义服务接口** 将检测结果返回给客户端，再由客户端进行可视化显示。

这个项目适合用来练习：

- ROS 2 Python 节点开发
- ROS 2 Service 通信机制
- 自定义 `.srv` 接口设计
- `cv_bridge` 在 ROS 图像消息与 OpenCV 图像之间的转换
- `face_recognition` 的基础使用
- OpenCV 图像绘制与显示

---

## 项目功能

本项目实现了一个完整的人脸检测服务流程：

1. 客户端读取本地测试图片
2. 客户端将 OpenCV 图像转换为 ROS 图像消息
3. 客户端调用 `face_detect` 服务
4. 服务端接收图像并执行人脸检测
5. 服务端返回：
   - 检测到的人脸数量
   - 检测耗时
   - 每张人脸的边框坐标
6. 客户端根据返回坐标在原图上绘制矩形框并显示结果

此外，项目还包含一个独立学习脚本 `learn_face_detect.py`，可用于不经过 ROS 2 通信、直接验证人脸检测功能。

---

## 项目结构

```text
ros2-face-recognition/
├── .gitignore
├── README.md
└── src/
    ├── chapt4_interfaces/
    │   ├── CMakeLists.txt
    │   ├── package.xml
    │   └── srv/
    │       └── FaceDetector.srv
    └── demo_python_service/
        ├── package.xml
        ├── setup.py
        ├── setup.cfg
        ├── resource/
        │   ├── default.jpg
        │   └── test1.jpg
        └── demo_python_service/
            ├── __init__.py
            ├── learn_face_detect.py
            ├── face_detect_node.py
            └── face_detect_client_node.py
````

---

## 功能包说明

### 1. `chapt4_interfaces`

该功能包用于定义项目中的自定义服务接口：

```text
srv/FaceDetector.srv
```

服务接口包含两部分：

#### Request

* `sensor_msgs/Image image`

#### Response

* `int16 number`：检测到的人脸数量
* `float32 use_time`：检测耗时
* `int32[] top`
* `int32[] right`
* `int32[] bottom`
* `int32[] left`

其中四个数组分别用于保存每张人脸边框的坐标。

---

### 2. `demo_python_service`

该功能包包含项目的主要 Python 节点与测试资源：

#### `learn_face_detect.py`

一个独立的人脸检测学习脚本，不依赖 ROS 2 的服务通信。

功能：

* 读取 `resource/default.jpg`
* 使用 `face_recognition.face_locations()` 检测人脸
* 用 OpenCV 在图像上绘制检测框
* 弹窗显示检测结果

适合用于先验证：

* `face_recognition` 是否安装成功
* OpenCV 是否能正常读取和显示图片
* 人脸检测逻辑本身是否正确

---

#### `face_detect_node.py`

服务端节点，节点名为：

```text
face_detect_node
```

功能：

* 创建名为 `face_detect` 的服务
* 接收客户端发送的图像消息
* 将 ROS 图像消息转换为 OpenCV 图像
* 使用 `face_recognition` 执行人脸检测
* 返回检测结果给客户端

当前使用的人脸检测参数为：

```python
self.number_of_times_to_upsample = 1
self.model = "hog"
```

说明：

* `number_of_times_to_upsample = 1`：对图像放大一次后检测，有助于识别较小人脸，但会增加耗时
* `model = "hog"`：使用 HOG 模型，速度较快，更适合 CPU 环境

额外逻辑：

* 如果客户端请求中没有携带图像数据，服务端会自动使用默认图片：
  `resource/default.jpg`

---

#### `face_detect_client_node.py`

客户端节点，节点名为：

```text
face_detect_client
```

功能：

* 创建到 `face_detect` 服务的客户端
* 读取本地测试图片 `resource/test1.jpg`
* 将 OpenCV 图像转换成 ROS 图像消息
* 异步调用服务
* 接收服务端返回的人脸数量、耗时和边框坐标
* 在原图中绘制人脸框
* 显示检测结果

客户端输出示例：

```bash
Number of faces detected: 1, Time taken: 0.15 seconds
```

---

## 系统工作流程

整个项目的数据流如下：

```text
本地图片
   ↓
客户端读取图片
   ↓
转换为 ROS Image 消息
   ↓
调用 face_detect 服务
   ↓
服务端接收请求
   ↓
转换为 OpenCV 图像
   ↓
face_recognition 检测人脸
   ↓
返回人数、耗时、坐标
   ↓
客户端绘制人脸框
   ↓
OpenCV 弹窗显示结果
```

---

## 运行环境

建议环境：

* Ubuntu 22.04
* ROS 2 Humble
* Python 3
* colcon

---

## 依赖安装

### 1. ROS 2 相关依赖

确保已经正确安装 ROS 2，并能正常使用 `colcon`。

### 2. Python 依赖

本项目核心依赖包括：

* `face_recognition`
* `opencv-python`
* `cv_bridge`
* `ament_index_python`
* `rclpy`

其中 `face_recognition` 通常依赖 `dlib`，安装时可能需要较完整的编译环境。

如果你使用的是系统 Python，推荐优先确认 ROS 环境与 Python 环境兼容后再安装。

---

## 编译方法

在工作空间根目录执行：

```bash
colcon build --packages-select chapt4_interfaces demo_python_service
```

编译完成后加载环境：

```bash
source install/setup.bash
```

---

## 运行方法

建议开启两个终端。

### 终端 1：启动服务端

```bash
source install/setup.bash
ros2 run demo_python_service face_detect_node
```

### 终端 2：启动客户端

```bash
source install/setup.bash
ros2 run demo_python_service face_detect_client
```

运行后客户端会：

1. 在终端输出检测到的人脸数量和耗时
2. 弹出 OpenCV 窗口显示带有人脸框的结果图像

---

## 单独测试学习脚本

如果你只想先测试人脸检测逻辑，不跑 ROS 2 服务通信，可以运行：

```bash
source install/setup.bash
ros2 run demo_python_service learn_face_detect
```

该脚本会直接读取默认图片并显示检测结果。

---

## 关键设计说明

### 为什么这里使用 Service？

这个项目使用的是 **Service（请求-响应）** 模式，而不是 Topic。

这样设计比较适合当前任务，因为：

* 输入是一张图片，请求具有明确的开始和结束
* 输出是一次性的检测结果
* 逻辑上更接近“发一张图，返回一组结果”

如果后续想做连续检测、摄像头实时识别，更适合改成 Topic 或 Action。

---

## 已知特点与注意事项

### 1. 客户端显示阶段会阻塞

客户端在显示图像时使用了：

```python
cv2.waitKey(0)
```

这会让程序一直等待键盘输入，因此窗口不关闭时，节点不会自然退出。

### 2. 虽然请求是异步的，但显示仍然可能卡住退出流程

客户端调用服务时使用了异步方式，但 OpenCV 窗口本身仍可能造成阻塞。

### 3. 图片路径依赖包资源安装位置

代码通过：

```python
get_package_share_directory('demo_python_service')
```

获取图片路径，因此需要保证 `resource/default.jpg` 和 `resource/test1.jpg` 已随包正确安装。

### 4. `face_recognition` 环境配置可能比较敏感

如果系统中没有正确安装 `dlib` 或 `face_recognition`，程序可能无法运行。

---

## 可改进方向

这个项目已经完成了基础的人脸检测服务流程，后续可以继续扩展：

### 1. 支持摄像头实时检测

把当前“读取本地图片”改成：

* USB 摄像头输入
* 视频流检测
* 订阅 ROS 图像话题

### 2. 支持更多检测模型

当前使用的是：

```python
model = "hog"
```

未来可以尝试：

* `cnn` 模型
* GPU 加速方案

### 3. 增强错误处理

例如：

* 图片读取失败
* 服务不可用
* 返回结果为空
* OpenCV 窗口异常关闭

### 4. 支持结果保存

除了弹窗显示，还可以把画完框的图片保存到本地。

### 5. 扩展为识别而不仅是检测

当前项目做的是 **face detection**（检测人脸位置），后续可以进一步扩展到：

* 人脸编码
* 人脸比对
* 多人身份识别

---

## 学习价值

通过这个项目，可以比较完整地练习以下内容：

* ROS 2 Python 节点编写
* ROS 2 Service 服务端 / 客户端通信
* 自定义 `.srv` 接口设计
* `cv_bridge` 图像格式转换
* OpenCV 图像绘制与显示
* `face_recognition` 的基础调用流程
* 异步回调处理方式

---

## License

Apache-2.0
