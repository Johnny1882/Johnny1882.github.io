---
title: FPGA控制的多人在线游戏
header:
  overlay_image: "assets/wide_bgs/snow.jpeg"
categories:
  - Projects
tags:
    - C#
    - Unity
    - Python
    - AWS
    - Project
toc: true
---

# 信息处理报告

## 系统目标

我们的目标是创建一个使用 FPGA 作为方向控制器的引人入胜的多人游戏。通过利用 AWS 服务器、本地 FPGA 处理和 Unity 基础设施，团队开发了一个合作型 2D 竞技游戏。

## 游戏描述

我们在 Unity 上使用 C# 开发了一个合作实时多人游戏。
- 该游戏支持两名玩家在同一台笔记本电脑上实时游戏，并且可以通过增加控制节点来扩展以容纳更多玩家。
- 目标是与随机生成的怪物战斗。玩家必须合作击败怪物，任何玩家死亡游戏就结束。

我们的目标是复制类似于在 PlayStation 上玩多人游戏的控制台多人游戏体验。

## 性能指标

### 低延迟

我们使用了 UDP 通信协议，并在数据传输中使用了低采样率以减少延迟。FPGA 命令与游戏中实现之间的延迟约为 300ms，提高了用户体验。

### 高精度

我们增加了一个 FIR 滤波器以减少噪声，并设置了 `data_x` 和 `data_y` 的最佳范围来决定移动方向。

### 多人合作游戏

使用 AWS 服务器，我们将信息传输到 Unity 计算机，以在同一屏幕上显示两名玩家的实时控制，模拟 PS5 或 Nintendo Switch 的体验。

### 移动方向的高灵活性

玩家可以在 X、Y 和对角方向上控制他们的游戏角色。

## 设计决策

我们将项目分为三个阶段：

1. FPGA 设计和数据处理
2. 服务器端设计
3. Unity 游戏开发

我们根据团队成员的专长将他们分配到每个阶段，以便同时进行工作。

## FPGA 设计和数据处理

### 使用 FIR 滤波数据

```c
#define N 49 
float COEFF[N] = {0.0046,0.0074,-0.0024,-0.0071,0.0033,0.0001,-0.0094,0.0040,0.0044,-0.0133,0.0030, 
0.0114,-0.0179,-0.0011,0.0223,-0.0225,-0.0109,0.0396,-0.0263,-0.0338,0.0752,-0.0289,-0.1204,0.2879, 
0.6369,0.2879,-0.1204,-0.0289,0.0752,-0.0338,-0.0263,0.0396,-0.0109,-0.0225,0.0223,-0.0011,-0.0179, 
0.0114,0.0030,-0.0133,0.0044,0.0040,-0.0094,0.0001,0.0033,-0.0071,-0.0024,0.0074,0.0046}; 

#define OFFSET -32 
#define PWM_PERIOD 16 
#define QUITLETTER '~' 
#define CHARLIM 64 

alt_32 firFilter(alt_32 *sample, int n, alt_32 input){ 
 int index = n-1; 
 alt_32 results; 
 results = 0; 
 for (int j = 0; j<index; j++){ 
   sample[j] = sample[j+1]; 
  } 
 for (int i= 0; i<index; i++){ 
  alt_32 num = sample[index-i]; 
  results = results + num/n; 
 } 

 results = results + input/n; 
 sample[n-1] = input; 
 return results; 
}
```

我们使用 FIR 滤波数据而不是原始数据来减少错误。MATLAB 生成了 49 个系数。该函数接受一个样本数组、样本数量（`n`）和一个新输入值，并返回滤波后的输出值。尽管增加滤波器会增加延迟，但系统其余部分的延迟优化使其保持较低水平，因此我们保留了滤波器以获取其降噪性能。

### 实现七段显示

