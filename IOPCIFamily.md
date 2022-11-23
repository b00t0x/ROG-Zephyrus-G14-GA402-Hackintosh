# Replacing IOPCIFamily.kext

**NOTE : This workaround is no longer required after applying the [probeBusGated patch](https://github.com/AMD-OSX/AMD_Vanilla/compare/f0cf7827578216047325220784a469c77e8e7b98...3be0cb6c4a6651e8dde9026c6de637473eac24d6) ( [d0fce4d](https://github.com/b00t0x/ROG-Zephyrus-G14-GA402-Hackintosh/commit/d0fce4deb7d7cb807e3494c9577be4e505800fbb) ).**

## Problems
macOS Monterey has many problems with GA402 because vanilla IOPCIFamily.kext is not working well.
- Random boot failure
- Slow boot time
  - Stuck long time at `PCI configuration begin`
- GPU acceleration not working
  - Only framebuffer works
- Mixed PCIe Gen configuration not working
  - M.2 2280 slot is running on Gen4, doesn't work with Gen3 SSD.
    - Only Gen4 SSDs works
    - SATA SSDs will not work because SATA is not wired to the slot.
  - M.2 2230 slot is running on Gen3, doesn't work with any WiFi card because there is no Gen3 WiFi card on the market.
    - For example, DW1560 / 8265NGW are Gen1 cards, and even AX200 / AX210 are Gen2 cards.

Big Sur and older don't have these problems, but macOS before Monterey can't recognize Navi 23 cards, so using them is not the option.

## Solution
Replacing IOPCIFamily.kext and AppleACPIPlatform.kext with Big Sur's one. AppleACPIPlatform.kext requires the same version IOPCIFamily.kext, so both are needed to replace.

Replacing kexts is only available after installation, so using Gen4 SSD and disabling dGPU DeviceProperties to install Monterey, or using another machine to prepare that is Monterey installed SSD.

### Extract kexts from Big Sur
Retrieving kexts from /S/L/E on Big Sur system is the easiest way, but if you don't have a Big Sur system, you can extract them from the installer.

1. Create Big Sur installer with [macadmin-scripts](https://github.com/munki/macadmin-scripts)  
( I used the 11.6.7 installer )
2. Mount the dmg
3. open `Install macOS Big Sur.app/Contents/SharedSupport/SharedSupport.dmg` with [Pacifist](https://www.charlessoft.com/)
4. Find kexts in /S/L/E of dmg
5. Right click and select `Extract to Custom Location...` ( Admin privileges not needed )

### Install kexts
Installing kexts to /S/L/E is a bit complicated.

Before doing these steps, you must [disable SIP](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/post-issues.html#disabling-sip).  
Following [this guide](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/post-issues.html#writing-to-the-macos-system-partition) to mount system partition then copy kexts.

```
mkdir ~/livemount

# identify diskXsY by diskutil command
sudo mount -o nobrowse -t apfs  /dev/diskXsY ~/livemount
cd ~/livemount/System/Library/Extensions

# replace kexts
sudo rm -rf IOPCIFamily.kext AppleACPIPlatform.kext
sudo cp -r /path/to/bigsur/IOPCIFamily.kext /path/to/bigsur/AppleACPIPlatform.kext ./
sudo chmod -R 755 IOPCIFamily.kext AppleACPIPlatform.kext
sudo chown -R 0:0 IOPCIFamily.kext AppleACPIPlatform.kext

# create cache and snapshot
sudo kmutil install --volume-root ~/livemount --update-all
sudo bless --folder ~/livemount/System/Library/CoreServices --bootefi --create-snapshot
```
Then reboot the laptop.

Finally you can uncomment the `DeviceProperties` on config.plist.
