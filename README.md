# 电光双域波形孪生与激光复刻系统

基于安路 PH1A180 FPGA：摄像头对准示波器屏幕取波形，同时在**电域**（DAC 输出到另一台示波器）和**光域**（舵机带动激光笔在紫外感光纸上绘制）中复现。

## 开发环境

| 项目 | 说明 |
|---|---|
| 开发板 | MLK-H10（PH1A180SFG676，速度等级 -2） |
| 软件 | 安路 TD 6.2.1 SP1（工程文件记录版本 6.2.178840） |
| 语言 | Verilog / SystemVerilog |
| 顶层模块 | `merge_top_full` |
| 工程文件 | `merge_syn2.al` |

## 三种工作模式

按开发板 **D9** 键循环切换，LED0 / LED1 / LED2 分别指示当前模式。

| 模式 | 数据流 | LED |
|---|---|---|
| 模式一 | 摄像头 → DAC | LED0 |
| 模式二 | AD → 舵机（激光笔绘图） | LED1 |
| 模式三 | 摄像头 → 舵机（激光笔绘图） | LED2 |

其他按键：**F8** 抓取一帧摄像头波形（128 点）；**D8** 提高摄像头曝光。
显示为 HDMI 1920×1080，左半屏为 AD 示波器界面，右半屏为摄像头画面。

## 目录结构

```
merge_syn2.al            TD 工程文件
constraints/             引脚约束 merge_pins_full.adc，时序约束 merge_syn2_timing.sdc
stage4/rtl/              顶层 merge_top_full.v、b_core.v（模式控制、周期提取、舵机绘图等）、
                         采样器 wave_sample32_uart.v、串口复用与打印
stage3/rtl/, rtl_r2/     显示合成、复位、摄像头右半屏读取、帧缓存
stage2/                  HDMI 发送相关（串行复位同步、LVDS 并串转换等）
rtl/                     显示时序 ui_vtc_xy.v、AD 示波器核心 b_scope_core.v
ip/                      PLL、HDMI 发送控制器（加密 IP）
b_src/                   AD 接口、NCO、IQ 检测、波形分类、串口发送
a_src/uisrc/             摄像头采集（MIPI/CSI、ISP）、DDR3、波形提取、DAC904 回放
bitstream/               编译好的位流 merge_syn2.bit（压缩版 merge_syn2_compress.bit）
```

目录名中的 stage2/3/4 是分阶段合并时留下的名称，没有改动，以保证工程路径一致。

## 使用方法

1. 用 TD 6.2.1 SP1 打开 `merge_syn2.al`。
   工程文件中记录的根目录是原开发机路径 `D:/anlogic/merge_cam_scope/merged`。
   如果在其他电脑上打开后提示找不到文件，请在 TD 中把工程目录指到本仓库根目录，
   或把本目录放到该路径下。
2. 三个 RAM IP（`ui_ram_awb_delay_signal`、`ui_ram_demosaic`、`ui_ram_zhenghe`）
   的 `.ipc` 已包含，必要时在 TD 中重新生成。
3. 依次执行综合、布局布线、生成位流，下载到开发板；
   也可以直接下载 `bitstream/merge_syn2.bit`。
4. 连接 HDMI 显示器，摄像头对准示波器屏幕，按 D9 选择模式。

## 接线与引脚

引脚以 `constraints/merge_pins_full.adc` 为准。
DAC904、AD9226、舵机串口经转接板接入开发板，转接板电平转换器件如下：

- J14 电平转换板：TXS0108E，把 J14 的 1.8 V IO 转为 3.3 V，供 TTL 串口舵机使用；
- FMC 转接板：SN74AVC16T245（数据）和 LSF0204-Q1（时钟），完成 3.3 V 与 FMC VADJ 之间的转换。

若实际接线与约束文件不一致，请先按实际接线修改引脚约束再编译。

## 说明

- 仓库内 `*.pre_*`、`*.bak` 等历史备份与 TD 中间文件已排除（见 `.gitignore`）。
- 仿真使用 ModelSim，按阶段与原工程做对比回归；仿真脚本未包含在本压缩包中。