```c
while(loop){ 
    printf("<--> Please Type Your Username (6 bits maximum) and Press Enter: <-->\n"); 

    char text[2*CHARLIM] = "";  
    char prevLetter = '!'; 
    int length = 0; 
    int running = 1; 
    prevLetter = alt_getchar();   
    prevLetter = get_text(prevLetter, &length, text, &running); 

    IOWR_ALTERA_AVALON_PIO_DATA(HEX5_BASE, getBin(text[0])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX4_BASE, getBin(text[1])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX3_BASE, getBin(text[2])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX2_BASE, getBin(text[3])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX1_BASE, getBin(text[4])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX0_BASE, getBin(text[5]));

    // print score 
    char text[2*CHARLIM] = ""; 
    char prevLetter = '!'; 
    int length = 0; 
    int running = 1; 
    prevLetter = alt_getchar(); 
    prevLetter = get_text(prevLetter, &length, text, &running); 

    IOWR_ALTERA_AVALON_PIO_DATA(HEX5_BASE, getBin(text[0])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX4_BASE, getBin(text[1])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX3_BASE, getBin(text[2])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX2_BASE, getBin(text[3])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX1_BASE, getBin(text[4])); 
    IOWR_ALTERA_AVALON_PIO_DATA(HEX0_BASE, getBin(text[5]));
```

我们设计了一个可以显示字符和数字的七段显示器。它在游戏的两个部分中使用：

1. 游戏开始时显示用户名。
2. 游戏结束时显示分数。

### 实现按钮

```c
button = IORD_ALTERA_AVALON_PIO_DATA(BUTTON_BASE); 
if(button == 0b10){//press key0 to stop 
 char stop='a'; 

    still_in_game = 0; // if will finish the game and print the mark 
    alt_up_accelerometer_spi_read_x_axis(acc_dev, & x_read); 
    alt_up_accelerometer_spi_read_y_axis(acc_dev, & y_read); 
    alt_up_accelerometer_spi_read_z_axis(acc_dev, & z_read); 
    x_read = firFilter(& samples_x, n, x_read); 
    y_read = firFilter(& samples_y, n, y_read); 
    z_read = firFilter(& samples_z, n, z_read); 
    char x_dir, y_dir; 
    x_dir = 'a'; 
    y_dir = 'a'; 
    alt_printf("%c%c\n", x_dir, y_dir); 
}
```

按下 key 0 停止游戏并在屏幕上显示分数。

### Nios 和 Python 之间的数据传输

运动数据被收集并传输到 Python 进行客户端处理。为了解决数据传输瓶颈，我们添加了 “usleep” 函数来创建传输时间间隔，在不影响数据的情况下显著提高了性能和稳定性。

该命令作为使用 Python subprocess 库在内部创建的 bash 系统调用中执行。按下按钮后，一个字符串 “aa” 会从 Nios 发送出去，接收到它后，正在运行的子进程终止，另一个在 `send_on_jtag1(cmd)` 函数中打开，以接收从数据库返回的分数数据，然后将其打印在 FPGA 上。

## 服务器和数据库

### 服务器

服务器在 AWS EC2 实例上设置，运行 Ubuntu 20.04。我们选择 AWS EC2 是因为其可扩展性和可靠的基础设施。必要的软件，包括 Apache Web 服务器、Python 3.8 和 UMP，已经安装。AWS CLI 允许本地数据库访问

#### 通信协议

我们尝试了几种通信协议：

1. 使用 DynamoDB 延迟显著。
2. TCP 减少了延迟，但仍有 2-3 秒的延迟。
3. UDP 将延迟降低到 1 秒以下。

我们选择了低延迟的 UDP 协议，这对游戏至关重要。

#### 关于 TCP 和 UDP 的更多信息

TCP 的可靠性和面向连接的特性增加了开销，导致延迟。UDP 作为无连接协议，传输数据包更快，延迟更低，非常适合像游戏这样的实时应用。然而，UDP 的不可靠性可能导致数据包丢失，导致错误或故障。可能需要数据包丢失检测、错误修正和拥塞控制等技术来提高性能。

### 数据

我们的游戏是 2D 的，攻击是自动的，所以我们只需要传输玩家的移动信息。FPGA 处理并输出两个数字，分别表示垂直和水平速度。

