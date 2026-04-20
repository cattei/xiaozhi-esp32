# Zectrix AI便利贴 — 完整硬件清单（源码反推）

> 基于 Zectrix 开源固件源码（`zectrix-s3-epaper-4.2`）分析，BoardType: `zectrix-s3-epaper-4.2`

---

## 一、核心主控

| 项目 | 型号/参数 | 来源文件 |
|---|---|---|
| **主控芯片** | ESP32-S3-WROOM-1-N16R8 | config.h + 产品规格 |
| **CPU** | Xtensa 32位 LX7 双核 240MHz | 芯片固有 |
| **Flash** | 16MB | config.h（N16R8） |
| **PSRAM** | 8MB | config.h（N16R8） |
| **Wi-Fi** | 2.4GHz 802.11 b/g/n | 芯片固有 |
| **蓝牙** | BLE 5.0 | 芯片固有 |

---

## 二、屏幕模组

| 项目 | 型号/参数 | 引脚 | 来源文件 |
|---|---|---|---|
| **屏幕类型** | 4.2 英寸 E-ink 电子墨水屏 | — | config.h |
| **屏幕驱动 IC** | **SSD1683** | — | custom_lcd_display.h 第113行注释 + 初始化命令 |
| **分辨率** | 400 × 300（横屏） | — | config.h `EXAMPLE_LCD_WIDTH/HEIGHT` |
| **显示颜色** | 黑白双色（1bpp） | — | custom_lcd_display.h `COLOR_IMAGE` |
| **刷新方式** | 全刷 + 局刷（2bpp交织） | — | custom_lcd_display.cc |
| **SPI 总线** | SPI3_HOST | — | config.h `EPD_SPI_NUM` |
| **SPI 时钟** | GPIO12 | EPD_SCK_PIN | config.h |
| **SPI 数据** | GPIO13 | EPD_MOSI_PIN | config.h |
| **片选** | GPIO11 | EPD_CS_PIN | config.h |
| **数据/命令** | GPIO10 | EPD_DC_PIN | config.h |
| **复位** | GPIO9 | EPD_RST_PIN | config.h |
| **忙状态** | GPIO8 | EPD_BUSY_PIN | config.h（低电平=忙） |
| **屏幕电源** | GPIO6 | EPD_PWR_PIN | config.h |
| **SPI 频率** | 40MHz（写）/ 8MHz（读） | — | custom_lcd_display.cc |

### SSD1683 关键命令序列

```
0x00 → 0x2F, 0x2E    （PSD 驱动设置）
0xE9 → 0x01           （OTP 加载）
0x40                  （读温度传感器）
0xE0 → 0x02           （设置边界）
0xE6                  （LUT 温度补偿）
0xA5                  （全刷激活）
0x10                  （DTM1 数据写入）
0x04                  （Power On）
0x12 → 0x00           （Display Refresh）
0x02 → 0x00           （Power Off）
```

### SSD1683 温度补偿 LUT

```
≤5°C    → 232（-24）
≤10°C   → 235（-21）
≤20°C   → 238（-18）
≤30°C   → 241（-15）
≤127°C  → 244（-12）
其他    → 232（-24）
```

---

## 三、音频系统

| 项目 | 型号/参数 | 引脚 | 来源文件 |
|---|---|---|---|
| **音频编解码** | **ES8311** | — | config.h `AUDIO_CODEC_ES8311_ADDR` |
| **I2C 地址** | ES8311 默认地址 | — | config.h |
| **I2C SDA** | GPIO47 | AUDIO_CODEC_I2C_SDA_PIN | config.h |
| **I2C SCL** | GPIO48 | AUDIO_CODEC_I2C_SCL_PIN | config.h |
| **I2S 主时钟** | GPIO14 | AUDIO_I2S_GPIO_MCLK | config.h |
| **I2S 位时钟** | GPIO15 | AUDIO_I2S_GPIO_BCLK | config.h |
| **I2S 字选择** | GPIO38 | AUDIO_I2S_GPIO_WS | config.h |
| **I2S 播放输出** | GPIO45 | AUDIO_I2S_GPIO_DOUT | config.h |
| **I2S 录音输入** | GPIO16 | AUDIO_I2S_GPIO_DIN | config.h |
| **功放控制** | GPIO46 | AUDIO_CODEC_PA_PIN / Audio_AMP_PIN | config.h |
| **音频电源** | GPIO42 | Audio_PWR_PIN | config.h |
| **采样率** | 16kHz（输入+输出） | — | config.h |
| **输入通道** | 1（单麦） | — | es8311_audio_codec.cc `input_channels_ = 1` |
| **输入增益** | 30 | — | es8311_audio_codec.cc `input_gain_ = 30` |
| **I2S 模式** | 全双工（I2S_NUM_0） | — | es8311_audio_codec.cc |
| **MCLK 倍频** | 256× | — | es8311_audio_codec.cc `I2S_MCLK_MULTIPLE_256` |
| **数据位宽** | 16bit | — | es8311_audio_codec.cc |
| **PA 电压** | 5.0V | — | es8311_audio_codec.cc `pa_voltage = 5.0` |
| **DAC 电压** | 3.3V | — | es8311_audio_codec.cc `codec_dac_voltage = 3.3` |

