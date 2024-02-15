# ROG Zephyrus G14 (GA402) Hackintosh [Archived]
Hackintosh for ASUS ROG Zephyrus G14 (GA402) 2022.

### ** 2024-02-1x : IMPORTANT NOTICE **
I no longer own a G14 so this repository will not be updated. I have updated the repo to be as up to date as possible at this time.  
I'll be happy if someone fork this repo and take my work over.

<img width="1813" alt="Screen Shot 2022-07-02 at 15 50 09" src="https://user-images.githubusercontent.com/2325377/176993855-832f4ff0-b2c8-41f3-88fc-7c35c2f7bd3b.png">

Currently not all functions and components work properly, but many important components like CPU and GPU working.

## Requirements
- Download latest [NootRX.kext](https://github.com/ChefKissInc/NootRX) from [here](https://github.com/ChefKissInc/NootRX/actions/workflows/main.yml)
- Set your own `MLB` / `ROM` / `SystemSerialNumber` / `SystemUUID` to config.plist

## Specifications
### Hardware
- Model : GA402RJ
  - GA402RK ( RX 6800S SKU ) should work too
- CPU : AMD Ryzen 7 6800HS 3.2 GHz ( Rembrandt )
- ( iGPU : AMD Radeon 680 )
  - no way to make it work
- dGPU : AMD Radeon RX 6700S 8 GB
  - It is named 6700, but it has same Navi 23 chip as the desktop 6600 series.
  - Must be set primary by MUX switch setting ("Ultimate" mode) with Armoury Crate app on Windows.
- Memory : DDR5-4800MHz 24 GB ( 8 GB onboard + 16 GB SO-DIMM )
- WiFi : Intel AX210 WiFi 6 ( replaced from original MediaTek M.2 2230 card )

### Software
- macOS Sonoma
- macOS Ventura
- macOS Monterey
- OpenCore 0.9.8
- [BIOS 312](https://web.archive.org/web/20220928185508/https://rog.asus.com/laptops/rog-zephyrus/rog-zephyrus-g14-2022-series/helpdesk_bios/)

## Current status
### Working
- CPU with 8 cores and 16 threads enabled
  - CPU power management with [AMD Power Gadget](https://github.com/trulyspinach/SMCAMDProcessor) and [RyzenAdj](https://gist.github.com/newperson1746/00cccb51aa2fe5e31e8e2f577bec3a88)
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
  - another reference : https://github.com/MotorBottle/S3-Sleep-on-Rog-X13-G14-15-2021-2022-using-OpenCore

### Not working / Problems
- BIOS version 315 or newer
  - The system became very unstable after updating to BIOS 315, I have to stay on BIOS 312.
- Radeon 680 iGPU
  - Probably never works, that's why I bought the laptop with dGPU.
  - Left USB-C DP alt doesn't work because of it's controlled by iGPU.
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

#### Kexts
- NootRX.kext
  - to get RX 6700S working.
- RadeonSensor.kext / SMCRadeonGPU.kext
  - to get dGPU temperature.
- RestrictEvents.kext
  - to change CPU name on About This Mac.
- VoodooI2C.kext / VoodooGPIO.kext
  - build from latest master

#### NVRAM
- `boot-args`
  - `debug=0x44` : to use ryzenadj
- `ExposeSensitiveData` : `15`
  - to show machine name in AMD Power Gadget.
- csr-active-config : `01000000`
  - to use ryzenadj
