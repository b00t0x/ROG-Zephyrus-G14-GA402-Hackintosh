# ROG Zephyrus G14 (GA402) Hackintosh [WIP]
Hackintosh for ASUS ROG Zephyrus G14 (GA402) 2022.

<img width="1813" alt="Screen Shot 2022-07-02 at 15 50 09" src="https://user-images.githubusercontent.com/2325377/176993855-832f4ff0-b2c8-41f3-88fc-7c35c2f7bd3b.png">

Currently not all functions and components work properly, but many important components like CPU and GPU working.

## Specifications
### Hardware
- Model : GA402RJ
  - GA402RK ( RX 6800S SKU ) should work too
- CPU : AMD Ryzen 7 6800HS 3.2 GHz ( Rembrandt )
- ( iGPU : AMD Radeon 680 )
  - no way to make it work
- dGPU : AMD Radeon RX 6700S 8 GB
  - It is named 6700, but it has same Navi 23 chip as the desktop 6600 series.
  - Must be set primary by MUX switch setting with  Armoury Crate app on Windows.
- Memory : DDR5-4800MHz 24 GB ( 8 GB onboard + 16 GB SO-DIMM )
- WiFi : Intel AX210 WiFi 6 ( replaced from original MediaTek M.2 2230 card )

### Software
- macOS Sonoma 14.0 beta 1 (23A5257q)
- macOS Ventura 13.4 (22F66)
- macOS Monterey 12.6.8 (21G725)
- OpenCore 0.9.4

## Current status
### Working
- CPU with 8 cores and 16 threads enabled
  - CPU power management with [AMD Power Gadget](https://github.com/trulyspinach/SMCAMDProcessor) and [RyzenAdj](https://gist.github.com/b00t0x/c2b940a4a7a05c7169b54aa0a1be8cd3)
  - `ryzenadj` command is [attached](ryzenadj) to this repo.
- dGPU acceleration
  - 120Hz LCD
    - HiDPI custom resolution with [one-key-hidpi](https://github.com/xzhih/one-key-hidpi)
  - 4K / 60Hz output via HDMI / USB-C (Right) ports
- WiFi ( card replacing required )
- PCIe 4.0 NVMe SSD
- All USB ports
- Keyboard
- Internal speaker
- Battery status
- Integrated camera
- Trackpad
- Sleep
  - disable hybrid sleep and enable S3 by using [UMAF](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjh0rGV0sn_AhUdt1YBHa5-BpIQFnoECAkQAQ&url=https%3A%2F%2Fgithub.com%2FDavidS95%2FSmokeless_UMAF&usg=AOvVaw2O06EbRqc__MbbahWlp7-7&opi=89978449)

### Not working / Problems
- BIOS version 315 or newer
  - The system became very unstable after updating to BIOS 315, I have to stay on BIOS 312. **Warning**: If you're on 317+, **DO NOT** attempt to flash 312, you risk irreversibly bricking your device.
- Radeon 680 iGPU
  - Probably never works, that's why I bought the laptop with dGPU.
  - Left USB-C DP alt doesn't work because of it's controlled by iGPU.
- Hardware video decode / encode acceleration
  - VideoProc says no acceleration at all.
- Hardware LCD backlight brightness control
  - Only software control available with [MonitorControlLite](https://github.com/MonitorControl/MonitorControlLite).
- Keyboard backlight brightness control
  - Always on
- Keyboard Fn hotkeys
  - Only sound hotkeys work
- Longer battery life
  - Even in dGPU mode, Windows should run for 3 hours but it only runs for around an hour and a half in macOS.
  - In iGPU mode, Windows should run for 10 hours, but iGPU doesn't work in macOS.

## Configurations
### config.plist
#### DevirtualiseMmio / MmioWhitelist
In spite of the laptop doesn't have TRX40 chipset, DevirtualiseMmio quirk is required to avoid stuck on `[EB|#LOG:EXITBS:START]`. MmioWhitelist is also required to avoid blackout issue.

#### dGPU DeviceProperties

- `device-id` : `FF730000`
  - spoofed as RX 6600 series.
- `no-gfx-spoof` : `True`
  - required to use WhateverGreen.kext without blackout issue.
- `agdpmod` : `pikera`
  - required to use WhateverGreen.kext without blackout issue.
- `@0,name` : `ATY,Henbury`
  -  must to be set to enable GPU accelaration.

#### Kexts
- DirectHW.kext
  - required to use `ryzenadj` command. read [this doc](https://gist.github.com/b00t0x/c2b940a4a7a05c7169b54aa0a1be8cd3) for detail.
- RadeonSensor.kext / SMCRadeonGPU.kext
  - to get dGPU temperature.
- RestrictEvents.kext
  - to change CPU name on About This Mac.
- VoodooI2C.kext / VoodooGPIO.kext
  - build from latest master

#### NVRAM
- `boot-args`
  - `debug=0x44` : to enable DirectHW.kext
- `ExposeSensitiveData` : `15`
  - to show machine name in AMD Power Gadget.
- `csr-active-config` : `01000000`
  - to load DirectHW.kext

#### PlatformInfo
`MacPro7,1` smbios is required to enable internal display. ( `iMacPro1,1` doesn't work ) I don't know why.

You should set your own `MLB` / `ROM` / `SystemSerialNumber` / `SystemUUID` .