---

## 四、RTC 时钟

| 项目 | 型号/参数 | 引脚 | 来源文件 |
|---|---|---|---|
| **RTC 芯片** | **PCF8563T** | — | config.h 注释 `RTC (PCF8563T/5)` |
| **I2C 地址** | 0x51 | — | config.h `RTC_I2C_ADDR` |
| **中断引脚** | GPIO5 | RTC_INT_GPIO | config.h |
| **通信总线** | 与音频共用 I2C0（SDA=GPIO47, SCL=GPIO48） | — | zectrix-s3-epaper-4.2.cc |

### PCF8563 寄存器映射

```
0x00 = Control_1       0x01 = Control_2
0x02 = Seconds         0x03 = Minutes
0x04 = Hours           0x05 = Days
0x06 = Weekdays        0x07 = Months
0x08 = Years           0x09 = Alarm Minute
0x0A = Alarm Hour      0x0B = Alarm Day
0x0C = Alarm Weekday   0x0E = Timer Control
0x0F = Timer Value
```

### PCF8563 Control_2 位定义

```
Bit 3 = AF  (Alarm Flag)
Bit 2 = TF  (Timer Flag)
Bit 1 = AIE (Alarm Interrupt Enable)
Bit 0 = TIE (Timer Interrupt Enable)
```

---

## 五、NFC

| 项目 | 型号/参数 | 引脚 | 来源文件 |
|---|---|---|---|
| **NFC 芯片** | **GT23SC6699** | — | config.h 注释 `NFC (GT23SC6699)` |
| **I2C 地址** | 0x55 | — | config.h `NFC_I2C_ADDR` |
| **场检测引脚** | GPIO7 | NFC_FD_GPIO | config.h |
| **场检测有效电平** | 低电平（0） | — | config.h `NFC_FD_ACTIVE_LEVEL = 0` |
| **NFC 电源** | GPIO21 | NFC_PWR_GPIO | config.h |
| **通信总线** | 与音频共用 I2C0（SDA=GPIO47, SCL=GPIO48） | — | zectrix-s3-epaper-4.2.cc |
| **块大小** | 16 字节 | — | zectrix_nfc.h `kBlockSize = 16` |
| **UID 块地址** | 0x00 | — | zectrix_nfc.h |
| **用户数据区** | 块 0x01~0x37（共 848 字节） | — | zectrix_nfc.h |
| **命令区起始** | 块 0x04 | — | zectrix_nfc.h `kCommandBlockAddress` |
| **NDEF 支持** | Text / URI | — | zectrix_nfc.cc |

---

## 六、按钮

| 项目 | 引脚 | 功能 | 来源文件 |
|---|---|---|---|
| **确认/BOOT** | GPIO0 | 短按确认，长按 1000ms | config.h `BOOT_BUTTON_GPIO` / `TODO_CONFIRM_BUTTON_GPIO` |
| **上翻页** | GPIO39 | 短按触发 | config.h `TODO_UP_BUTTON_GPIO` |
| **下翻页/电源** | GPIO18 | 短按触发（与电源键复用） | config.h `TODO_DOWN_BUTTON_GPIO` / `VBAT_PWR_GPIO` |

---

## 七、电源与充电

| 项目 | 引脚 | 功能 | 来源文件 |
|---|---|---|---|
| **系统总电源** | GPIO17 | 高电平开机，低电平关机 | config.h `VBAT_PWR_PIN` |
| **屏幕电源** | GPIO6 | 高电平=开 | config.h `EPD_PWR_PIN` |
| **音频电源** | GPIO42 | 高电平=开 | config.h `Audio_PWR_PIN` |
| **功放控制** | GPIO46 | 高电平=开 | config.h `Audio_AMP_PIN` |
| **充电检测** | GPIO2 | 低电平=充电中 | config.h `CHARGE_DETECT_GPIO` / `CHARGE_DETECT_CHARGING_LEVEL = 0` |
| **充满检测** | GPIO1 | 高电平=充满 | config.h `CHARGE_FULL_GPIO` |
| **电池类型** | 2000+mAh 18650 锂电池 | — | 产品规格 |
| **电池电压检测** | ADC1 通道3（GPIO4） | — | zectrix-s3-epaper-4.2.cc `ADC_CHANNEL_3` |
| **电压衰减比** | 2倍（`raw_voltage * 2`） | — | zectrix-s3-epaper-4.2.cc |

