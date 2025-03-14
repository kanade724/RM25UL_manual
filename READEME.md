# RoboMaster2025 联盟赛视觉组新兵手册
## 如何连接NUC
+ 首先保证你的电脑和NUC连接同一个网络在连接前可以先输入以下命令<br>
    ```shell
    ping ip
    ```
    其中ip为NUC的ip地址，若ping成功则说明网络连接正常，若ping失败则需要重新连接网络,NUC的ip地址请访问网关10.0.0.1，通常的网关密码：*35086020*<br>
    | 兵种      | 用户名 |  IP |
    | :----------- | :-----------: | -----------: |
    | 大全向（哨兵）      | nuc1       |10.0.0.172 |
    | 舵轮（步兵）   | nuc2        |----------- |
    | 英雄   | nuc3        |----------- |
1. SSH协议<br>
    + 进入你的电脑终端，输入以下命令
        ```shell
        ssh username@ip
        ```
        其中username为NUC的用户名，ip为NUC的ip地址<br>
        若连接成功则会出现以下界面
            ```
            username@ip's password:
            ```<br>
        所有NUC的密码：*35086020*<br>
        + 这种连接方式是最快的但是只允许你使用终端交互NUC
    + 进入你的VSCode，打开远程，上方输入栏选择**连接到主机**，输入以下信息
        ```
        username@ip
        ```
        其中username为NUC的用户名，ip为NUC的ip地址<br>
        若连接成功则会出现以下界面
            ```
            username@ip's password:
            ```<br>
        所有NUC的密码：*35086020*<br>
        并打开资源管理器，你可以在资源管理器中看到NUC的文件系统<br>
        + 这种连接允许你使用VSCode交互NUC，可以使用VSCode充当文本编辑器
2. RDP协议<br>
    + 打开你的电脑系统搜索栏，搜索“rdp”,并点开**远程桌面连接**<br>
        输入ip，ip为NUC的ip地址<br>
        若连接成功则会出现蓝色界面，用户名输入username<br>
        其中username为NUC的用户名，密码：*35086020*<br>
        + 这种连接方式是最慢的但是可以使用桌面GUI交互NUC
## 1. 自瞄
## **比赛前切记注意!!! 检查armor_detector识别颜色是否为对方颜色！！！**
### 1.1 如何手动启动自瞄
+ 首先打开终端，快捷键**ctrl+alt+t**
+ 输入以下命令（进入桌面启动上下位机通信程序）
    ```shell
    cd 
    ./aucting.sh
    ```
**另开终端**
+ 输入以下命令（进入桌面启动自瞄程序）
    ```shell
    cd autoaim/
    ./bringup.sh
    ```
### 1.2 如何逐包开启自瞄
+ 目前完整的自瞄程序由四个节点构成，分别为
  + **camera** 摄像头节点
  + **armor_detector** 对相机传输图像进行处理和识别装甲板
  + **armor_processor** 对装甲板的位置进行预测
  + **change_to_autoaim** 将预测的位置转换为云台pitch和yaw的角度

由于这四个节点是相互联系的，所以需要一个一个包启动
+ camera
    ```shell
    cd autoaim/
    source install/setup.bash
    source /opt/ros/humble/setup.bash
    ros2 run camera camera
    ```
+ armor_detector
    ```shell
    cd autoaim/
    source install/setup.bash
    source /opt/ros/humble/setup.bash
    ros2 run armor_detector armor_detector_node
    ```
+ armor_processor
    ```shell
    cd autoaim/
    source install/setup.bash
    source /opt/ros/humble/setup.bash
    ros2 run armor_processor armor_processor_node
    ```
+ change_to_autoaim
    ```shell
    cd autoaim/
    source install/setup.bash
    source /opt/ros/humble/setup.bash
    ros2 run change_to_autoaim change_to_autoaim
    ```

### 1.3 针对1.1出现问题的自查
+ 检查需要的外部设备是否连接<br>
打开终端输入以下命令
    ```shell
    lsusb
    ```
观察输出是否有
<br>**Image DHcamera** 和 **STMXXXX Virtual Comm Port**
<br>若有则说明连接正常，若没有则需要重新连接外部设备，前一个是摄像头（需要接入**USB3.0**），若没有检查相机是否接入；后一个是下位机通信接口若没有检查C板是否接入（也有可能是线的问题，需要检查usb线是否支持数据传输）<br>
+ 检查上下位机通信程序是否启动：<br>
经典问题
    ```shell
    segmentation fault
    ```
    可以输入以下命令将ACM0端口赋予777权限，并检查ACM0端口有无跳转到ACM1端口，若有则需要将ACM1端口赋予777权限
    ```shell
    sudu chmod 777 /dev/ttyACM0
    ```
若启动异常则需要重新启动或尝试车辆重启，请参考1.1，重新启动无法解决请找通信相关负责人协助<br>    
<br>

+ 检查自瞄程序是否启动：<br>
若启动异常则需要重新启动或尝试车辆重启，重新启动无法解决请尝试1.2逐包启动，同时观察终端输出是否有异常，以下是一些常见异常情况
    + camera节点启动异常：<br>
        + 常见问题1：<br>
            ```shell
            no camera
            ```
            检查摄像头SN是否配置正确，若不正确，使用GUI打开GalaxyView，将摄像头SN配置正确，然后重新启动<br>

        + 常见问题2：<br>
            ```shell
            segmentation fault
            ```
            检查camera程序对于相机分辨率参数是否调整正确，若还有问题则尝试多次启动，可能是相机被占用或其他内存问题<br>

    + armor_detector、armor_processor、change_to_autoaim节点启动异常：<br>
        + 常见问题：<br>
        代码连接库出现问题，或者其他编译问题，重新编译即可解决
            ```shell
            rm -rf build/ install/ log/
            colcon build
            source install/setup.bash
            ```

+ 检查是否为自启故障（以下是一些有关自启动的常见命令）
    + 关闭/开启自瞄自启脚本<br>
        ```shell
        sudo systemctl stop autoaim.service
        sudo systemctl start autoaim.service
        ```
    + 关闭/开启上下位机通信脚本<br>
        ```shell
        sudo systemctl stop contact.service
        sudo systemctl start contact.service
        ```        
    + 检查自瞄/上下位机通信自启动状态<br>
        ```shell
        sudo systemctl status autoaim.service
        sudo systemctl status contact.service
        ```
## 2. 导航
xxx