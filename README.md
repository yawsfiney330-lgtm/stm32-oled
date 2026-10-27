# STM32 OLED 字符显示

基于现有 STM32CubeIDE `oled` 工程，使用 STM32F103C8Tx、I2C1 和波特律动 CH1116 OLED 驱动，在 128×64 OLED 上显示英文、数字、符号和中文。

当前内容：

```text
OLED 10Hz
Frame:0000000001
!@#$%&*+-=?:
    刘何瑾瑜
```

任务四：屏幕内容每 100 ms 刷新一次，即 10 Hz。第二行帧数每次刷新加 1，约一秒增加 10。

主循环使用 `HAL_GetTick()` 的无符号时间差调度，先检查距离上一次帧开始是否达到 `OLED_REFRESH_PERIOD_MS`（100 ms），再清空显存、绘制文字并调用 `OLED_ShowFrame()`。绘制和 I2C 传输时间包含在周期内，不是在传输后再延时 100 ms。屏幕控制器内部扫描频率仍由原驱动配置。

调试器中可以观察 `oledFrameCount`（已发送帧数）、`oledFrameStartMs`（本帧开始时刻）、`oledFramePeriodMs`（两帧起点间隔）和 `oledRenderTimeMs`（绘制及发送耗时）。正常运行时周期应为 100 ms，耗时应小于 100 ms。调试暂停、断点、通信异常或其他耗时任务会影响实际刷新率。

## 工程结构

- `Core/Src/main.c`：应用入口及显示内容；自定义代码放在 USER CODE 区域。
- `Core/Src/oled.c`、`Core/Inc/oled.h`：现有 OLED 驱动与绘制接口。
- `Core/Src/font.c`、`Core/Inc/font.h`：ASCII 字库及新增的 `fontDemo16x16` 中文字库。
- `Drivers/`：CMSIS 和 STM32 HAL 驱动，随仓库保留以支持重新构建。
- `oled.ioc`：STM32CubeMX 外设与时钟配置。
- `.project`、`.cproject`、`.settings/stm32cubeide.project.prefs`：IDE 工程配置。
- `STM32F103C8TX_FLASH.ld`：链接脚本。

## 打开和运行

在 STM32CubeIDE 中导入本目录的 Existing Projects into Workspace，然后构建 `oled` 的 Debug 配置，并通过 ST-Link 下载运行。工程当前使用 I2C1：PB6 为 SCL，PB7 为 SDA；屏幕按原驱动地址配置使用。

中文字模由 [波特律动取模工具](https://led.baud-dance.com/) 生成：文泉驿·点阵字16、16×16、列行式、阳码、波特律动 OLED 驱动模板。源文件使用 UTF-8。增加其他汉字时，需要同时补充对应字模。

初始版本已于 2026-10-27 使用 STM32CubeIDE 2.1.1 构建，0 错误、0 警告；已完成 ST-Link 下载校验，运行后读回的显存与预期内容一致，I2C 状态无错误。

## Git 日常使用

在本工程目录打开终端：

```bash
git status
git diff
git add Core/Src/main.c
git commit -m "调整 OLED 显示内容"
git log --oneline --decorate -10
```

一次提交对应一个明确的修改。修改中文时，通常需要一起提交 `main.c`、`font.c` 和 `font.h`。

`Debug/`、`Release/`、编译产物、自动生成的编译器发现配置和本机 `.launch` 调试配置被忽略；它们仍保留在电脑上。源码、驱动、`.ioc` 和必要 IDE 配置纳入版本管理。

仓库先在本地使用。可以通过 `git remote -v` 查看是否配置了远程地址。
