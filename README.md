# VitoLED_Display 一线通显示屏开源技术说明
## 1. 项目概述

Vito Type-C 是一款参考 PocketTouch 显示方案、基于 DJI RM330 初代遥控器屏幕 SMI550 改造的 5.5 英寸便携触摸显示屏。项目通过 USB Type-C 接口接收 DP Alt Mode 视频信号，并采用 CH32V305GBU6 搭配 CherryUSB 协议栈，通过 Type-C 中保留的 USB 2.0 通道上报 HID 多点触摸数据，使显示和触摸集中在一根线缆上。

![](https://image.lceda.cn/pullimage/BG1haDgVW3Lm797wXFCpBmSnekO5Ie0ptKuyWFsF.png)

项目面向便携副屏、移动工作台、嵌入式调试终端和小型人机交互界面等场景。整机采用全贴合触摸盖板和超薄铝合金外壳，兼顾便携性、显示效果与结构强度。

同Github项目地址：

## 2. 主要特性

| 项目       | 参数或说明                       |
| ---------- | -------------------------------- |
| 显示尺寸   | 5.5 英寸                         |
| 分辨率     | 1080 × 1920                      |
| 刷新率     | 60 Hz                            |
| 默认方向   | 竖屏，主机端可切换横屏           |
| 触摸方式   | 电容触摸                         |
| 触摸点数   | 10 点                            |
| 触摸报点率 | 120 Hz                           |
| 视频输入   | USB Type-C DP Alt Mode           |
| 视频转换   | LT7911D：DisplayPort 转 MIPI DSI |
| 主控       | CH32V305GBU6                     |
| USB 协议栈 | CherryUSB HID 多点触摸设备       |

## 3. 系统组成与工作原理

### 3.1 系统链路

整机的视频链路和触摸上报链路如下：

```text
视频：主机 → USB Type-C（DP Alt Mode）→ LT7911D → MIPI DSI → SMI550

触摸：HX83112A → CH32V305GBU6（触摸数据处理）
                        │
                        ▼
               CherryUSB 协议栈（运行于 CH32V305GBU6）
                        │ HID 多点触摸报告
                        ▼
               USB Type-C（USB 2.0）→ 主机
```

### 3.2 各模块职责

- USB Type-C 接口负责接入 DP Alt Mode 视频信号，同时通过 USB 2.0 通道承载 HID 多点触摸通信。
- LT7911D 将主机输出的 DisplayPort 视频转换为屏幕所需的 MIPI DSI 信号。
- SMI550 显示模组负责图像显示，并通过内置的 HX83112A 提供电容触摸能力。
- CH32V305GBU6 负责触摸初始化、数据读取和触点信息处理；屏幕上电时序需结合 Vito 硬件完成适配。
- CherryUSB 运行于 CH32V305GBU6，提供 USB 设备协议栈，将触点信息组织为 HID 多点触摸报告并发送给主机。

### 3.3 一线通使用条件

主机的 Type-C 接口必须支持 DP Alt Mode，所用 Type-C 线缆也需要具备相应的视频传输能力。仅支持充电或 USB 2.0 数据的线缆不能完成视频传输。

## 4. 显示子系统

### 4.1 屏幕与兼容范围

项目提供对 SMI550-E66、SMI550-D66、SMI550-B66 的固件支持。

可直接安装 GS055 屏幕，但 GS055 的初始化指令和触摸协议与 SMI550 不同，当前未提供相应支持，不能直接视为软硬件完全兼容。

### 4.2 MIPI DSI 显示参数

通过网站可以读出hex固件内的屏参信息：

![](https://image.lceda.cn/pullimage/j6p820WoYyDemOuVIgDiR3S1VeyTMryuhwFldraJ.png)

### 4.3 LT7911D 固件

项目提供 LT7911D 固件：

| 文件                               | 用途                                                   |
| ---------------------------------- | ------------------------------------------------------ |
| `LT7911D_SMI550_swap_20260703.hex` | 正式固件，需要主机 TypeC 握手 AltMode 输入 DP 视频信号 |

## 5. 触摸与 USB 通信

### 5.1 触摸方案

触摸芯片是 Himax 的 HX83112A 显示与触摸一体芯片。

屏幕触摸控制感谢@洋洋 大佬通过购买小白控遥控器并拆机抓包后逆向得到触摸屏通信协议. 同时也参考了 Github 上几个开源的 Himax 驱动. 通信过程使用 PulseView 抓包。项目地址：[baoqi-zhong/PocketTouch: 带电容式触摸的TypeC一线通便携显示器](https://github.com/baoqi-zhong/PocketTouch/tree/main)

### 5.2 CherryUSB HID 多点触摸

Vito 采用 CH32V305GBU6 搭配 CherryUSB 协议栈，在主机侧呈现为 HID 多点触摸设备。触摸固件的处理流程为：

1. CH32V305GBU6 初始化 HX83112A，读取并解析触摸数据。
2. 固件处理触点编号、有效状态、坐标和触点数量，并完成显示方向对应的坐标映射。
3. 通过 CherryUSB 的 HID 设备功能向主机发送多点触摸报告，由主机触摸驱动接收。

## 6. 固件烧录与调试

![](https://image.lceda.cn/pullimage/JxgGA46zkhq5KjLkMNna3Au7E1JB64EKt9gTpxUh.png)

另见工程[龙讯 · 升级开发工具](https://oshwhub.com/lemon_wifi/project_yoxflzuh)，枚举一个`CH341设备`及`WCH LINK-E设备`。

### 6.1 CH32V305 烧录方式

目录包含以下文件：

- `CH32V305_DJI055_MCU.hex`

使用 WCH-LinkE USB 转 SWD 工具，通过板载图中 **橙色** 处引脚进行烧录。

### 6.2 LT7911D 烧录方式

目录包含文件：

- `LT7911D_SMI550_swap_20260703.hex`

使用 CH341 USB 转 I2C 工具，通过板载图中 **蓝色** 处引脚进行烧录。

### 6.3 建议下载顺序

1. 检查 Type-C 接口、供电轨和关键电源时序，确认无短路或异常发热。
2. 烧录并运行 `CH32V305GBU6 固件`，检查 CherryUSB HID 多点触摸设备是否能够正常枚举。
3. 烧录` LT7911D 固件`，连接支持 DP Alt Mode 的主机验证图像输入。
4. 检查 1080 × 1920、60 Hz 显示模式及横竖屏切换。
5. 验证单点、多点、边缘区域、滑动连续性和 120 Hz 触摸报点表现。

## 7. 硬件与结构设计

### 7.1 硬件设计

硬件围绕 Type-C 视频输入、LT7911D 显示转换、CH32V305GBU6 触摸控制和屏幕供电四部分展开。视频与触摸分别使用 DP 和 USB 2.0 通道，通过同一个 Type-C 接口连接主机。

| 电路模块    | 设计内容                                                     |
| ----------- | ------------------------------------------------------------ |
| Type-C 输入 | 引出 DP 视频、USB 2.0 数据和供电通道；配套完成 CC 检测、DP Alt Mode 协商及正反插信号映射，具体实现以原理图为准 |
| 显示转换    | LT7911D 接收 DP 视频，并通过 MIPI DSI 连接屏幕；外围包括供电、时钟、复位和固件下载相关电路 |
| 触摸主控    | CH32V305GBU6 连接屏幕触摸接口，完成 HX83112A 初始化和触摸数据处理；USB 设备端运行 CherryUSB，向主机上报 HID 多点触摸数据 |
| 屏幕接口    | 根据 SMI550 模组定义连接 MIPI DSI、触摸通信、复位、供电和背光信号，并核对连接器针数、间距及接触面方向 |
| 电源与背光  | 将 Type-C 输入电源转换为各芯片和屏幕所需的电源轨，配置去耦、滤波及背光驱动；电压、电流和上电时序按实际器件资料与原理图确定 |
| 下载与调试  | 预留 CH32V305GBU6 下载接口、LT7911D 烧录测试点，以及电源、复位和触摸通信测试点，便于分模块调试 |

### 7.2 结构设计

外壳仅 TypeC母座 高度较高，所以外露 TypeC母座 的外壳设计，外壳壁厚夹紧内屏。

![](https://image.lceda.cn/pullimage/Ja3Y3Qsr4PtrGq25GmAwjz73uAffECfC45kLTEmJ.png)

## 8. 复刻所需资料与器件

### 8.1 关键器件

| 类别         | 建议选择                                             |
| ------------ | ---------------------------------------------------- |
| 屏幕模组     | SMI550-E66/D66/B66                                   |
| 视频桥接芯片 | LT7911D                                              |
| 主控制器     | CH32V305GBU6                                         |
| 外部接口     | 支持 DP Alt Mode 与 USB 2.0 的 USB Type-C 接口的线缆 |
| 结构件       | PETG 外壳                                            |

## 9. BOM

下表为按当前方案整理的主要物料清单。已确定的核心器件列出型号，其余器件需根据最终原理图和 PCB 补齐位号、封装、规格及数量；本表暂不作为可直接下单的完整生产 BOM。

| 类别         | 器件或物料          | 型号 / 规格            | 数量 | 说明                                                         |
| ------------ | ------------------- | ---------------------- | ---- | ------------------------------------------------------------ |
| 显示模组     | 5.5 英寸触摸屏      | SMI550-E66 / D66 / B66 | 1    | 1080 × 1920，使用与模组匹配的初始化配置                      |
| 视频转换     | DP 转 MIPI DSI 芯片 | LT7911D                | 1    | 配套项目显示固件                                             |
| 触摸主控     | MCU                 | CH32V305GBU6           | 1    | 运行触摸驱动与 CherryUSB HID 固件                            |
| 外部接口     | USB Type-C 母座     | CX90M-16P              | 1    | 核对安装方式、板厚及外壳开口                                 |
| 屏幕偏置电源 | 正负压电源芯片      | LP3102QVF              | 1    | 输入 V33S，VP / VN 分别连接 LCD_P / LED_N；使能信号为 LCDP_EN，输出电压未在图中标明 |
| 背光电源     | 背光驱动芯片        | SY7203DBC              | 1    | 输入 V33S，输出连接 BL_A，FB 连接 BL_LED1_N；EN/PWM 由 LCD_PWM_OUT 控制 |
| 电源转换     | 1.8 V 稳压芯片      | WL2803E18-5/TR         | 1    | 输入 V33S，输出 VCC_1V8                                      |

## 10. 图集

![](https://image.lceda.cn/pullimage/BG1haDgVW3Lm797wXFCpBmSnekO5Ie0ptKuyWFsF.png)

![](https://image.lceda.cn/pullimage/ETXhT4pAEb3KhFKQrURsEPxulOzPB0Rw7OaPViAe.png)

![](https://image.lceda.cn/pullimage/j9GvxZYrqQXFkTbCrAGk5Q2BTzg1Zz30A9xbYJiu.png)

![](https://image.lceda.cn/pullimage/Ja3Y3Qsr4PtrGq25GmAwjz73uAffECfC45kLTEmJ.png)

## 11. 展望

Vito一线通显示屏采用 TypeC 一线通，支持10点触摸。

*但是未加入PRSwap/DrSwap反向上行host充电功能，可能是结构自身原因，无法再做一个TypeC供充电器使用了。*