#### Python 代码

##### 连接

```python
def init():  # connect and remember three clients 
    print("initializing, please connect player1") 
    msg1, add1 = server_socket.recvfrom(2048) 
    time.sleep(1) 

    print("initializing, please connect player2") 
    msg2, add2 = server_socket.recvfrom(2048) 
    time.sleep(1) 

    print("initializing, please connect destination") 
    msg3, add3 = server_socket.recvfrom(2048) 
    time.sleep(1) 

    return add1, add2, add3 

def connect():  # refuse and retry the connection when one player appears multiple times  
    add1, add2, add3 = init() 
    if (add1 == add2): 
            print("two player have the same address") 
            add1, add2, add3 = init() 
    if (add1 == add3 or add2 == add3): 
            print("destination and player have the same address") 
            add1, add2, add3 = init() 
    return add1, add2, add3 

add1, add2, add3 = connect()
```

这段代码进行连接。由于我们无法知道客户端的公共 IP，服务器会接收每个客户端的一条消息并记住 IP。如果一个 IP 多次出现，服务器将拒绝并重试连接。

##### 传输

```python
while True: 
    #cadd below is the client process address 
    cmsg, cadd = server_socket.recvfrom(2048) 
    cmsg = cmsg.decode() 
    id = cmsg[:2]  # get first 3 characters of string

    # Information like scores and Health is transmitted using Database, so the transmission is one-way. We add a header in front of the message so that the game can distinguish between player1 and player2.

    if (cadd == add1): 
        cmsg = 'C1:' + cmsg 
        print("C1:" + cmsg) 
        server_socket.sendto(cmsg.encode(), add3) 

    if (cadd == add2): 
        cmsg = 'C2:' + cmsg 
        print("C2:" + cmsg) 
        server_socket.sendto(cmsg.encode(), add3)
```

### 数据库

游戏数据库旨在为多用户存储各种与游戏相关的数据。数据库托管在 Amazon Web Services (AWS) DynamoDB 上，这是一种 NoSQL 数据库服务。

DynamoDB 的优点：

1. 无论数据结构如何，都可以处理不同类型的数据，在游戏服务器中创造了灵活性。
2. 低延迟，非常适合我们实时多人游戏服务器的快速响应时间需求。

游戏数据库的功能包括：

1. **检索玩家数据:** \`get_player_data\` 函数检索给定玩家 ID 的数据。
2. **添加新玩家及其登录数据:**  \`new_player_data\` 和 \`new_login_data\` 函数分别添加新玩家数据和登录凭证。
3. **更新玩家数据：** \`update_player_data\` 函数更新数据库中的玩家数据。
4. **扫描表格：** \`show_table\` 函数输出指定表的所有项目。

#### 延迟测试

延迟由软件和硬件部分组成。

**软件部分：**

```text
协议/延迟  测试1    测试2    测试3    测试4    测试5    平均
UDP       165ms     343.2ms   263ms     302.3ms   187.2ms   252.14ms
```

**硬件部分:**

```text
运行50次：

平均值  最大值     最小值
0.62 ms  250.23 ms 0.53 ms

```

### Unity 游戏开发

为了控制玩家的移动，FPGA 输出被解析为两个值并通过 UDP 传输。从 UDP 服务器收集的数据被写入两个 txt 文件以控制玩家的移动。游戏结束后，分数被写入另一个 txt 文件，更新到数据库，并显示在 FPGA 上。

测试流程包括对每个组件的单元测试和在将它们集成到最终系统中时的迭代改进。

## 相关链接

要查看最终的系统交付成果和演示，请参阅我们与本次课程作业一起提交的视频。它展示了现场游戏以及对实现的评论。Group14

[相关代码](http://host.py/)

[成果展示](https://imperiallondon-my.sharepoint.com/:v:/g/personal/bz1521_ic_ac_uk/EQWuP7PvxvNBj7r5vfuSiUABnPlbUIslF7B5U7uDMGvb-w)
