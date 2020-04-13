#### Remove Intel ME on Dell Latitude E7250

Intel ME System Tools v10 is needed

> NOTE: Downgraded to BIOS A08 before performing SPI unlock

1. Remove the keyboard with the top cover and locate the `Realtek ALC3235` HDA chip 
2. Unlock the SPI flash by shorting pins 1&5 on `Realtek ALC3235` chip during PWROK (powering on)  _Note: the power button is accessible without the top panel_
3. Dump full firmware with FPT `fptw.exe -D fullspi.bin`  **OR**  flash descriptor only  `fptw.exe -DESC -D fd.bin`
4. Use `me_cleaner -s fullspi.bin` on a full dump **OR** `ifdtool -M 1 fd.bin` on a flash descriptor dump
5. Flash the full dump FPT `fptw.exe -F fullspi.bin`  **OR**  in case only flash descriptor `fptw.exe -DESC -F fd.bin`
6. Reboot
7. Intel ME should be disabled now and MEI unavailable

> NOTE: BIOS upgrades skip ME FW upgrade when ME is disabled. Safe to upgrade.


##### Device information:


- Broadwell Mobile @ Wildcat Point LP
- i7-5600U
- OEM BIOS


Intel Boot Guard: Yes (Measured & Verified Boot)
ME version: 10.0.60.3000 LP
ME capabilities:

```
MEBx Version:                           10.0.0.0007
Gbe Version:                            0.2
VendorID:                               8086
PCH Version:                            3
FW Version:                             10.0.60.3000 LP
LMS Version:                            Not Available
MEI Driver Version:                     11.0.5.1189
Wireless Hardware Version:              Not Available
Wireless Driver Version:                Not Available

FW Capabilities:                        0x5DF65A45

    Intel(R) Active Management Technology - PRESENT/ENABLED
    Intel(R) Standard Manageability - NOT PRESENT
    Intel(R) Capability Licensing Service - PRESENT/ENABLED
    Protect Audio Video Path - PRESENT/ENABLED
    Intel(R) Dynamic Application Loader - PRESENT/ENABLED
    Intel(R) NFC Capabilities - NOT PRESENT
    Intel(R) Platform Trust Technology - NOT PRESENT

Intel(R) AMT State:                     Enabled
TLS:                                    Enabled
Last ME reset reason:                   Power up
Local FWUpdate:                         Enabled

Get BIOS flash lockdown status...done
BIOS Config Lock:                       Enabled

Get GbE flash lockdown status...done
GbE Config Lock:                        Enabled

Get flash master region access status...done
Host Read Access to ME:                 Disabled
Host Write Access to ME:                Disabled
SPI Flash ID #1:                        EF4017
SPI Flash ID VSCC #1:                   20252025
SPI Flash ID #2:                        EF4016
SPI Flash ID VSCC #2:                   20252025
SPI Flash BIOS VSCC:                    20252025
Protected Range Register Base #0 0x0
Protected Range Register Limit #0 0x0
Protected Range Register Base #1 0x0
Protected Range Register Limit #1 0x0
Protected Range Register Base #2 0x0
Protected Range Register Limit #2 0x0
Protected Range Register Base #3 0x0
Protected Range Register Limit #3 0x0
Protected Range Register Base #4 0x0
Protected Range Register Limit #4 0x0
BIOS boot State:                        Post Boot

Get Intel(R) AMT state command...done
Link Status:                            Link down

Get LanInterfaceSettings command for wired interface...done
MAC Address:
Get Provisioning Tls Mode command...done

Get provisioning state command...done
20-47-47-ae-d6-97
IPv4 Address:                           0.0.0.0

Get LanInterfaceSettings command for wireless interface...done
Command response reports interface doesn't exist

Get IPv6InterfaceStatus command for wired interface...done
Command response reports interface was disabled
IPv6 Enablement:                        Disabled

Get privacy/security level info command...done
Privacy/Security Level:                 Default

Get provisioning state command...done
Configuration state:                    Not started

Get Provisioning Tls Mode command...done
Provisioning Mode:                      PKI
Capability Licensing Service:           Enabled

Get ME FWU OEM Tag command...done
OEM Tag:                                0x00000000

Get System Integrator ID command...done
Slot 1 Board Manufacturer:              0x00001028

Get System Integrator ID command...This slot is unused.
Slot 2 System Assembler:                Unused

Get System Integrator ID command...This slot is unused.
Slot 3 Reserved:                        Unused

Get M3 Autotest command...done
M3 Autotest:                            Enabled

Get CLink Status command...done
C-link Status:                          Enabled

Get ME FWU Platform Attribute (WLAN ucode) command...done
Wireless Micro-code Mismatch:           No
Wireless Micro-code ID in Firmware:     0x095A
Wireless LAN in Firmware:               Intel(R) Dual Band Wireless-AC 7265
Wireless Hardware ID:                   No Intel WLAN card installed
Wireless LAN Hardware:                  No Intel WLAN card installed

Get ME FWU Platform Attribute (WLAN ucode) command...done
Localized Language:                     English

Get ME FWU Info command...done
Independent Firmware Recovery:          Disabled
Keybox:                                 Not Provisioned

Get Oem Public Key Hash command...done
OEM Public Key Hash (FPF):              141CDDA87A2B83365FBE497111152182CFBC704A65D1F4EA2A758D6CC628D6D1
OEM Public Key Hash (ME):

Get ACM SVN command...done
ACM SVN FPF:                            0x3

Get KM SVN command...done
KM SVN FPF:                             0x0

Get BSMM SVN command...done
BSMM SVN FPF:                           0x0

Get Oem Boot Guard Policy command...done

                                        FPF                 ME
                                        ---                 --
Force Boot Guard ACM:                   Enabled
Protect BIOS Environment:               Enabled
CPU Debug Disabled:                     Disabled
BSP Initialization Disabled:            Disabled
Measured Boot:                          Enabled
Verified Boot:                          Enabled
Key Manifest ID:                        0xf
Enforcement Policy:                     0x3

```

