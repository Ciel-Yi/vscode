````markdown name=README.md url=https://github.com/Ciel-Yi/vscode/blob/bde71c5e79f0a8381667ff84926de13348172735/README.md
#  实验1：linux&ros基础
专业班级：电信2202  
学号：U202213875  
姓名：易笑天  

## 实验目的
- 了解：linux操作系统，linux操作系统的安装，ROS2的环境配置  
- 掌握：linux命令行的使用，ROS的工作空间，功能包，节点。  
- 重点：linux和ROS2基本命令的使用

## 实验任务
- 配置ubantu环境
- 安装ROS2
- 测试ROS2
- 车辆电量心跳发布器

## 实验内容
### 1. 安装VMware 
VMware Workstation Pro 17 下载安装教程 
注意选择将VMware Workstation 17 用于个人用途 
### 2. VMware导入Ubuntu22.04的镜像 
Ubuntu22 镜像网站：Index of /ubuntu-releases/22.04/   
点击创建虚拟机   
选择典型—下一步   
选择ubuntu22.04镜像的下载位置   
输入你的用户名和密码   
选择安装位置，下一步  
分配磁盘大小，建议60GB以上   
自定义硬件参数，这里给到了8GB的内存和8核的CPU处理器 
完成创建 
### 3. 打开虚拟机，按照向导完成Ubuntu的配置 
### 4. 配置ROS2环境 
按Ctrl+Alt+t 召唤出终端   
输入鱼香ros命令一键安装   
`wget http://fishros.com/install -O fishros && . fishros`  
选择1，安装humble版本的ROS2（桌面版），同时也安装Vscode ide以进行后续项目的开发 
### 5. Hello,world 节点测试 
新建一个终端，输入以下命令启动一个数据的发布者节点：   
`ros2 run demo_nodes_cpp talker`   
启动第二个终端，通过以下命令启动一个数据的订阅者节点：   
`ros2 run demo_nodes_py listener`   
如果“Hello World”字符串在两个终端中正常传输，说明通信系统没有问题   
### 6. 小海龟仿真测试 
新建一个终端，输入以下命令启动小海龟的可视化仿真：   
`ros2 run turtlesim turtlesim_node`  

<!-- 使用正确的 Markdown / HTML 图片语法，且通过 raw.githubusercontent.com 获取原始图片以确保在 README 中正确显示 -->
<img src="https://raw.githubusercontent.com/Ciel-Yi/vscode/main/run_turtle.png" width="50%" alt="turtle.png" /> 

再新建一个终端，输入以下命令，使用键盘控制小海龟的移动：  
`ros2 run turtlesim turtle_teleop node`
### ROS2 基础 
#### 1. 创建工作空间目录： 
`mkdir -p ros_car_status/src`
#### 2. 创建功能包 
`cd ros_car_status/src`   
`ros2 pkg create --build-type ament_cmake ros_car_status`   
其中，使用 --build-type 指定编译系统为 ament_cmake   
ros_car_status：自定义功能包名称   
其中，有 `[WARNING]: Unknown license 'TODO: License declaration'. ROS2`  
建议创建一个 License 文件以说明该功能包的发布许可。可以使用 -  
license LICENSE 参数指定： 
`ros2 pkg create --build-type ament_cmake --license Apache-2.0 ros_car_status`   
生成的文件目录如下：   
#### 3. 编译源文件： 
使用code .命令，用vscode打开工作空间 
在 ros_car_status/src 目录下新增 publisher.cpp 文件，文件内容如下：  

```cpp
#include <chrono> 
#include <iomanip>      // 为了 std::fixed << std::setprecision 
#include "rclcpp/rclcpp.hpp" 
#include "std_msgs/msg/float32.hpp"   // 用 float32 表示剩余电量百分比 
 
using namespace std::chrono_literals; 
 
class BatteryPublisher : public rclcpp::Node 
{ 
public: 
    BatteryPublisher() 
        : Node("battery_publisher"), battery_percent_(100.0f) 
    { 
        pub_ = create_publisher<std_msgs::msg::Float32>("/car_battery", 10); 
        timer_ = create_wall_timer(500ms, std::bind(&BatteryPublisher::timer_callback, this)); 
        RCLCPP_INFO(get_logger(), "小车电量监控节点已启动，初始电量：%.1f%%", battery_percent_); 
    } 
 
private: 
    void timer_callback() 
    { 
        auto msg = std_msgs::msg::Float32(); 
        msg.data = battery_percent_; 
        pub_->publish(msg); 
 
        if (battery_percent_ > 0.0f) 
        { 
            RCLCPP_INFO(get_logger(), "发布小车剩余电量：%.1f%%", battery_percent_); 
            battery_percent_ -= 1.0f;          // 每发一次减 1% 
        } 
        else 
        { 
            // 只提示一次 
            static bool warned = false; 
            if (!warned) 
            { 
                RCLCPP_WARN(get_logger(), "电量耗尽，请充电！"); 
                warned = true; 
            } 
        } 
    } 

    rclcpp::Publisher<std_msgs::msg::Float32>::SharedPtr pub_; 
    rclcpp::TimerBase::SharedPtr timer_; 
    float battery_percent_; 
}; 

int main(int argc, char **argv) 
{ 
    rclcpp::init(argc, argv); 
    rclcpp::spin(std::make_shared<BatteryPublisher>()); 
    rclcpp::shutdown(); 
    return 0; 
}
```

4. 编辑编译配置文件CMakeList.txt 
默认生成的`CMakeList.txt`文件内容如下： 
 
由于新增了源文件`publisher.cpp`，所以要配置该文件的编译规则。 
找CMakeLists.txt`，修改如下： 
 
5. 进入到`ros_car_status`工作空间，使用如下指令编译工程： 
`colcon build` 
6. 运行节点 
先设置环境变量，即让系统可以找到节点，进入到工作空间目录，执行如下指令： 
`source install/setup.bash` 
接着运行该节点： 
`ros2 run ros_car_status battery_publisher`    
但单次执行 `source install/setup.bash` 只对当前终端有效，新打开终端仍需再执行该命令，为了避免每次执行，可以把该命令加到当前用户的 .bashrc 文件中，该文件在用户的 `home `目录下。   
方法一：直接打开` ~/.bashrc `文件，在末尾添加 `source /你的/工作/空间/目录/install/setup.bash `，保存。   
方法二：使用命令 `echo "source /你的/工作/空间/目录/install/setup.bash" >> ~/.bashrc`   
最后，使用命令 `source ~/.bashrc` 使修改生效。  
````
