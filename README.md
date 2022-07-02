# ROG Zephyrus G14 (GA402) Hackintosh [WIP]
Hackintosh for ASUS ROG Zephyrus G14 (GA402) 2022.

<img width="1367" alt="Screen Shot 2022-07-02 at 2 13 00" src="https://user-images.githubusercontent.com/2325377/176945472-71390d21-e3cc-4382-b539-67a288fc7dee.png">

Currently not all functions and components work properly ( especially trackpad ), but many important components like CPU and GPU working.

## Specifications
### Hardware
- Model : GA402RJ
  - GA402RK ( RX 6800S SKU ) should work too
- CPU : AMD Ryzen 7 6800HS 3.2 GHz ( Rembrandt )
- ( iGPU : AMD Radeon 680 )
  - no way to make it work
- dGPU : AMD Radeon RX 6700S 8 GB
  - It is named 6700, but it has same Navi 23 chip as the desktop 6600 series.
- Memory : DDR5-4800MHz 16 GB ( 8 GB onboard + 8 GB SO-DIMM )
- WiFi : Intel AX210 WiFi 6 ( replaced from original MediaTek M.2 2230 card )

### Software
- macOS Monterey 12.4 (21F79)
- OpenCore 0.8.1

## Current status
### Working
- CPU with 8 cores and 16 threads enabled
  - CPU power management with [AMD Power Gadget](https://github.com/trulyspinach/SMCAMDProcessor) and [RyzenAdj](https://gist.github.com/b00t0x/c2b940a4a7a05c7169b54aa0a1be8cd3)
  - `ryzenadj` command [attached](ryzenadj) to this repo.
- dGPU acceleration
  - 120Hz LCD
    - HiDPI custom resolution with [one-key-hidpi](https://github.com/xzhih/one-key-hidpi)
  - 4K / 60Hz output via HDMI / USB-C (Right) ports
- WiFi ( card replacing required )
- PCIe 4.0 NVMe SSD
- All USB ports
- Keyboard
- Internal speaker
  - Sound quality is too bad
- Integrated camera

### Not working / Problems
- Radeon 680 iGPU
  - Probably never works, that's why I bought the laptop with dGPU.
  - Left USB-C DP alt doesn't work because of it's controlled by iGPU.
- Trackpad
  - I2C controlled, no idea to enable AMD I2C device.
- Sleep / Hibernation
- Hardware video decode / encode acceleration
  - VideoProc says no acceleration at all.
- Hardware LCD backlight brightness control
  - Only software control by modded [MonitorControl](https://github.com/MonitorControl) app.
- Keyboard backlight brightness control
  - Always on
- Keyboard Fn hotkeys
  - Only sound hotkeys work
- macOS Ventura beta
  - Stuck at `ACPI: sleep states` even if OpenCore nightly using.
- vanilla IOPCIFamily.kext / AppleACPIPlatform.kext
  - Must be replaced with Big Sur's one.
  - Without replacing, PCIe gen mixed SSD / WiFi not working.
- Longer battery life
  - Even in dGPU mode, Windows should run for 3 hours but it only runs for around an hour and a half in macOS.
  - In iGPU mode, Windows should run for 10 hours, but iGPU doesn't work in macOS.

## Configurations
### config.plist
#### APIC.aml
Modified APIC.aml is required to avoid early reboot. The `Processor ID` in this file has changed from 1 origin to 0 origin. Without this fix, the CPU topology will be corrupted.

Deleting original APIC.aml also required.

#### DevirtualiseMmio / MmioWhitelist
In spite of the laptop doesn't have TRX40 chipset, DevirtualiseMmio quirk is required to avoid stuck on `[EB|#LOG:EXITBS:START]`. MmioWhitelist is also required to avoid blackout issue.

### dGPU DeviceProperties
These properties are commented out because of not working before replacing IOPCIFamily.kext. To enable this, read [IOPCIFamily.md](IOPCIFamily.md).

- `device-id` : `FF730000`
  - spoofed as RX 6600 series.
- `no-gfx-spoof` : `True`
  - required to use WhateverGreen.kext without blackout issue.
- `agdpmod` : `pikera`
  - required to use WhateverGreen.kext without blackout issue.
- `@0,name` : `ATY,Henbury`
  -  must to be set to enable GPU accelaration.

#### Kexts
- AGPMInjector.kext
  - to attach `AGPM` to Framebuffer
- DirectHW.kext
  - required to use `ryzenadj` command. read [this doc](https://gist.github.com/b00t0x/c2b940a4a7a05c7169b54aa0a1be8cd3) for detail.
- itlwm.kext
  - due to replacing IOPCIFamily.kext, AirportItlwm not working.
- RadeonSensor.kext
  - to get dGPU temperature.
- RestrictEvents.kext / SMCRadeonGPU.kext
  - to change CPU name on About This Mac.

#### NVRAM
- `boot-args`
  - `debug=0x44` : to enable DirectHW.kext
- `ExposeSensitiveData` : `15`
  - to show machine name in AMD Power Gadget.

#### boot-args
- `csr-active-config` : `FF0F0000`
  - due to replacing IOPCIFamily.kext, SIP must be disabled.

#### PlatformInfo
`MacPro7,1` smbios is required to enable internal display. ( `iMacPro1,1` doesn't work ) I don't know why.

You should set your own `MLB` / `ROM` / `SystemSerialNumber` / `SystemUUID` .

### Brightness control
Currently hardware brightness control can't be enabled, so I'm using software gamma control by [MonitorControl](https://github.com/MonitorControl) app.

But MonitorControl assumes internal display should be hardware controlled, so I have to [patch](MonitorControl.diff) the app to disable internal display detection to the display is recognized as external.
