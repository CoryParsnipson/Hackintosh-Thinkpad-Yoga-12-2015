# Hackintosh-Thinkpad-Yoga-12-2015

This repository contains files and instructions for installing Hackintosh on a Lenovo Thinkpad Yoga 12 Gen 2 (from 2015).

## Software

* Mac OS 12.6.2 (Monterey)
* OpenCore 0.8.8

## Files

* installation-usb-files
  * EFI -> efi folder snapshot used to boot into the installer. 

* host-files
  * EFI -> efi folder of host computer. Contains "final" working revision of config.plist.

## Computer Specs

| Item     | Description                                                 |
| -------- | ----------------------------------------------------------- |
| Model    | Lenovo Thinkpad Yoga 12 Gen 2 (2015)                        |
| CPU      | Intel Core i7-5600U (Broadwell)                             |
| GPU      | Intel HD Graphics 5500 (Integrated GPU)                     |
| Storage  | Samsung 860 SSD QVO 2.5" SATA III 1TB                       |
| WLAN     | Intel Dual Band Wireless-AC 7265                            |
| Ethernet | None (onboard) / Thinkpad OneLink Pro Dock Gigabit Ethernet |
| Input    | PS2 trackpad, mouse, mousenub, Wacom Digitizer Pen          |

## What Works and What Doesn't

### Works

| Item                        | Notes                                                                              |
| --------------------------- | ---------------------------------------------------------------------------------- |
| Power management            |                                                                                    | 
| Battery                     |                                                                                    | 
| Sound                       | works oob, alcid=3                                                                 |
| Graphics                    | acceleration working, use platform-id 0x16160002                                   |
| laptop keyboard             | special Fn keys except brightness (use OS custom keyboard shortcuts as workaround) |
| laptop trackpad             |                                                                                    | 
| external usb kb             |                                                                                    |
| external usb mouse          |                                                                                    |
| trackpoint                  |                                                                                    |
| Digitizer                   | somewhat; recognizes pen as a mouse with one button.                               |
| OneLink Pro Docking Station | audio, ethernet, and usb hub work. External monitors do not work!                  |
| Sleep                       | no changes necessary                                                               |
| Integrated camera           |                                                                                    |
| Integrated microphone       |                                                                                    |
| Wireless                    |                                                                                    |
| Bluetooth                   |                                                                                    |

### Doesn't Work

| Item              | Notes                                                                                    |
| ----------------- | ---------------------------------------------------------------------------------------- |
| DRM               | DRM not supported on iGPU only systems (use Chrome instead of Safari)                    |
| External Monitors | DP/DVI port of OneLink Pro docking station unsupported. HDMI out of side probably works. |
| Sensors           | Brightness/ambient light, gyroscope not working                                          |
| SD card reader    | This might work, but I didn't look into it.                                              |
| Touchscreen       | Doesn't work at all.                                                                     |
| CFG unlock        | Couldn't unlock BIOS or dump image from update executable.                               |

## Guide

[Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)

The installation USB method was created with macOS offline installer method using a MacOS (Catalina) VM on Windows. The USB drive was 32 GB in capacity and supported only USB 2.0.

## Installation Media

The hackintosh live install USB was made on a 32GB PNY USB 2.0 drive. An image of MacOS Catalina was downloaded and installed on Windows 10 using VMware to create the live usb installer. The drive was formatted to use the mac filesystem (just follow the instructions for usb creation on macOS).

Previous attempts to create a USB install on windows and linux worked, but the hackintosh installation did not boot up. (I suspect the reason for this was an incorrect SSDT-EC file, but I have not gone back to confirm. I know for a fact that the aforementioned way using a Mac VM, so try that if all else fails.)

Note: In a VM, the operating system does not have graphics acceleration, so this process is painfully slow.

## Gathering Files

### USB map

This should be identical to the output given by the USBToolBox mapping tool. Pay attention to the "Assign" column for what values to use.

