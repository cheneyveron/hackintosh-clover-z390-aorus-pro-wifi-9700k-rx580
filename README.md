# 黑苹果 四叶草配置 Gigabyte z390 Aorus Pro WiFi 

## 硬件配置

- 主板：技嘉 Z390 Aorus Pro WiFi
- BIOS版本：F9
- CPU：i7-9700k
- 显卡：希世 RX580 4G + 技嘉 Gaming OC RTX 2080
- 内存：科赋雷霆 3000 C15 16G x2
- 硬盘：西部数据黑盘 SN750 500G
- 电源：海韵Focus 850W
- 散热：大镰刀千石船单风扇版本
- 网卡：BCM943602CS，在"深圳市成东数码科技"购买，3个月左右蓝牙就坏了，大家可以注意一下。

### 硬件选购Tips:

1. 主板：

- z390全系列主板自带的WiFi与蓝牙都是CNVi协议，无法正常使用。包括预留了mSATA接口的主板。
- Asus的BIOS最智能，可以根据当前散热条件和处理器的体制自动寻找最佳性能的CPU频率和电压，并且做工很好，几乎没有小毛病，但是价格太贵，尤其是供电缩的比较厉害。
- 华擎堆料最猛，但是BIOS不智能，需要手动超频，而且常常有小毛病。
- 技嘉的板子倒是颠覆了我的印象，12相豪华供电，提供了Windows下的EasyTune工具自动检测最佳频率，而且没有发现任何小毛病。技嘉还提供了一个Windows下直接修改BIOS开机画面的工具，深得DIYer的心：）缺点就是，全系列Aorus z390主板都没有提供板载DP接口，无法输出4k@60Hz。
- 不建议更新到F10版本的BIOS。F10仅更新了[Intel-SA-00233版本微码](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00233.html)，修复了CPU漏洞，要利用这些漏洞需要直接在物理上操作这台计算机，并且会导致性能下降，极端情况下下降40%。
-  如果不打算购置独显，又打算使用4K显示器，请选择带有DP口的主板，因为HDMI无法支持4k@60Hz的输出，即使显示器与主板上的HDMI口版本都是2.0。
- 不建议选择任何二手主板。

2. CPU：之所以选购9700k而非 8700k / 9900k，因为后两者发热量太大，需要使用高端240或360水冷并更换机箱，预算+1000。而9700k的性能足以应付接下来数年内的任何使用场景。
3. SSD：选购NVMe固态时，首先排除三星 970 Evo Plus、PM981系列。然后找评测，重点关注其缓存外速度。

## BIOS设置（重要）

1. Save & Exit → Load Optimized Defaults
2. Peripherals → Trusted Computing → Security Device Support →  Disable
3. Peripherals → Intel(R)Bios Guard → Intel BIOS Guard Support →  Disable
4. Peripherals → USB Configuration → Legacy USB Support →  Enabled
5. Peripherals → USB Configuration → XHCI Hand-off →  Enabled
6. Peripherals → Network Stack Configuration → Network Stack →  Disabled
7. Peripherals → SATA And RST Configuration → SATA Mode Selection →  AHCI
8. Chipset → Vt-d → Disabled
9. Chipset → Internal Graphics → Enabled
10. Chipset → DVMT Pre-Allocated → 256M
11. Chipset → DVMT Total Gfx Mem → MAX 
12. Chipset → Audio Controller → Enabled
13. Chipset → Above 4G Decoding → Disabled
14. Power → Platform Power Management → Disabled
15. Power → ErP → Disabled
16. Power → RC6 (Render Standby)

如果是4k显示器，想使用板载HDMI接口，请先将显示器的HDMI输入接口版本设置为1.4，一般在OSD菜单中可以设置。否则会显示器无输出。


## 软件说明

- 操作系统版本：macOS Mojave 10.14.5 & 10.14.6
- Clover 版本：5018
- CPU变频：正常。原生11档(1000 / 1300 / 3600 / 3700 / 4000 / 4100 / 4300 / 4400 / 4600 / 4800 / 4900)
- UHD630：正常。使用Clover注入设备ID。
- RX580：正常。原生驱动。
- 3.5mm声音 & HDMI：均正常使用，使用AppleALC驱动。
- USB：正常。使用`SSDT-UIAC.aml`搭配`USBInjectAll.kext`驱动。所有接口均可以使用，但是所有3.0接口都不可以使用2.0的设备。
- SSD Trim：正常。使用了Clover的KextsPatch。
- 有线网卡：正常。使用了IntelMausiEthernet.kext
- 睡眠唤醒：正常。
- 关机开机：正常。
- iCloud & App Store & iMessage & FaceTime：请自行生成Board Serial Number、序列号、SmUUID，并相应的修改SysPrameter系统参数中的“自定义UUID”，和RtVariables变量设置中的MLB、ROM。
- AirDrop & HandOff & Continuity：正常。
- 我写了一个小ssdt`SSDT-NVDISABLE.aml`，屏蔽了最靠近CPU的第一条PCIe接口。这个接口插着RTX 2080，如果不屏蔽，一来发热量大，二来会导致睡眠后无法唤醒。正常情况下其实不需要屏蔽这个接口，请把`EFI/CLOVER/ACPI/patched/SSDT-NVDISABLE.aml`挪出`patched`文件夹即可。

## 致谢

- [Apple](https://www.apple.com)：研发的 macOS 系统
- [Clover EFI bootloader](https://sourceforge.net/projects/cloverefiboot/)：强大的通用操作系统引导器
- @[**vit9696**](https://github.com/vit9696)：制作 Lilu & AppleALC
- @[**glasgood**](https://www.insanelymac.com/forum/profile/1077361-glasgood/)：写了一篇详细的Aorus Pro WiFi主板安装教程，见[这里](https://www.insanelymac.com/forum/topic/337837-glasgoods-macos-mojave-successguide-for-aorus-z390-pro/)
- [远景论坛](http://bbs.pcbeta.com) & [InsanelyMac](http://www.insanelymac.com)：提供交流的场所
