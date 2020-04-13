#### Remove Intel ME on Asus N53

Intel ME System Tools v7 is required

1. Unlock the SPI flash by shorting pins 1&5 on `Realtek ALC269` HDA chip during PWROK (powering on)  
2. Dump full firmware with FPT: `fptw.exe -D fullspi.bin`  
3. Use `me_cleaner -S fullspi.bin`  
4. Flash it back with FPT: `fptw.exe -F fullspi.bin`  
5. Reboot

##### Device information:


- Sandy Bridge
- i7-2670QM
- OEM BIOS


Intel Boot Guard: No
ME version: 7.0.4.1197
ME capabilities:

```
ME Capability: Full Network manageability                 : OFF
ME Capability: Regular Network manageability              : OFF
ME Capability: Manageability                              : OFF
ME Capability: Small business technology                  : OFF
ME Capability: Level III manageability                    : OFF
ME Capability: IntelR Anti-Theft (AT)                     : OFF
ME Capability: IntelR Capability Licensing Service (CLS)  : ON
ME Capability: IntelR Power Sharing Technology (MPC)      : ON
ME Capability: ICC Over Clocking                          : ON
ME Capability: Protected Audio Video Path (PAVP)          : ON
ME Capability: IPV6                                       : OFF
ME Capability: KVM Remote Control (KVM)                   : OFF
ME Capability: Outbreak Containment Heuristic (OCH)       : OFF
ME Capability: Virtual LAN (VLAN)                         : OFF
ME Capability: TLS                                        : OFF
ME Capability: Wireless LAN (WLAN)                        : OFF

CPU Upgrade State:                      Upgrade Capable
Cryptography Support:                   Disabled
Last ME reset reason:                   Power up
Local FWUpdate:                         Enabled
BIOS and GbE Config Lock:               Unknown
Host Read Access to ME:                 Disabled
Host Write Access to ME:                Disabled
SPI Flash ID #1:                        EF4016
SPI Flash ID VSCC #1:                   20052005
SPI Flash BIOS VSCC:                    20052005
BIOS boot State:                        Post Boot
OEM Id:                                 00000000-0000-0000-0000-000000000000
OEM Tag:                                0x00000000

```


Hardware:


```
00:00.0 Host bridge: Intel Corporation 2nd Generation Core Processor Family DRAM Controller (rev 09)
00:01.0 PCI bridge: Intel Corporation Xeon E3-1200/2nd Generation Core Processor Family PCI Express Root Port (rev 09)
00:02.0 VGA compatible controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
00:16.0 Communication controller: Intel Corporation 6 Series/C200 Series Chipset Family MEI Controller #1 (rev 04)
00:1a.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #2 (rev 05)
00:1b.0 Audio device: Intel Corporation 6 Series/C200 Series Chipset Family High Definition Audio Controller (rev 05)
00:1c.0 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 1 (rev b5)
00:1c.1 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 2 (rev b5)
00:1c.3 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 4 (rev b5)
00:1c.5 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 6 (rev b5)
00:1d.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #1 (rev 05)
00:1f.0 ISA bridge: Intel Corporation HM65 Express Chipset Family LPC Controller (rev 05)
00:1f.2 SATA controller: Intel Corporation 6 Series/C200 Series Chipset Family 6 port SATA AHCI Controller (rev 05)
00:1f.3 SMBus: Intel Corporation 6 Series/C200 Series Chipset Family SMBus Controller (rev 05)
01:00.0 VGA compatible controller: NVIDIA Corporation GF108M [GeForce GT 540M] (rev a1)
03:00.0 Network controller: Intel Corporation Centrino Wireless-N 1000 [Condor Peak] ### note: replaced this with Atheros
04:00.0 USB controller: Fresco Logic FL1000G USB 3.0 Host Controller (rev 04)
05:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 06)
```



Flash Descriptor information:

```
C:\Intel ME System Tools v7 r2\Intel ME System Tools v7 r2\Flash Programming Tool\WIN64>fptw64.exe -I
Intel (R) Flash Programming Tool. Version: 7.1.50.1166
Copyright (c) 2007-2011, Intel Corporation. All rights reserved.
Platform: Intel(R) HM65 Express Chipset Revision: Unknown
Reading HSFSTS register... Flash Descriptor: Valid
    --- Flash Devices Found ---
    W25Q32FV    ID:0xEF4016    Size: 4096KB (32768Kb)

    --- Flash Image Information --
    Signature: VALID
    Number of Flash Components: 1
        Component 1 - 4096KB (32768Kb)
    Regions:
        Descriptor - Base: 0x000000, Limit: 0x000FFF
        BIOS       - Base: 0x180000, Limit: 0x3FFFFF
        ME         - Base: 0x001000, Limit: 0x17FFFF
        GbE        - Not present
        PDR        - Not present
    Master Region Access:
        CPU/BIOS - ID: 0x0000, Read: 0x0B, Write: 0x0A
        ME       - ID: 0x0000, Read: 0x0D, Write: 0x0C
        GbE      - ID: 0x0118, Read: 0x08, Write: 0x08
Total Accessable SPI Memory: 4096KB, Total Installed SPI Memory : 4096KB
FPT Operation Passed
```