Also, see [here](https://github.com/kymodoke/MacOS-Thinkpad-Yoga-12#usb-ports) for another reference to this map.


| Port # | Type    | Assign       | Description                           |
| ------ | ------- | ------------ | ------------------------------------- |
| 1      | USB 2.0 | USB 3 Type A | Left external USB port                |
| 2      | USB 2.0 | USB 3 Type A | Right external USB port               |
| 3      | USB 2.0 | Internal     | OneLink Pro Dock port                 |
| 4      | USB 2.0 | Internal     | Bluetooth                             |
| 5      | USB 2.0 | Internal     | Synaptics Touch                       |
| 6      | USB 2.0 | Internal     | Integrated Camera                     |
| 7      | USB 2.0 |              | Not used                              |
| 8      | USB 2.0 | Internal     | ISD-V4 (tablet)                       |
| 9      | USB 2.0 |              | Not used                              |
| 10     | USB 2.0 |              | Not used                              |
| 11     | USB 2.0 |              | Not used                              |
| 12     | USB 2.0 | USB 3 Type A | Left external USB port 3.0 companion  |
| 13     | USB 2.0 | USB 3 Type A | Right external USB port 3.0 companion |
| 14     | USB 2.0 | Internal     | OneLink Pro Dock port 3.0 companion   |
| 15     | USB 2.0 |              | Not used                              |

### ACPI (SSDT) Files

Following the guide for Broadwell laptops, these files are needed:

* SSDT PLUG
* SSDT EC
* SSDT PNLF
* SSDT IRQ

Note, SSDT XOSI or SSDT GPIO are not needed if there are no I2C devices attached (like in this case).

#### Important Note for SSDT EC

The SSDTTime generated file was incorrect! This causes kernel boot hang after opencore hand-off (i.e. after `EXITBS:START`).

If SSDTTime detects that the embedded controller is at `\_SB.PCI0.LPCB.NVT6`, this is **incorrect**! Generate this file manually or copy the `.dst` output of SSDTTime and change all instances of `\_SB.PCI0.LPCB.NVT6` to `\_SB.PCI0.LPCB`.

> Make sure all ACPI entries have `Enabled -> True` under `ACPI > Add`. Don't forget to replace `ACPI -> Patch`.

### Device Properties

Use `0x16160002` (i.e. `02001616`) for `platform-id`. This is exactly the Intel HD 5500 graphics card.

The framebuffer memory patch needs to be added or it will hang on boot with `IOG Flags 0x3 (0x51)`.

| Key                      | Type | Value      |
| ------------------------ | ---- | ---------- |
| framebuffer-patch-enable | Data | <01000000> |
| framebuffer-stolenmem    | Data | <00003001> |
| framebuffer-fbmem        | Data | <00009000> |

### Platform Info

Use MacBookAir7,2 for the SMBIOS. This computer is very similar in specs to the Thinkpad Yoga 2015.

More details from kymodoke: [https://github.com/kymodoke/MacOS-Thinkpad-Yoga-12#2-smbios](https://github.com/kymodoke/MacOS-Thinkpad-Yoga-12#2-smbios)

## BIOS Settings

If a setting is not mentioned, it can be left at default value.

* Config -> USB -> USB 3.0 enabled
* Config -> CPU -> hyperthreading enabled
* Config -> Intel AMT -> disabled
* Security -> Security Chip -> disabled
* Security -> Virtualization -> Vt-d -> disabled
* Secure boot -> disabled
* Startup -> UEFI/Legacy Boot -> UEFI Only
* Startup -> UEFI/Legacy Boot -> CSM Support -> Yes
* Startup -> Boot Mode -> Diagnostics

Boot Mode can be changed back to Quick once everything is done. The rest of the settings must remain.

Make the USB stick higher than other boot loaders in the boot order settings. This can be reversed after the installation is complete.

## Post Install Tasks

Pretty much follow the opencore guide for this section. Some minor changes are necessary (detailed below).

### Move EFI from USB to hard drive

Follow the OpenCore guide. In this section, I describe the specifics of my EFI folder, because it's not a singular MacOS install.

In my case, I triple-booted mac with Windows 10 and Fedora. The EFI partition needs to be close to the beginning (first partition, if possible, but not necessary).

Mount the EFI partition and copy the USB installer contents into it. The EFI partition should look like this:

```
(EFI partition name)/
├─ mach_kernel
├─ System/
├─ EFI/
│  ├─ Boot/
│  │  ├─ bootx64.efi
│  ├─ Microsoft/
│  ├─ fedora/
│  ├─ mac/
│  │  ├─ BOOT/
│  │  ├─ OC/
```

Copy the usb contents (BOOT and OC folders) into a new folder under EFI/\<new folder name\>. Here I called it "mac". The `fedora` and `Microsoft` folders are there from the Fedora Linux and Windows 10 installs, respectively.

### Modify Grub to Chainload into OpenCore

Sometimes people prefer to have OpenCore load Grub or be the only bootloader. I wanted to keep Grub.

I booted into Fedora and added a new entry based on (this guide)[https://github.com/SayantanRC/URLs/blob/master/grub_to_opencore.md].

Open `/etc/grub.d/40_custom` and add a new entry:

```
menuentry 'Mac OS 12.6.2 (Monterey)' $menuentry_id_option 'macOS-efi' {
  savedefault
  insmod chain
  insmod part_gpt
  insmod fat
  set root = (hd2,gpt2)
  chainloader /efi/mac/OS/OpenCore.efi
  set root = (hd2,gpt2)/efi/mac
}
```

> NOTE: might take some trial and error to find what hd2,gpt2 should be on your system. This value should be the partition that the EFI folder is in (it is in sda2 for mine). Grub console tab completion might help discover this value.

> NOTE: where it says "mac", replace with what you put under EFI/\<new folder name\>.

Don't forget to rebuild grub config once the changes are done.

```
> su
> grub2-mkconfig --output=/boot/grub2/grub.cfg
```

#### Modify OpenCore for "Easy Boot"

1. In the boot picker, press `ctrl + enter` on the entry you want to set as default. (Choose mac and not windows).
1. Set `Misc > ShowPicker` to `False` in the config plist to disable the boot menu. (uses default entry)

Once this is done, after you chainload from grub, Opencore will skip the boot menu and automatically load the mac partition. So opencore is invisible to the user.

### Fix "0251 System CMOS Checksum bad"

If you see this message upon every reboot after booting into mac...

Set `Kernel > Quirks > DisableRTCChecksum` to `True` in config.plist.

Don't forget to modify Windows' registry to fix clock on dual boot systems. Instructions [here](https://superuser.com/questions/494432/force-windows-8-to-use-utc-when-dealing-with-bios-clock).

### Better CPU Frequency Throttling Support

If you want to add CPUFriend.kext and CPUFriendFriend, the i7-5600U can support clock frequencies as low as 600 Mhz, so use that value when generating the kext.

### Fix Boot Chime

Add the `StartupMode = 0x00` optional NVRAM variable to hear sound.

Also use "2" for the `AudioOutMask`.

## References and Links

1. [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
1. [Kymodoke/MacOS-Thinkpad-Yoga-12](https://github.com/kymodoke/MacOS-Thinkpad-Yoga-12) - EFI repo from an older clover install
1. [racka98/Lenovo-Thinkpad-T450-T450s-Hackintosh-Guide-Opencore](https://github.com/racka98/Lenovo-Thinkpad-T450-T450s-Hackintosh-Guide-Opencore) - Thinkpad T450 is literally the same computer but without tablet functionality.
1. [SamarthCat success post](https://www.reddit.com/r/hackintosh/comments/p2c4h8/success_big_sur_on_a_2015_lenovo_thinkpad_yoga_12/) - Proof that Big Sur is able to be installed on this type of computer.
1. [SayantanRC's Guide for Grub to Opencore](https://github.com/SayantanRC/URLs/blob/master/grub_to_opencore.md) - Bootloader modifications for dual-boot systems.
1. [Fix Windows Time Incorrect on Dualboot Hackintosh](https://superuser.com/questions/494432/force-windows-8-to-use-utc-when-dealing-with-bios-clock) - Modify Windows registry to use UTC.
1. [Thread about OneLink Pro Docking Station External Monitor](https://www.tonymacx86.com/threads/lenovo-onelink-pro-dock-2-or-3-monitors.254570/) - debugging external monitor off docking station
1. [Mac Doesn't Support MST](https://www.reddit.com/r/macsysadmin/comments/esk3rz/macos_and_displayport_multistream_transport/) - Multi-stream DisplayPort unsupported (docking station external monitors unsupported)
