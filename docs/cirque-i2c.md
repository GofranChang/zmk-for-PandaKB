# 左半边 Cirque I²C（nice!view 转接板）

适用于 PandaKB Corne、nice!nano v2 和 ZMK v0.3。转接板安装在左半边，
SDA→MOSI、SCL→SCK、DR→CS 的映射已由用户确认，实机仍需测试。
使用固定提交的 [petejohanson/cirque-input-module](https://github.com/petejohanson/cirque-input-module/tree/0de55f36bc720b5be3d8880dc856d4d78baf5214)。

## 接线

| 转接板 | nice!view 插座信号 | nice!nano GPIO |
| --- | --- | --- |
| SDA | MOSI | P0.17 |
| SCL | SCK | P0.20 |
| VCC | VCC（3.3 V） | 电源 |
| GND | GND | 地 |
| DR | CS | P0.08，高电平有效 |

Cirque 必须已配置为 I²C 模式，默认 7 位地址为 `0x2a`。
SDA/SCL 需要上拉到 3.3 V（确认转接板是否已经提供）。DR 是 data-ready 中断输入。

触摸板占用左侧显示插座，Cirque 硬件 shield 会关闭该侧 OLED/显示功能。
左侧不要再叠加 `Nice_view_config`、`nice_view_custom` 或 `nice_oled`；
右半边和 dongle 的显示仍可使用。RGB 灯带使用独立 SPI3，不受影响。
引脚定义位于 `boards/shields/panda_cirque_i2c/boards/nice_nano_v2.overlay`。

## 选择固件

`build.yaml` 保留原有目标，额外增加三个固件：

| 使用方式 | 左半边 | 中央端/右半边 |
| --- | --- | --- |
| 无 dongle | `Corne_left_cirque_i2c` | 右侧使用原有普通 Corne_R 固件 |
| 带 dongle | `Corne_peripheral_left_cirque_i2c` | dongle 刷 `Corne_central_dongle_cirque_receiver`；右侧继续使用原有 dongle peripheral 固件 |

带 dongle 时，左半边将鼠标事件通过 BLE split input 通道 `0` 转发给 dongle。
dongle 必须刷配套 receiver 固件，才能将事件转换为鼠标报告。
无 dongle 时，左半边本身是中央端，直接处理触摸板输入。

附加 shield：

- `panda_cirque_i2c`：无 dongle 的左侧中央端，直接产生鼠标报告。
- `panda_cirque_i2c_peripheral`：带 dongle 的左侧外设端，转发触摸板事件。
- `panda_cirque_receiver`：叠加到 dongle，接收触摸板事件。

## 本地构建

在已安装 Zephyr SDK 并初始化好 west 的 ZMK 工作区执行，路径按实际工作区调整：

```sh
west update
west build -s zmk/app -b nice_nano_v2 -d build/corne-left-cirque -- \
  -DZMK_CONFIG=/home/gofran/workspace/zmk-for-PandaKB/config \
  -DZMK_EXTRA_MODULES=/home/gofran/workspace/zmk-for-PandaKB \
  -DSHIELD="Corne_L panda_cirque_i2c"
```

带 dongle 时，将 shield 改为 `Corne_dongle_left panda_cirque_i2c_peripheral`，
并另行构建 `Corne_dongle_central dongle_display panda_cirque_receiver`（加 Studio USB snippet）。

驱动默认提供相对鼠标移动与轻触点击。安装方向不同可调整 trackpad 节点的
`rotate-90`、`x-invert`、`y-invert` 属性。本适配未增加手势滚动或更改现有键位。

## 验证状态

已通过 YAML 构建清单、Git 差异格式检查，以及上述三个目标的 DTS 预处理与 Zephyr dtlib 解析检查（使用 ZMK v0.3 / Zephyr 3.5 官方头文件）。
此检查不包含完整 Kconfig、绑定 schema 检查或固件编译；实机移动、点击、休眠唤醒仍需验证。