### 电池电量计算公式

```
computed_percent = (-1 * V² + 9016 * V - 19189000) / 10000
```

其中 V 为 10 次采样平均电压（mV），结果限制在 0~100%。

---

## 八、LED 指示灯

| 项目 | 引脚 | 功能 | 来源文件 |
|---|---|---|---|
| **状态 LED** | GPIO3 | 充电中闪烁、充满常灭、空闲常亮 | board_power_bsp.cc `PowerLedTask` |

### LED 行为

| 状态 | GPIO3 行为 |
|---|---|
| 充电中 | 闪烁（200ms 亮 / 2800ms 灭） |
| 充满 | 常灭（低电平） |
| 空闲 | 常亮（高电平） |
| 工厂测试闪烁 | 500ms 周期翻转 |

---

## 九、通信接口汇总

| 总线 | 引脚 | 挂载设备 |
|---|---|---|
| **I2C0** | SDA=GPIO47, SCL=GPIO48 | ES8311（音频）、PCF8563T（RTC）、GT23SC6699（NFC） |
| **SPI3** | SCK=GPIO12, MOSI=GPIO13, CS=GPIO11 | SSD1683（墨水屏） |
| **I2S0** | MCLK=GPIO14, BCLK=GPIO15, WS=GPIO38, DOUT=GPIO45, DIN=GPIO16 | ES8311（音频） |
| **ADC1** | CH3（GPIO4） | 电池电压检测 |
| **UART** | TX/RX（扩展口外露） | 外部扩展 |
| **GPIO** | 2路（扩展口外露） | 外部扩展 |

---

## 十、完整 GPIO 分配表

| GPIO | 方向 | 功能 | 备注 |
|---|---|---|---|
| GPIO0 | 输入 | BOOT/确认按钮 | 长按 1000ms |
| GPIO1 | 输入 | 充满检测 | 高电平=充满 |
| GPIO2 | 输入 | 充电检测 | 低电平=充电中 |
| GPIO3 | 输出 | 状态 LED | 充电/状态指示 |
| GPIO4 | 输入 | 电池电压检测 | ADC1 CH3 |
| GPIO5 | 输入 | RTC 中断 | PCF8563T INT |
| GPIO6 | 输出 | 屏幕电源 | 高电平=开 |
| GPIO7 | 输入 | NFC 场检测 | 低电平=有场 |
| GPIO8 | 输入 | 屏幕忙状态 | 低电平=忙 |
| GPIO9 | 输出 | 屏幕复位 | — |
| GPIO10 | 输出 | 屏幕数据/命令 | — |
| GPIO11 | 输出 | 屏幕片选 | — |
| GPIO12 | 输出 | SPI 时钟 | 屏幕用 |
| GPIO13 | 输出 | SPI MOSI | 屏幕用 |
| GPIO14 | 输出 | I2S 主时钟 | 音频用 |
| GPIO15 | 输出 | I2S 位时钟 | 音频用 |
| GPIO16 | 输入 | I2S 录音输入 | 麦克风 |
| GPIO17 | 输出 | 系统总电源 | 高电平=开 |
| GPIO18 | 输入 | 下翻页/电源键 | 复用 |
| GPIO21 | 输出 | NFC 电源 | 高电平=开 |
| GPIO38 | 输出 | I2S 字选择 | 音频用 |
| GPIO39 | 输入 | 上翻页按钮 | — |
| GPIO42 | 输出 | 音频电源 | 高电平=开 |
| GPIO45 | 输出 | I2S 播放输出 | 喇叭 |
| GPIO46 | 输出 | 功放控制 | 高电平=开 |
| GPIO47 | I/O | I2C SDA | 音频/RTC/NFC 共用 |
| GPIO48 | I/O | I2C SCL | 音频/RTC/NFC 共用 |

---

## 十一、产品物理参数

| 项目 | 参数 |
|---|---|
| **整机尺寸** | 97mm × 97mm × 8.75mm |
| **电池** | 2000+mAh 18650 锂电池 |
| **接口** | Type-C（充电+数据） |
| **扩展口** | 3.3V / GND / TX / RX / 2×GPIO |
| **安装方式** | 磁吸背板 |
| **外壳** | 注塑开模（极简高端款）/ DIY 款 / 彩色 3D 打印款 |

---

*文档生成时间：2026-04-20*
*数据来源：Zectrix 开源固件源码分析*
