# cpu

[toc]

### 概述

#### 1.有四个特权级：
* ring 0		
能够访问所有机器指令（内核处于ring 0特权级）
* ring 1
* ring 2
* ring 3		
能够访问部分机器指令（应用程序处于ring 3特权级）

#### 2.实现特权级切换的方式：中断