# ASUS ROG Z490G Hackintosh

Building a Hackintosh on ROG STRIX Z490-G GAMING using OpenCore.

## Software
| Name | Version |
| :-: | :-: |
| macOS | 15.1 |
| OpenCore | 1.0.2 |
| AppleALC | Latest |
| Lilu | Latest |
| NVMeFix | Latest |
| VirtualSMC | Latest |
| WhateverGreen | Latest |


## Hardware
| Part | Model | Comments |
| :-: | :-: | :-: |
| CPU | Intel Core i5-10600KF | Non-iGPU Model |
| MOBO | ASUS ROG Z490G | Wi-Fi Version |
| RAM | Samsung DDR4-3200 32GBx2 | Total 128GB |
| GPU | AMD RX 5700 XT | Support from macOS 11.4 onwards |
| Storage 1 | SK Hynix Gold P31 NVMe 1TB | Main OS |
| Storage 2 | Hikvision C2000 Pro 1TB | Used for Data Store |
| PSU | ABKO SETTLER 600W | 80 Plus Standard |
| Wi-Fi Card | Intel Wi-Fi AX201 | Needs AirportItlwm.kext and Patching |

**Legend:**
- ✅ - Fully functional
- 🟡 - Partially functional
- 🟠 - Not tested yet

## Features
| Function | Status | Comments |
| :-: | :-: | :-: |
| USB | ✅ | See USB mapping section |
| Ethernet | ✅ | Using Apple's I225LM driver |
| Wi-Fi | ✅ | Needs AirportItlwm.kext and Patching |
| Bluetooth | ✅ | Works natively |
| AirDrop, Handoff, Universal Clipboard | 🟡 | Airdrop works only single-direction |
| Hardware Acceleration | ✅ | Using AMD Hardware Acceleration, see Hardware Acceleration section |
| DRM | 🟡 | Not working on YouTube Movies, Only some Netflix videos works |
| Sleep | ✅ | You can use Keyboard or Mouse to wake up |
| USB-C on dGPU | ✅ | For data transmission, XHCI-AMD6800.kext is needed, see USB-C section |
| Universal Control | 🟠 | Not tested yet |

## BIOS
Few things need to be taken care of in the BIOS.

| Setting | Value|
| :-: | :-: |
| Fast Boot | Disable |
| Secure Boot | Disable |
| VT-d | Disable |
| CSM | Disable |
| Intel SGX | Disable |
| VT-x | Enable |
| Above 4G decoding | Enable |
| Hyper-Threading | Enable |
| XHCI Hand-off | Enable |
| DVMT Pre-Allocated | 64MB |
| SATA Mode | AHCI |
| iGPU Multi-Monitor | Enable |

Note: In the newer version of BIOS, when enabling **Above 4G decoding**, you may enable **Re-size BAR Support** if your hardware supports it. However, you need to make some changes to the config.plist, check out the Re-size BAR section. Make sure you connect the monitor to the dGPU.

## Hardware Acceleration and DRM

If you do not set hardware acceleration correctly, DRM-related content will not play. Since we're using a discrete AMD GPU here, we can make use of AMD's own hardware acceleration coder.

In the terminal, type the following commands one by one.

`defaults write com.apple.AppleGVA gvaForceAMDKE -bool YES`

`defaults write com.apple.AppleGVA gvaForceAMDAVCEncode -bool YES`

`defaults write com.apple.AppleGVA gvaForceAMDAVCDecode -bool YES`

`defaults write com.apple.AppleGVA gvaForceAMDHEVCDecode -bool YES`

These should make hardware acceleration work correctly and have no problem playing back DRM content.

## Re-size BAR

With the supported hardware, you may be able to enable re-size BAR in your BIOS under the PCI subsystem configuration tab in the BIOS.

macOS has no support for re-size BAR but starting from OpenCore 0.7.5, two quirks can be used to config it so that you can enjoy re-size BAR in Windows while not breaking macOS.

`Booter >> Quirks >> ResizeAppleGpuBars: reduces GPU PCI BAR size to be compatible with macOS.`

`UEFI >> Quirks >> ResizeGpuBars: configure the GPU PCI BAR size for systems other than macOS.`

For `ResizeAppleGpuBars`, you want to set it to `0` if you **enable** re-size BAR and `-1` if **disable** it. Other values may break macOS as discussed [here](https://www.insanelymac.com/forum/topic/349485-how-to-opencore-074-075-differences/).

For `ResizeGpuBars`, it depends on your GPU's memory. Set it to `n` where `n` is the minimal integer that makes `2^n MB` greater than or equal to the video memory you have.

For example, if I have 16GB video memory, then I should set it to `14` since `2^14 = 16384MB` which is basically 16GB.

## USB-C on RX 6800

By default, the USB-C port on RX 6800 only has the functionality of video output and power delivery due to the fact that macOS loads a wrong kext called **AppleAMDUSBXHCIPCI.kext**.

We'll use a custom kext based on XCHI-unsupported.kext called **XHCI-AMD6800.kext** to force macOS loads the correct kext **AppleUSBXHCIPCI.kext**.


## Modify config.plist in the OC Folder
This repository contains EFI based on OpenCore 1.0.2. If you're using the same mobo, then this EFI is likely working for you. But if you have different parts other than mobo, please read the following content and modify it accrodingly.

You need [ProperTree](https://github.com/corpnewt/ProperTree) to open and edit config.plist.

### DeviceProperties
Since I have both iGPU and dGPU, I set it as output by dGPU and iGPU is only used for hardware accleration.

If you wish to use the iGPU as output, you'll need to change **AAPL,ig-platform-id** from **0300C89B** to **00009B3E**.

`PciRoot(0x0)/Pci(0x1C,0x4)/Pci(0x0,0x0)` is used for tricking Apple's I225LM driver into supporting our I225-V network controller. If this does not work on your mobo, try `PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)`.

### NVRAM

For Navi users(RX 5000/6000 series), you need to add **agdpmod=pikera**.


If you hate all the debug info when booting, you can also remove **-v** parameter but I suggest you remove it only when your hacking is up and running fine.

### PlatformInfo
I left this part blank intentionally because you really need your own serial number.

To create a new serial number, you can use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)

As for SMBIOS, it depends on your CPU:

| SMBIOS | CPU |
| :-: | :-: |
| iMac20,1 | i7-10700K and lower (8 core and lower) |
| iMac20,2 | i9-10850K and higher (10 core) |
| MacPro7,1 | F series with dGPU Only |
| iMacPro1,1 | F series with dGPU Only (Recommended if you don't want annoying notification **memory module misconfigured**) |

Using GenSMBIOS, you'll get **Type** , **Serial** , **Board Serial** and **SmUUID**

**Type** goes to Generic -> SystemProductName

**Serial** goes to Generic -> SystemSerialNumber

**Board Serial** goes to Generic -> MLB

**SmUUID** goes to Generic -> SystemUUID

**TL;DR** You need your own serial number and if you have a different CPU or want to use iGPU as output, you need to change a few things in config.plist

## Screenshots and Benchmark (Tested on macOS 12.0.1)

**Hardware Info**

![Hardware](Images/Sensei.png)

**Geekbench 5**

CPU Scores (M1 Max is around 1770 single and 12600 multi)

![CPU](Images/CPU.png)

GPU Scores (Unstable due to the short benchmark period, can change from time to time)

![GPU1](Images/GPU1.png)

![GPU2](Images/GPU2.png)

Blackmagic Raw Speed Test

![RAW](Images/BlackmagicRaw.png)

