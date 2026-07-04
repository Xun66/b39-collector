# B39 空气质量检测仪 ESPHome 接入

这个目录提供一份 ESPHome 示例配置，用 ESP32-S3 的 USB-OTG Host 功能直接读取正和清远 B39 空气质量检测仪的 USB-CDC 数据，并通过 Home Assistant 展示传感器实体。

## 硬件要求

- [焊接了双 USB 口的 ESP32-S3 开发板](https://item.taobao.com/item.htm?id=669443108979)（注意，ESP32-C3 不支持 USB Host 功能）
- **部分开发板需短接 USB-OTG 焊盘才能为空气质量检测仪输出 5V 电源。请确认可自行焊接或商家可帮忙焊接。**

## 通信协议

B39 通过 USB-CDC 主动上报文本行，行尾为 `\r\n`。每行包含 8 个逗号分隔字段：

```text
particle,pm25,hcho,co2,temperature,humidity,tvoc,sequence
```

示例：

```text
0,0,1,0,24,56,0,451
```

不同传感器组合的 SKU 会保留相同的字段顺序，未搭载或无数据的字段通常为 `0`。本配置已经保留所有数据实体，请根据自己购买的传感器组合，将不用的传感器项目用井号 `#` 注释掉（见 `sensor:` 一段）。

## 使用方法

1. 确保 ESPHome Builder 右上角“三个点” -- “密钥”中已配置以下配置项：

```yaml
# 输入你的 Wi-Fi 名称（只支持 2.4G 或双频合一）
wifi_ssid: "your-ssid"

# 输入你的 Wi-Fi 密码
wifi_password: "your-password"

# 随机生成 ota_password
# https://www.random.org/passwords/?num=5&len=16&format=html&rnd=new
ota_password: "ota-password"

# 随机生成 api_encryption_key （如果暂不开启API加密则不用配置）：
# https://esphome.io/components/api/#:~:text=Copy-,Regenerate,-NOTE
api_encryption_key: "your-api-encryption-key"
```

2. 将标有 COM 的 TypeC 口连接到电脑，将标有 USB 口的 TypeC 口连接空气质量检测仪。
3. 在 ESPHome Builder 右下角选择“创建设备” -- “高级配置选项” -- “空配置”，粘贴 `configuration.yaml` 中的内容。
4. **大多数情况下，YAML 无需修改即可工作。**后续可以根据实际购买的传感器组合，在 YAML 中将不用的传感器项目用 `#` 注释掉；也可以根据网络状况配置 `wifi.use_address` 或 `wifi.manual_ip`，以及**开启 API 加密（强烈建议）**。
5. 保存文件，将开发版按住Boot再按下RST，并选择“安装（Install）” -- 选择“插入此电脑，或手动下载” -- 随后打开 [ESPHome Web](https://web.esphome.io/) 进行烧录即可，随后按ESPHome Web上的提示可以查看板子日志。一旦联网成功后，后续安装无需再使用 USB 连接设备，选择无线方式即可。
6. 在 HA -- Settings -- 设备（Devices）中查看是否已经自动发现或添加了 `B39 Air Monitor xxxxxx`，若没有，请右下角添加 ESPHome -- 输入设备的IP地址，继续，如设置了加密Key输入加密Key -- 完成，可以看到传感器数据。

## 指示灯

此配置使用 GPIO48 上的单颗 WS2812B 作为指示灯：

- Home Assistant API 连接正常：绿色闪烁。
- API 未连接：红色闪烁。

## 注意事项

- 如果 B39 能通信但不充电，请检查开发板 USB-OTG 供电焊盘是否正确短接。
- 实测B39 的 USB 口数据推送频率约为 30 秒一次。
