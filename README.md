# ASUS X299 Hackintosh

# Introduction
The ASUS X299 Hackintosh repository contains OpenCore EFI distributions and related files that can be used as a reference when setting up your X299 Hackintosh with the OpenCore bootloader.  While the EFIs can be used as a starting point for all ASUS X299 boards, it is still highly recommended to review the [OpenCore Vanilla Desktop Guide](https://dortania.github.io/OpenCore-Install-Guide/) and [Skylake-X section](https://dortania.github.io/OpenCore-Install-Guide/config-HEDT/skylake-x.html) for a proper guide.

# Repository Folders
| Folder | Description |
| :------------- | :---------- |
| BASE-EFI | OpenCore EFI precompiled with the OpenCanary GUI enabled that should be valid for all ASUS X299 boards. |
| Custom BIOS Collection | Contains modified BIOS files that have custom boot logos |
| Personal EFI Collection | Contains personal custom EFIs for various boards. |

# Personal Build
Personal EFI and USB mapping can be found [here](https://github.com/shinoki7/ASUS-X299-Hackintosh/tree/main/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G).

![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/WSX299Sage10G.png)

## Specifications
### Components

| Component        | Model                                | Notes |
| ---------------- | ---------------------------------------|-------------------|
| Motherboard | ASUS WS X299 Sage/10G | BIOS 3302 |
| Processor | Intel i9-10980XE | |
| CPU Cooler | Fractal Design Celsius+ S36 Dynamic | |
| RAM | 4x16 Corsair Vengeance LPX 3200 Mhz | |
| Boot Drive | Samsung 970 EVO 1 TB | |
| Graphics Card | AMD Radeon Pro W5500 | |
| Wifi/Bluetooth Card | Broadcom BCM943602CDP |  |
| Power Supply | Corsair RM 850x | |
| Case | Lian Li PC 011 Dynamic | |

### PCIe Slot Layout
| Slot | Speed | Device | Notes |
| ----- | ----- | ---------------------------------------|-------------------|
| 1 | x16 | | |
| 2 | x8 | ASUS ThunderboltEX 4 | Downgraded to BIOS 3302, causes error code 62 on BIOS 3405 |
| 3 | x8 | | |
| 4 | x8 | | |
| 5 | x8 | AMD Radeon Pro W5500 | |
| 6 | x8 | | |
| 7 | x8 | Broadcom BCM943602CDP | |

### M.2/U.2 Layout
| Slot | Device | Notes |
| ----- | ---------------------------------------|-------------------|
| U.2_1 | | |
| M.2_1 | | |
| M.2_2 | Samsung 970 EVO 1 TB | |

## What Works / What Doesn't Work
- [x] Sleep / Wake
- [x] Wifi and Bluetooth
- [x] Handoff, Continuity, AirDrop, Continuity Camera, and Unlock with Apple Watch
- [x] iMessage, FaceTime, App Store, iTunes Store
- [x] 10G Ethernet
    * Modified EEPROM to work with SmallTree drivers. Refer to section [Intel 10 Gigabit NICs with Small Tree macOS Drivers](https://github.com/shinoki7/ASUS-X299-Hackintosh#intel-10-gigabit-nics-with-small-tree-macos-drivers) for more info.
- [x] HEVC, H.264
- [x] Onboard audio
- [x] TRIM
- [x] USB 2.0
- [x] USB 3.2 Gen 1
- [x] USB 3.2 Gen 2
- [x] DRM
- [x] Native NVRAM
- [x] CPU Power Management
- [x] USB Power
- [x] Thunderbolt 3/4 Hot plug
- [ ] SideCar
    * Due to some T2 chip dependancies on MacPro7,1 and iMacPro1,1 SMBIOS

## Radeon Pro W5500 Issues
Replaced RX 580 with a Radeon Pro W5500 and have encountered a couple issues.  The major issue appears to be an issue with AGPM.  Not sure if it's specific to the W5500 or just have an issue with my GPU so YMMV when using W5500.

### 1. Kernel Panics with Green/Black screen
macOS will randomly kernel panic and freeze (sometimes once a day, sometimes multiple) on a green screen (HDMI) or black screen (DP).  The computer will either stay on a green/black screen until manually rebooting or macOS will reboot automatically with a kp log.

Things I've tried:
* Various versions of macOS Big Sur
* Different SMBIOS (iMacPro1,1, MacPro7,1)
* Different Ram/XMP settings
* Enable/Disable Whatevergreen + boot-arg agdpmod=pikera
* Enable/Disable AGPMInjector.kext

### 2. Hit or miss DP/HDMI audio
DP/HDMI audio sometimes does not work and audio selection is missing from macOS Sound Preferences on consecutive reboots.  Not sure if this is due to using DP to HDMI adapters as I haven't tried testing with just DP cables attached.

### Workarounds
After some troubleshooting, I found that keeping a video open in the background seems to prevent kernel panics.  The video does not have to be continuously playing so I just had a small video clip set to open on login.

The better workaround I found was to do a device spoof to a 5500XT with WEG+agdpmod=pikera flags.  WEG+agdpmod=pikera isn't required on SMBIOS MacPro7,1 but have it enabled just in case.

| Key | Class | Value |
| :--- | :--- | :--- |
| compatible | String | pci1002,7341 |
| device-id | Data | 40730000 |
| agdpmod | String | Pikera |

Also added the Radeon boost SSDT located [here](https://www.tonymacx86.com/threads/amd-radeon-performance-enhanced-ssdt.296555/).  With device spoofing, DP/HDMI audio is more stable as well although the monitor/tv has to be on at boot for audio to work (at least with DP to HDMI adapters, haven't tested DP to DP connections).  This isn't really an issue for me since I only have audio connected to my main monitor.

## ASUS Thunderbolt EX4 (Thunderbolt 4) (WIP)
For Thunderbolt 4, currently the only available Thunderbolt 4 PCIe card is the ASUS ThunderboltEX 4.  

### Current Observations
Note these observations are based on the devices tested listed below.  YMMV and issues I'm running into may just be due to these devices.

* Initially ran into issues where card was causing kernel panics in macOS and error code 62 on reboots.  Found this issue on the latest BIOS with resizable bar support.  Downgraded BIOS to 3302 and the card works normally.
* Thunderbolt header may not necessarily be needed or jumped.  macOS recognizes the card on boot without a header plugged in.
* The 6 pin power and USB 2.0 internal cable have to be connected.  
* Thunderbolt settings have to be enabled in BIOS.  The device loaded in a different slot with Thunderbolt disabled but it showed 'No driver installed'.
* SSDT is required for device to be recognized.

### What Works
* Thunderbolt 3 hot-plug
* USB 2.0 hot-plug/cold-plug
* USB 3.1 Gen 2 hot-plug/cold-plug
* Sleep

### What Doesn't Work
* Thunderbolt 3 cold/warm boot

### Devices Tested:
* Sabrent Thunderbolt 3 M.2 NVMe SSD Enclosure (EC-T3NS)

### To-Do:
* Extract original firmware and try installing custom firrmware for Thunderbolt Local Node support.
* Try more thunderbolt 3/4 devices.

## Screenshots
### System Report
![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/aboutthismac.png)
![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/memory1.png)
![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/memory2.png)
![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/graphics.png)
![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/pci.png)

### USB Mapping
`USBMap-WS X299 Sage 10G.zip` contains a full USB Mapping kext for the XHCI controller and both ASMedia USB 3.2 Gen 2 Controllers.

![](/Personal%20EFI%20Collection/WS%20X299%20Sage%2010G/Images/usbmapping.png)

# Optional USB Peripherals for Internal USB 2.0 Devices
* 1. [USB 3.0 20 Pin Female to USB 2.0 Pin Male Adapter](https://www.amazon.com/gp/product/B01MFB04JP/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
    * This adapter converts the internal USB 3.0 19 pin header to a USB 2.0 9 pin.
* 2. [USB 2.0 9 Pin Header 1 to 4 Extension Hub Splitter](https://www.amazon.com/gp/product/B085KVH16T/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) to connect Bluetooth.
* 3. [USB 2.0 IDC 5 Male to USB A Male adapter](https://www.amazon.com/gp/product/B000V6WD8A/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
    * Uses one of the USB ports on the back of the motherboard to connect internal devices in the case.  You can get two of these and remove 1 of the pins to make a "9 pin internal adapter".  If the cable is too short from the back to inside the case, you can get some regular USB A extension cables.
* 4. PCIe USB 3.0 Card with Internal USB Connector
    * [Inateck USB 3.0 Card](https://www.amazon.com/Inateck-Express-Controller-Internal-Connector/dp/B00JFR2H64/ref=sr_1_3?dchild=1&keywords=inateck+pcie+card&qid=1592455853&s=electronics&sr=1-3)
    * [StarTech USB 3.1 PCIe Card](https://www.amazon.com/gp/product/B01I39D15A/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
    * Can use the internal header on the card for the case USB ports or to connect internal devices.
    * NOTE: Wake from Bluetooth devices does not work with this so it's best to connect Bluetooth to one of the motherboard ports.

# Intel 10 Gigabit NICs with Small Tree macOS Drivers
Intel 10 gigabit NICs such as the X540 or X550 (found on the WS X299 Sage/10G) do not have official support in macOS.  However with modding the EEPROM, we can modify the Subsystem ID to 000a to be compatible with the SmallTree macOS Drivers.  Original method by Squuiid outlined in this [MacRumors thread](https://forums.macrumors.com/threads/modify-retail-intel-10gbe-nics-to-use-small-tree-macos-drivers.1968456/).  

**[The instructions below are outlined for the X550-AT2 found in the WS X299 Sage/10G].  Other Intel NICs may have different commands and/or offsets.  [Proceed at your own risk!]**

**[Before EEPROM modding: No Driver Installed]**
![](/Resources/Images/eeprombefore.png)

**[After EEPROM modding and SmallTree drivers: Driver Installed]**
![](/Resources/Images/eepromafter.png)

## Requirements
1. [Ubuntu 16.04 LTS](https://releases.ubuntu.com/16.04/)
    * Download the 64-bit PC (AMD64) desktop image
    * Newer versions of Ubuntu do not work.
2. USB Flash drive 8GB+
3. Windows install for installing Ubuntu to USB drive using [Rufus](https://rufus.ie)
4. Ethernet Drivers (SmallTree is recommended but Sonnet also works)
    * [SmallTree macOS drivers](https://small-tree.com/support/downloads/10-gigabit-ethernet-driver-download-page/)
        * Download the macOS Catalina drivers for macOS Big Sur
    * [Sonnet macOS drivers](http://www.sonnettech.com/support/kb/kb.php?cat=513&expand=&action=a3#a3)

## Instructions
1. Install the Ubuntu 16.04 LTS ISO image to a flash drive using Rufus.
2. Boot Ubuntu and click "Try Ubuntu without Installing".
    * Once Ubuntu is loaded, make sure that you have an Internet connection.
3. Open Terminal and enter in the following commands to install "net-tools" and "ethtool"
    * `sudo apt-get install net-tools`
    * `sudo apt-get install ethtool`
4. To locate the address of your ethernet devices, enter in command: `ifconfig`.  In my case my X550 ports were `enp180s0f0` and `enp180s0f1`.
![](/Resources/Images/ifconfig.png)
5. To locate the current Vendor/Device-ID of your ethernet devices, enter in command `lspci -nn -vvv | grep Ethernet`.
![](/Resources/Images/lspcigrep.png)
6. Using command `sudo ethtool -e enp180s0f0 | less` , we can search for the offset of the Subsytem Vendor/Subsystem ID (X550 is offset 0x240 - 0x243).
![](/Resources/Images/beforeoffset.png)
7. For SmallTree driver support, we will modify the Subsytem-ID from `8712` to `000a`.  
For the X550-AT2, the commands are:
    * `sudo ethtool -E enp180s0f0 magic 0x15638086 offset 0x242 value 0x0a`
    *  `sudo ethtool -E enp180s0f0 magic 0x15638086 offset 0x243 value 0x00`

    * `sudo ethtool -E enp180s0f1 magic 0x15638086 offset 0x242 value 0x0a`
    * `sudo ethtool -E enp180s0f1 magic 0x15638086 offset 0x243 value 0x00`
![](/Resources/Images/afteroffset.png)
8. For Sonnet driver support, we will modify the Subsystem-Vendor-ID from `1043` to `16b8` and the Subsystem ID from `8712` to `7212`.  
For the X550-AT2, the commands are:
    * `sudo ethtool -E enp26s0f0 magic 0x15638086 offset 0x240 value 0xb8`
    * `sudo ethtool -E enp26s0f0 magic 0x15638086 offset 0x241 value 0x16`
    * `sudo ethtool -E enp26s0f0 magic 0x15638086 offset 0x242 value 0x12`
    * `sudo ethtool -E enp26s0f0 magic 0x15638086 offset 0x243 value 0x72`

    * `sudo ethtool -E enp26s0f1 magic 0x15638086 offset 0x240 value 0xb8`
    * `sudo ethtool -E enp26s0f1 magic 0x15638086 offset 0x241 value 0x16`
    * `sudo ethtool -E enp26s0f1 magic 0x15638086 offset 0x242 value 0x12`
    * `sudo ethtool -E enp26s0f1 magic 0x15638086 offset 0x243 value 0x72`
    * Note if you want to revert back to SmallTree or Intel drivers, we will modify the Subsystem-Vendor-ID back to `1043`.  For the X550-AT2, the commands are:
        * `sudo ethtool -E enp180s0f0 magic 0x15638086 offset 0x240 value 0x43`
        * `sudo ethtool -E enp180s0f0 magic 0x15638086 offset 0x241 value 0x10`

        * `sudo ethtool -E enp180s0f1 magic 0x15638086 offset 0x240 value 0x43`
        * `sudo ethtool -E enp180s0f1 magic 0x15638086 offset 0x241 value 0x10`
9.  Reboot back into macOS and install the appropriate drivers and your ethernet ports should be enabled!

# Additional Kexts
* [SmallTreeIntel82576](https://github.com/khronokernel/SmallTree-I211-AT-patch/releases)
  * Enables ethernet for I211 NICs
  * Version 1.3 is for macOS Catalina, Version 1.2.5 is for macOS 10.13 and 10.14
* [AGPMInjector](https://github.com/Pavo-IM/AGPMInjector)
  * Apple Graphics Power Management injector

# Credits
* [Apple](https://www.apple.com) : macOS
* [Acidanthera](https://github.com/acidanthera) : OpencorePkg, kexts, etc.
* [Dortania](https://github.com/dortania) : Opencore guide
* [Pavo](https://github.com/Pavo-IM) : AGPMInjector
