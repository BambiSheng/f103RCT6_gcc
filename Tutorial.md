# 利用VSCode+EIDE优雅地开发STM32

![image-20231022205637954](https://s2.loli.net/2023/10/22/mJp8iEB4ACOlyKg.png)

## 前言

- 在即将到来的电子设计大赛中，我们将要使用STM32开发板来完成小车的开发。而如果你笔者一样并不喜欢STM32CubeIDE、Keil等主流STM32开发环境上的编码体验，希望借助VSCode丰富的插件生态~~(例如Copilot）~~、强大的代码补全来优雅地完成STM32的开发，那么这篇Weekly**或许**将为你带来公元2023年VSCode平台上最好的STM32开发环境配置方案！
- `EIDE（Embedded IDE）`是一款Vscode平台上的嵌入式开发插件，支持 6 种不同的编译器与4种主流的烧录工具，可以被用来开发 `stm32/stm8/risc-v` 等项目，在提供了与Keil、IAR等嵌入式开发IDE相似的开发逻辑和UI的同时，也能让用户获得VSCode平台更好的编码体验。同时，EIDE也支持跨平台开发（win/Linux/Mac）、导入Keil_MDK/IAR/Eclipse项目，功能非常强大。在此基础上借助`Cortex-Debug`插件，我们可以实现在VSCode中无缝完成编码、编译、烧录与调试。

## 环境配置概述

对于STM32的开发，我们一般会经历以下几个流程：

- 模板代码框架的生成：完成引脚、时钟、外设等的基本配置，同时生成基础代码
- 用户代码的编写：在基本代码框架的基础上根据用户所需要实现的功能完成代码编写
- 编译：将写好的代码编译为二进制文件
- 烧录：将编译生成的二进制文件烧录至开发板中
- 调试：与同学们在程设课上接触到的断点调试类似，通过调试器连接开发板来实现打断点、监视变量、内存等，从而完成Debug

依据此流程，我们需要以下工具：

- STM32CubeMX：大家已经很熟悉了，是意法半导体开发的一款为STM32系列芯片生成基础开发代码的图形化界面软件；我们可以在STM32CubeMX上选择所需的芯片型号、完成串口、引脚、ADC等的配置

- VSCode及一些有助于你Coding的插件：根据自己的喜好和需要配置即可

- ARM-GCC/AC5/AC6等：这些都是STM32开发中常用的编译器，需要指出的是：

  - AC5/AC6是Keil MDK中的付费编译器，且只有Windows平台版本；
  - 而GCC是全平台的开源免费编译器；
  - 当然，开源免费的GCC在性能表现上与付费闭源编译器相比还是有一定的差距，不过对于小型项目来说差别不大；以下是一张对比图：

  <img src="https://s2.loli.net/2023/10/22/i497QWBTw2sPaIV.png" alt="image-20231022205230521" style="zoom:50%;" />

- 烧录工具：例如JLink，OpenOCD，STM32 Cube Programmer等，与我们所用的硬件调试器种类相关；我们在电设中将要用到的ST-Link就可以使用OpenOCD或者Cube Programmer来配合进行烧录

- 调试工具：我们使用VSCode插件Cortex-Debug作为调试工具，同时EIDE也会提供常见芯片的支持包来作为辅助

在这个过程当中，VSCode+EIDE+Cortex-Debug的工作流能够几乎完美地涵盖编码、编译、烧录与调试这四个步骤；我们在EIDE的图形化界面中能够一键下载安装与切换常见的编译器、烧录工具以及芯片支持包，同时更改编译/调试的相关选项；与使用繁琐的命令行相比，这种具有集成性的GUI的工作流无疑能够大大减小用户（尤其是对开发环境并不是很熟悉的新手）在环境配置上所花的时间精力。接下来，笔者将以`Windows 11`系统为例来介绍EIDE+VSCode的环境配置。

## EIDE，启动！

在浏览本教程前，请确保**您的用户名不为中文，且一切相关软件的安装路径中均不存在中文**

### 测试环境

笔者在`Windows 11`操作系统下进行测试，EIDE/Cortex-Debug插件均为2023.10.20时的最新版本，STM32CubeMX版本为`6.9.10`

### 基本软件/插件安装

- 安装JAVA环境（推荐`JDK8`）：
  - 这一步是因为`STM32CubeMX`需要JAVA环境
  - 官网链接：[Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/#java8)
  - 也可以参考：[jdk8的安装教程保姆级-CSDN博客](https://blog.csdn.net/m0_53623945/article/details/123978985)
- 安装`STM32CubeMX`（最新版即可）：
  - 官网链接：[STM32CubeMX: Graphical tool - STMicroelectronics - STMicroelectronics](https://www.st.com/content/st_com/en/stm32cubemx.html)

- 安装`VSCode`：
  - 官网链接：[Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)
  - 并在VSCode扩展栏中安装以下插件：
    - `Embedded IDE`：我们的主角！
    - `C/C++ Extension Pack`：有助于C/C++开发的插件包
    - `Cortex Debug`：调试工具
    - （可选）`Chinese (Simplified) (简体中文) Language Pack`：汉化语言包

### EIDE配置

- 在VSCode左侧的资源栏中点击EIDE图标，进入EIDE提供的图形化界面

<img src="https://s2.loli.net/2023/10/22/s78fDBZVKPac53T.png" alt="image-20231022215312388" style="zoom: 33%;" />

- 点击操作-安装实用工具，在弹出的菜单栏中安装以下工具：
  - `CppCheck`
  - `OpenOCD Programmer`：烧录工具
  - `GNU Arm Emdedded Toolchain(stable)`：即ARM-GCC
  - `STM32 Cube Programmer CLI`：烧录工具
- 点击操作-打开插件设置，搜索`elf`，勾选`Axf To Elf`

<img src="https://s2.loli.net/2023/10/22/RO3fZEDdskYicqQ.png" alt="image-20231022220124937" style="zoom: 75%;" />

- 至此，我们已经完成了相关软件/插件的基础配置

### 使用GCC编译器完成点灯

- 现在我们将以`STM32F03RCT6`开发板为例，介绍使用GCC编译器时从基础代码生成到编译、烧录、调试的全流程

#### 生成基础代码并导入EIDE配置文件

- **[太长不看]**如果不想了解具体过程，也可以直接clone我配置好的基础代码：[f103RCT6_gcc (github.com)](https://github.com/BambiSheng/f103RCT6_gcc)
  
  - 用CubeMX打开`f103RCT6_gcc.ioc`，可以自行更改相关设置
  - 用VSCode打开`f103RCT6_gcc.code-workspace`，可以进入VSCode工作区
  - 然后就可以跳到下一小节：烧录点灯程序
  
- 打开STM32Cube MX

- 点击File-New Project

  <img src="https://s2.loli.net/2023/10/22/xHuJPDGKom3grY9.png" alt="image-20231022222045603" style="zoom: 50%;" />

- 在左上角搜索框中搜索开发板芯片型号，这里以`STM32F03RCT6`为例；选择完毕后点击右上角的Start Project

<img src="https://s2.loli.net/2023/10/22/1RJ3QlEBfHLYkje.png" alt="image-20231022224755169" style="zoom: 33%;" />

- 在左侧菜单栏中，点击SystemCore-SYS，更改Debug为Serial Wire

<img src="https://s2.loli.net/2023/10/22/PumaiLD1IlEpb8T.png" alt="image-20231022225020775" style="zoom:33%;" />

- 点击SystemCore-RCC，选择HSE项为Crystal/Ceramic Resonator

<img src="https://s2.loli.net/2023/10/22/iJztAVWay7FsHmd.png" alt="image-20231022230043430" style="zoom: 33%;" />

- 点击上侧菜单栏中的Clock Configuration，按照下图配置时钟树：

<img src="https://s2.loli.net/2023/10/22/rJazoLVXinGpcWd.png" alt="image-20231022225130734" style="zoom:33%;" />

- 接下来点击上侧菜单栏中的Project Manager，配置工程名称与路径，同时更改Toolchain/IDE项为Makefile

<img src="https://s2.loli.net/2023/10/22/69SvhfcOEHkl4sL.png" alt="image-20231022230341382" style="zoom:33%;" />

- 接下来点击左侧菜单栏的Code Generator，勾选“只复制必需的库文件”，然后点击Generate Code

<img src="https://s2.loli.net/2023/10/22/EoUVCwYxzpsnb1X.png" alt="image-20231022234345674" style="zoom:33%;" />

- 再打开VSCode-EIDE，点击新建项目-空项目-Cortex M项目

  <img src="https://s2.loli.net/2023/10/22/LeslPiMyDpF27zH.png" alt="image-20231022230525454" style="zoom:33%;" />

- 依据提示输入工程名称；**注意，工程名称必须与刚刚在CubeMX中输入的相同**（这里以`test_gcc`为例），然后在弹出的路径选择框中选择我们刚刚创建基础代码的路径（这里以`E:\projects\stm32\stm32cubeMX`为例）；此时VSCode右下角会弹出是否覆盖目录以及是否切换工作区的询问，全部选是即可

<img src="https://s2.loli.net/2023/10/22/O4udYlNmEfb7Ios.png" alt="image-20231022230940131" style="zoom:50%;" />

<img src="https://s2.loli.net/2023/10/22/CowbAHSaczmL7GW.png" alt="image-20231022231453145" style="zoom:50%;" />

<img src="https://s2.loli.net/2023/10/22/9YQUWO7Z5ERlr8y.png" alt="image-20231022231507892" style="zoom:50%;" />

- 此时我们的工作区配置就出现了！

<img src="https://s2.loli.net/2023/10/22/gHZA6VRdmFhUcPp.png" alt="image-20231022231600278" style="zoom:50%;" />

- 点击项目资源-添加源文件夹-普通文件夹，把目录下的Core/Driver添加进去

<img src="https://s2.loli.net/2023/10/22/KRuAvim2rPzWGYN.png" alt="image-20231022231802988" style="zoom: 50%;" />

- 构建配置项选择GCC，连接脚本路径改为`STM32F103RCTx_FLASH.ld`
- 项目属性中，添加以下包含目录
  - `Core/Inc`
  - `Drivers/CMSIS/Device/ST/STM32F1xx/Include`
  - `Drivers/CMSIS/Include`
  - `Drivers/STM32F1xx_HAL_Driver/Inc`
- 项目属性中，添加以下预处理宏定义
  - `USE_HAL_DRIVER`
  - `STM32F103xE`


<img src="https://s2.loli.net/2023/10/22/RQAJx3puDkg6Zb1.png" alt="image-20231022234723713" style="zoom: 50%;" />

- 将`startup_stm32f103xe.s`拖入Core文件夹下
- 点击右上角的构建，不出意外的话我们就能看到终端里出现编译成功的提示了！

<img src="https://s2.loli.net/2023/10/22/wquAlPSgz1oKkas.png" alt="image-20231022234946420" style="zoom:50%;" />

- 需要指出的是，这里只包含了我们点灯所需要的最基本的库文件；如果我们后续需要使用其他库文件时，需要类似地手动添加包含目录/宏定义

#### 烧录点灯程序

- 首先再次打开`.ioc`文件，进入CubeMX；更改PA8的引脚模式为`GPIO_Output`；重新生成代码

<img src="https://s2.loli.net/2023/10/23/MGSon6BtWDfal4E.png" alt="image-20231023001903167" style="zoom:33%;" />

- 让我们在`./Core/Src/main.c`中写一个小小的点灯程序

```c
/* USER CODE BEGIN 2 */
  int count = 0;
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    count++;
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_8);
    HAL_Delay(100);
  }
  /* USER CODE END 3 */
}
```

- 回到VSCode中，更改EIDE烧录配置为OpenOCD/STLink，重新点击右上角的构建，构建完成后点击烧录；不出意外的话，我们在终端中会收到烧录成功的提示，而我们的开发板上的灯也开始闪烁了！

<img src="https://s2.loli.net/2023/10/23/HPx6mh5YIwJME8C.png" alt="image-20231023002647800" style="zoom:50%;" />

#### 进行调试

- 找到`./.vscode/launch.json`，改为以下配置：

  ```json
  {
      "version": "0.2.0",
      "configurations": [
          {
              "cwd": "${workspaceRoot}",
              "type": "cortex-debug",
              "request": "launch",
              "name": "stlink",
              "servertype": "openocd",
              "executable": "build\\Debug\\test_gcc.elf",
              "runToEntryPoint": "main",
              "configFiles": [
                  "interface/stlink.cfg",
                  "target/stm32f1x.cfg"
              ],
              "toolchainPrefix": "arm-none-eabi"
          }
      ]
  }
  ```

- 点击VSCode左侧菜单栏的“运行与调试”，勾选stlink；然后在合适的地方打上断点，并在监视框内加入`count`变量，点击绿色三角开始调试

<img src="https://s2.loli.net/2023/10/23/wQ9Gne3vOLytXjT.png" alt="image-20231023003207184" style="zoom:33%;" />

- 如果不出意外的话，我们就能正常地监视`count`的值了！移动几步，我们看到`count`由0变成了1

<img src="https://s2.loli.net/2023/10/23/8RNn4jmacbsSIor.png" alt="image-20231023003645659" style="zoom:33%;" />

- 注意！如果提示`count`变量`optimize out`，可能是因为我们聪明的优化器把这个没用的变量优化掉了:(，此时只需降低优化等级即可

- 点击构建配置-构建器选项，在右侧的C/C++编译器菜单栏下即可更改代码优化等级，点击右上角全部保存即可

<img src="https://s2.loli.net/2023/10/23/2L4a15N6eknHjOG.png" alt="image-20231023004252793" style="zoom:33%;" />



### 关于AC5/AC6

- 如果你想使用Keil MDK中的AC5/AC6编译器，EIDE也给出了优秀的解决方案！不仅如此，我们在配置好以上两个编译器后，还可以一键将Keil项目导入EIDE进行开发，非常的便捷
- 具体配置过程可以参考[VSCode+EIDE开发STM32 哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nr4y1R7Jb/?spm_id_from=333.337.search-card.all.click)，这里不再赘述

## 结语

- 本期Weekly只是为大家简单地介绍了EIDE入门级的功能，EIDE还有更多更强大的功能正等待着大家去探索！
- 配置过程比较长，如果遇到意外情况，请善用Google！如果还有解决不了的问题，可以联系笔者交流讨论；我的邮箱是：`chengl22[at]mails[dot]tsinghua[dot]edu[dot]cn`
- 如果你发现了本文中的错误，也请务必联系我，万分感谢！
- 最后祝大家电设快乐！感谢你有耐心看到这里！

## 参考

- [Embedded IDE For VSCode ](https://em-ide.com/zh-cn/docs/intro/)

- [【精选】超好用的开发工具-VScode插件EIDE_vscode eide-CSDN博客](https://blog.csdn.net/m0_51220742/article/details/122344897?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-122344897-blog-107585306.235^v38^pc_relevant_anti_t3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-122344897-blog-107585306.235^v38^pc_relevant_anti_t3&utm_relevant_index=2)

- [VSCode+EIDE开发STM32 哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nr4y1R7Jb/?spm_id_from=333.337.search-card.all.click)