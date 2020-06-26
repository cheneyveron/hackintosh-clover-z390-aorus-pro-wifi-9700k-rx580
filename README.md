# 黑苹果 OpenCore配置 for Gigabyte z390 Aorus Pro WiFi 

[关于本机](https://img.itmanbu.com//wp-content/uploads/2019/12/aboutme.png)

更多图片在[这里](https://github.com/cheneyveron/hackintosh-clover-z390-aorus-pro-wifi-9700k-rx580/blob/master/pics.md)

## macOS Big Sur 11.0 特别说明

目前Big Sur仍存在诸多问题：

- 安装程序会进入死循环：无法通过macOS installer安装，需要在白果上，或虚拟机中安装完成后clone到本地。
- 第一次启动不成功：先启动Catalina，再重启进Big Sur即可。
- 电源管理缺陷：睡眠会重启，关机无法断电。白果上关机也出现失败情况，猜测系统BUG。
- ...

尝鲜建议另分一个区安装。

## 彩蛋：Z390的板载无线网卡

z390的板载网卡，除了华擎itx默认屏蔽CNVi功能外，都因CNVi协议默认开启而不支持macOS。

在关闭CFG Lock时，我搜了一下CNVi，发现在BIOS中有隐藏的开关，默认开启状态：

```
Form: Connectivity Configuration, FormId: 0x271C {01 86 1C 27 9C 07}
0x37DE4         Text: Statement.Prompt: CNVi present, TextTwo: No {03 08 9E 07 9F 07 A0 07}
0x37DEC         Text: Statement.Prompt: CNVi Configuration, TextTwo:  {03 08 A1 07 A1 07 00 00}
0x37DF4         One Of: CNVi Mode, VarStoreInfo (VarOffset/VarName): 0x10CD, VarStore: 0x1, QuestionId: 0x1C9, Size: 1, Min: 0x0, Max 0x1, Step: 0x0 {05 91 A2 07 A3 07 C9 01 01 00 CD 10 10 10 00 01 00}
0x37E05             One Of Option: Disable Integrated, Value (8 bit): 0x0 {09 07 A5 07 00 00 00}
0x37E0C             One Of Option: Auto Detection, Value (8 bit): 0x1 (default) {09 07 A4 07 30 00 01}
0x37E13         End One Of {29 02}
...
```

理论上来说，只要类似的执行`setup_var 0x10CD 0x0`，应该就可以关闭CNVi协议。然后替换上BCM94系的网卡+转接卡，是不是就能识别了呢？

我现在手头没有转接卡，有兴趣的朋友可以试一试。

## 硬件配置

- 主板：技嘉 Z390 Aorus Pro WiFi
- BIOS版本：F9
- CPU：i7-9700k @ 5.2GHz
- 显卡：希世 RX580 4G + 技嘉 Gaming OC RTX 2080
- 内存：科赋雷霆 3000 16G x2 + 海盗船  LPX 3000 16G C15
- 硬盘：西部数据黑盘 SN750 500G
- 电源：海韵Focus 850W
- 散热：猫头鹰NH-D15
- 机箱：Thermaltake Commander C35

- 网卡：BCM943602CS，在"深圳市成东数码科技"购买，3个月左右蓝牙就坏了，大家可以注意一下。目前使用的是USB蓝牙，同样可以用HandOff

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
4. 超频心得：在Windows下，使用NH-D15可以把i7-9700k超频至全核心5.4GHz，AVX Offset -10，单烤FPU、双烤CPU与FPU都只有80度左右，也不会崩溃。在macOS下，5.4GHz时会kernel panic无法开机，或者开机后迅速崩溃。5.3时在系统下没有崩溃，但是跑GeekBench 5 GPU部分时会崩溃，FCPX导出时也会黑屏。目前降低到5.2稳定运行，未发现问题。另外，在Windows下可以直接在BIOS中分别指定CPU频率与AVX的频率，而macOS必须要通过指定CPU频率与AVX Offset，否则最高频率就只有指定的AVX频率。

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
16. Power → RC6 (Render Standby) -> Disabled

如果是4k显示器，想使用板载HDMI接口，请先将显示器的HDMI输入接口版本设置为1.4，一般在OSD菜单中可以设置。

### 解锁CFG Lock

由于技嘉使用的是AMI的BIOS，所以即便是隐藏了很多设置项目，重新制作了BIOS界面，我们还是可以通过一些通用的修改工具修改他们。

通常的方式有两种：

1. 使用AMIBCP修改ROM中的默认设置，然后刷入修改后的ROM。这种方式比较简单，不再赘述。
2. 使用setup_var修改隐藏设置。这种方式和直接进入BIOS修改各种选项其实没有不同，所以如果清空COMS，设置项就会恢复初始值。这样可以避免刷新BIOS的风险。

#### 使用setup_var修改隐藏设置：

1. 查找CFG Lock的地址：
  
  ①. 使用[UEFITool](https://github.com/LongSoft/UEFITool/releases)打开自己的BIOS ROM。
  
  ②. 搜索Unicode String：`CFG Lock`
  
  ③. 双击下方搜索到的地方，跳转到一个`PE32 image section`，右击`Extract as is`，命名为`setup.bin`
  
  ④. 使用[Universal-IFR-Extractor](https://github.com/LongSoft/Universal-IFR-Extractor/releases)提取出所有设置项的地址：`path/to/ifrextract path/to/setup.bin path/to/setup.txt`
  
  ⑤. 使用文本编辑器打开提取出的`setup.txt`，搜索`CFG Lock`即可找到其地址。

如果你也是这个主板，并且是F9版本的BIOS，那么CFG Lock项的地址是`0x5C1`。F9版本的BIOS提取出的`setup.txt`我放在了`attachment/setup.txt.zip`中，如果你还需要设置其他项目，直接去里面查即可。

2. 设置CFG Lock：

 ①. 到 [这里](https://github.com/datasone/grub-mod-setup_var/releases)  下载`modGRUBShell.efi`
 
 ②. 将U盘格式化为FAT32格式，新建`EFI/BOOT`文件夹，然后将`modGRUBShell.efi`改名为`BOOTX64.EFI`放进去
 
 ③. 按住F12开机，选择U盘，然后输入`setup_var 0x5C1 0x00`即可

## 软件说明

- 操作系统版本：macOS Catalina 10.15.5 & macOS Big Sur 11.0
- OpenCore 版本：0.6.0(2020.06.26当天编译)
- CPU变频：正常。原生7档(800 / 1300 / 2000 / 2700 / 3400 / 4000 / 5200)
- UHD630：正常。DeviceProperties -> Add -> `PciRoot(0x0)/Pci(0x2,0x0)` -> `AAPL,ig-platform-id`注入ID `0300983E`。
- RX580：正常。原生驱动。
- 3.5mm声音 & HDMI：均正常使用，使用AppleALC驱动。DeviceProperties -> Add -> `PciRoot(0x0)/Pci(0x1b,0x0)` -> `layout-id`注入ID `0B000000`。
- USB：前面板仅保留3.0接口，后面板只留了TypeC接口，请自行修改`kexts/USBPorts.kext`
- SSD Trim：正常。
- 有线网卡：正常。使用了`kexts/IntelMausi.kext`
- 睡眠唤醒：正常。新安装的macOS需要将`hibernatemode`改为0，即内存睡眠。默认的3为混合睡眠，非苹果固件难以支持。另外，z390主板需要在 `系统偏好设置` -> `节能` 中打开 `启用电能小憩`功能才能自动进入睡眠。
- 关机开机：正常。
- iCloud & App Store & iMessage & FaceTime：请自行生成`MLB`、  `ROM`、`SystemSerialNumber`、`SystemUUID`，并相应的修改PlatformInfo -> Generic。
- AirDrop & HandOff & Continuity：正常。
- 我写了一个小ssdt`SSDT-NVDISABLE.aml`，屏蔽了最靠近CPU的第一条PCIe接口。这个接口插着RTX 2080，如果不屏蔽，一来发热量大，二来会导致睡眠后无法唤醒。正常情况下其实不需要屏蔽这个接口，请把ACPI->Add->0的Enable改为False。

## 为何使用 OpenCore

OpenCore的思路是，通过完善ACPI表与UEFI固件来运行macOS：

- 一方面，通过修改ACPI表，可以让硬件的描述与操作方式符合苹果的ACPI规范，从而macOS可以正确的识别和操作硬件。
- 另一方面，通过修改UEFI固件，可以提供一些固件原本没有而macOS需要使用的方法，或者将现有方法改造为macOS可以调用的接口。

这类似于从Docker到Kubernates的转变，从方法论的角度进行了总结，给出了黑果发展的方向。

OpenCore官方([这里](https://github.com/acidanthera/OpenCorePkg))提供了非常详尽的文档，建议阅读Configuration.pdf即知道每个配置项的存在的意义和作用了，待有时间再补充详细修改的地方。

## 致谢

- [Apple](https://www.apple.com)：研发的 macOS 系统
- [Clover EFI bootloader](https://sourceforge.net/projects/cloverefiboot/)：强大的通用操作系统引导器
- @[**vit9696**](https://github.com/vit9696)：制作 Lilu & AppleALC
- @[**glasgood**](https://www.insanelymac.com/forum/profile/1077361-glasgood/)：写了一篇详细的Aorus Pro WiFi主板安装教程，见[这里](https://www.insanelymac.com/forum/topic/337837-glasgoods-macos-mojave-successguide-for-aorus-z390-pro/)
- @[**cfmwan**](http://i.pcbeta.com/space-uid-8977.html)：分享的其EFI，其中有制作的修复睡眠、USB等功能的SSDT，见[这里](http://bbs.pcbeta.com/viewthread-1832693-1-1.html)
- [远景论坛](http://bbs.pcbeta.com) & [InsanelyMac](http://www.insanelymac.com)：提供交流的场所