Hardware (with Intel ME disabled):

```
00:00.0 Host bridge: Intel Corporation Broadwell-U Host Bridge -OPI (rev 09)
00:02.0 VGA compatible controller: Intel Corporation HD Graphics 5500 (rev 09)
00:03.0 Audio device: Intel Corporation Broadwell-U Audio Controller (rev 09)
00:04.0 Signal processing controller: Intel Corporation Broadwell-U Processor Thermal Subsystem (rev 09)
00:14.0 USB controller: Intel Corporation Wildcat Point-LP USB xHCI Controller (rev 03)
00:19.0 Ethernet controller: Intel Corporation Ethernet Connection (3) I218-LM (rev 03)
00:1c.0 PCI bridge: Intel Corporation Wildcat Point-LP PCI Express Root Port #1 (rev e3)
00:1c.3 PCI bridge: Intel Corporation Wildcat Point-LP PCI Express Root Port #4 (rev e3)
00:1d.0 USB controller: Intel Corporation Wildcat Point-LP USB EHCI Controller (rev 03)
00:1f.0 ISA bridge: Intel Corporation Wildcat Point-LP LPC Controller (rev 03)
00:1f.2 SATA controller: Intel Corporation Wildcat Point-LP SATA Controller [AHCI Mode] (rev 03)
00:1f.3 SMBus: Intel Corporation Wildcat Point-LP SMBus Controller (rev 03)
01:00.0 SD Host controller: O2 Micro, Inc. SD/MMC Card Reader Controller (rev 01)
02:00.0 Network controller: Qualcomm Atheros QCA9565 / AR9565 Wireless Network Adapter (rev 01)
```

Flash Descriptor information:

```
C:\Intel ME System Tools v10.0 r7\Flash Programming Tool\WIN64>fptw64.exe -I

Intel (R) Flash Programming Tool. Version:  10.0.30.1054
Copyright (c) 2007 - 2014, Intel Corporation. All rights reserved.

Platform: Intel(R) Premium Express Chipset
Reading HSFSTS register... Flash Descriptor: Valid

    --- Flash Devices Found ---
    W25Q64BV    ID:0xEF4017    Size: 8192KB (65536Kb)
    W25Q32BV    ID:0xEF4016    Size: 4096KB (32768Kb)

    --- Flash Image Information --
    Signature: VALID
    Number of Flash Components: 2
        Component 1 - 8192KB (65536Kb)
        Component 2 - 4096KB (32768Kb)
    Regions:
        Descriptor - Base: 0x000000, Limit: 0x000FFF
        BIOS       - Base: 0x600000, Limit: 0xBFFFFF
        ME         - Base: 0x005000, Limit: 0x5FFFFF
        GbE        - Base: 0x001000, Limit: 0x004FFF
        PDR        - Not present
    Master Region Access:
        CPU/BIOS - ID: 0x0000, Read: 0x0B, Write: 0x0A
        ME       - ID: 0x0000, Read: 0x0D, Write: 0x0C
        GbE      - ID: 0x0118, Read: 0x08, Write: 0x08
 Based on the Host Region FRACC the Host/CPU/BIOS has ( 0x00004A4B ) :
             Read    Write
    Desc  :  Yes      No
    Host  :  Yes     Yes
    ME    :   No      No
    GbE   :  Yes     Yes
    PDR   :   No      No

Total Accessable SPI Memory: 12288KB, Total Installed SPI Memory : 12288KB

FPT Operation Passed

```
