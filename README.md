# Neutralizing Intel ME via internal flashing with Intel FPT



 - [Introduction](#introduction)
 - [What is needed and why not `flashrom`](#what-is-needed-and-why-not-flashrom)
 - [Checking the connectivity](#checking-the-connectivity)
 - [Determining whether the FD is locked](#determining-whether-the-fd-is-locked)
 - [Determining whether the Intel Boot Guard is enabled](#determining-whether-the-intel-boot-guard-is-enabled)
 - [Unlocking FD or/and ME](#unlocking-fd-orand-me)
   - [Motherboard jumper / switch](#motherboard-jumper--switch)
   - [Asserting HDA_SDO HIGH on Intel High Definition Audio chips ( >= 6 chipset series )](#setting-hda_sdo-pin-high-on-intel-high-definition-audio-chips---6-series-chipsets-)
   - [Asserting GPIO33 LOW ( <= 5 chipset series )](#setting-gpio33-pin-low-on-chipset-series--5)
   - [BIOS options & hidden options](#bios-options--hidden-options)
 - [Neutralizing ME and Flashing via FPT](#neutralizing-me-and-flashing-via-fpt)
   - [Boot Guard enabled \*OR\* modifying only `HAP` / `AltMeDisable` bits](#in-case-you-have-intel-boot-guard-with-verified-boot-enabled--or--you-only-want-to-use-hapaltmedisable-bits)
   - [Boot Guard disabled \*AND\* cleaning ME region](#in-case-you-dont-have-intel-boot-guard-with-verified-boot-enabled--and--you-want-to-flash-a-neutralized-me-region)
 - [Share your experience](#share-your-experience)



## Introduction

_Intel Flash Programming Tool_ (FPT) is an utility used for internal flash memory programming via [SPI on Intel _Platform Controller Hub_ (PCH)](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Articles/Intel%20docs/Intel%20SPI%20Programming%20Guide.pdf) and older _I/O Controller Hub_ (ICH).

FPT makes part of the _Intel ME System Tools_ toolset, which is available to OEMs/vendors such as Dell, MSI, Lenovo, Gigabyte, Asus, Acer and others. This toolset is used to fine-tune ME firmware before delivering it to the end user. Fortunately for freedom seekers, vendors often include these tools in the distribution package of BIOS updates, thus making them available to the general public. It is available for DOS, UEFI, WIN32, WIN64 and LINUX64 (only versions >= 12).

FPT is able to access the flash memory chips (e.g. Winbond W25Q64FV) in a same way as using external programmers, providing the ability to read and write the following flash parts:

- Flash Descriptor (FD)  
  > The Flash Descriptor is a data structure that is programmed on the SPI flash part. The Descriptor data structure describes the layout of the flash as well as defining configuration parameters for the PCH. The descriptor is on the SPI flash itself and is not in memory mapped space like PCH programming registers. The maximum size of the Flash Descriptor is 4 KBytes. The information stored in the Flash Descriptor can only be written during the manufacturing process as its read/write permissions must be set to Read Only when the computer leaves the manufacturing floor.  
  > **Flash Descriptor contains `AltMeDisable` or `High Assurance Platform (HAP)` bits that disable Intel ME**
- BIOS 
- [Intel Management Engine (ME)](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Articles/Why%20disabling%20IntelME/About%20Intel%20ME%20by%20Libreboot.pdf)  
- Gigabit Ethernet (GbE)
- Platform Data Region (PDR)

All these regions combined is a full flash memory available directly from OS.

Usually most of those regions are not available for write for security reasons (e.g. firmware rootkits), however it's possible to disable write protection using various servicing modes often provided by manufacturers. Intel also have some security override methods in its platforms specifications for OEMs.

Intel platform development guides for OEMs often mention "Flash Descriptor Security Override" or "Intel ME Debug Mode" ("Service Mode" on Dell hardware), that could be enabled via different ways depending on the manufacturer. Some desktop motherboards have a physical jumper that could be tagged as "Service Mode", "FD/Flash/Security Override", "FD Unlock", "ME/TXE/SPS Unlock", "ME/TXE/SPS Service", "Manufacturing Mode" or similar, making full flash memory access facilitated, but laptops do not have such jumpers and rely on something simulated, such as a pin on Intel High Definition Audio codecs to enable a servicing mode. [Unlocking FD or/and ME](#unlocking-fd-orand-me) section provides a better overview of some common methods.


## What is needed and why not `flashrom`

Logically thinking, why would we use FPT when `flashrom` on Linux could provide same functionality via `--programmer internal`? Unfortunately it still doesn't support [intel_spi](https://github.com/torvalds/linux/blob/8afda8b26d01ee26a60ef2f0284a7f01a5ed96f8/drivers/mtd/spi-nor/intel-spi.c) ([description on kernel.org](https://www.kernel.org/doc/Documentation/mtd/intel-spi.txt)) which is a module used on Linux to provide the PCH SPI access. 
There are some attempts to make a software flasher like [https://github.com/system76/intel-spi](https://github.com/system76/intel-spi), but there is nothing specific and universally usable.

FPT is available for Linux starting from versions 12 only, in case you have this ME version or higher you could try to perform everything described here on Linux.

However it's recommendable to use Windows to follow this guide because **a)** all FPT versions are available for Windows **b)** avoiding any kind of headache related to compatibility/complexity. You could also do this on DOS or UEFI since FPT is available for them, but they won't be described here to avoid any confusion. 

Now, be sure you have the following in order to follow this guide:

- **Windows** 7/8/10  _\_OR\__  **Linux**  in case you have ME >=12

  The command examples in this guide will be given using `fptw64.exe` which is a FPT executable for Windows 64-bit; on Linux the executable is named `FPT` thus should be called like `./FPT`;
  Windows 7 x64 Professional is recommended because it is guaranteed to work when following this guide.
  Windows 8 and 10 weren't tested and you may have issues on using FPT due to possible additional protection mechanisms in kernel.
  
  The OS should be running from an internal or external hard drive on the hardware which will be re-flashed

- **Intel MEI driver**

  Intel Management Engine Interface is not required for FPT, but it's required for communicating with Intel ME. In our specific case it is needed to run `MEInfo` in order to determine the Intel Boot Guard status on your system for choosing a correct method of disabling Intel ME.

  It is important to know and have a correct version of a driver, but there is no universal way of determining which is a correct version for each case.

  You could first try [this driver](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Intel_ME_System_Tools/Intel%20MEI%20Driver%20INF%20v11.0.5.1189%20%28WinXP%20-%20Win10%29.7z) which is a universal MEI driver for all versions <=11. This driver is for Windows XP, 7, 8, and 10. You must manually load it by guessing the correct MEI device in the Windows Device Manager, which usually looks like 'PCI Simple Communications Controller' or anything else related to PCI, but it's also common that it is shown as 'Unknown device'.
	
  If it didn't work, try visiting the official site of the manufacturer of your system in order to search for MEI drivers. Most manufacturers provide a driver package called like 'Intel Management Engine Components', 'Intel Management Engine Software' and similar.

  > This guide suppose you have a clean Windows installation. In case you are a regular Windows user you may have Intel MEI interface driver already installed, check for `Intel Management Engine Interface` in `System devices` in the _Windows Device Manager_. 
  >
  > On Linux the driver is provided by kernel, so you don't need to install anything. However, it is still necessary to know the ME version for downloading the correct version of _Intel ME System Tools_.
  >

- **Intel ME System Tools**

  This distribution includes _Intel Flash Programming Tool_ (FPT) and other interesting software to get information on chipset and ME, such as _MEInfo_, which can be used to determine whether you have Intel Boot Guard enabled or not.

  The FPT binary is usually named like `fptw64.exe`, `fptw.exe` or `fpt.exe` and could be found in the `Flash Programming Tool` folder inside the _Intel ME System Tools_ distribution

  It's highly recommended to get the same version of the _Intel ME System Tools_ distribution that match your ME firmware version.  
  You can find and download _Intel ME System Tools_ here: https://github.com/mostav02/Remove_IntelME_FPT/tree/master/Intel_ME_System_Tools

This guide supposes you are already familiar with `me_cleaner` ([https://github.com/corna/me_cleaner](https://github.com/corna/me_cleaner)) which is a key instrument in disabling Intel ME



## Checking the connectivity

**[!] Important**: FPT and MEInfo must be always run in the escalated privileges environment, running cmd.exe even if you're using an admin account is not enough.

- Run `cmd.exe` in the escalated privileges mode (Right-click on `cmd.exe` and and choose _Run as administrator_)

- Move to the directory `Flash Programming Tool` which is inside the extracted _Intel ME System Tools_ distribution.

- Run FPT with `-i ` and optionally `-verbose` flags (e.g. `fptw64.exe -i`) to check whether you have the supported SPI and ensure FPT is able to use it.

It should display various information on your BIOS chips and the flash descriptor information. 


## Determining whether the FD is locked

Running `fptw64.exe -I` usually outputs the proper information regarding flash regions lock status.  
A typical output looks like this:  

```
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

Which means the FD and ME regions are not writable, but FD can be dumped.  
When the FD is unlocked, it usually outputs like this:  

```
             Read    Write
    Desc  :  Yes     Yes
    Host  :  Yes     Yes
    ME    :  Yes     Yes
    GbE   :  Yes     Yes
    PDR   :  Yes     Yes

```

However, some FPT versions may give false-positives regarding access status using `-I`, so a guaranteed way is to try reading the flash memory and writing it back, in this case the FD or ME region.

Read the FD with FPT:  
`fptw64.exe -DESC -D myFD.bin`

Try to write the FD back:  
`fptw64.exe -DESC -F myFD.bin`

<div id="important-note-on-flashing-anything">

| WARNING: Even that Intel PCH SPI is fast to write, it's still highly recommended to have the backup/redundant power, such as an attached battery or having your PC plugged into a UPS. Unexpected power loss during the flash process **will** result into the bricked device and you will need to use the external programmer to flash the FD back in order to recover the machine.  |
| --- |

</div>

In case you were able to write the FD, you don't likely need to do anything more and can skip directly to the [Neutralizing ME and Flashing via FPT](#neutralizing-me-and-flashing-via-fpt) section where you can safely disable ME via the `AltMeDisable` or `High Assurance Platform (HAP)` bits.

In case the FD is locked, you still could try to check whether ME is unlocked. 

Read the ME with FPT:  
`fptw64.exe -ME -D myME.bin`

Try to write the ME back:  
`fptw64.exe -ME -F myME.bin`


In case you were able to flash the dumped ME **_and_** you don't have the Boot Guard Enabled you could safely neutralize ME. Read the [Determining whether the Intel Boot Guard is enabled](#determining-whether-the-intel-boot-guard-is-enabled) before skipping to [Neutralizing ME and Flashing via FPT](#neutralizing-me-and-flashing-via-fpt).




## Determining whether the Intel Boot Guard is enabled

Move to the directory `MEInfo` which is inside the extracted _Intel ME System Tools_ distribution.

Run the MEInfo utility with a `-verbose`flag  to check if you the following flags present and enabled:

```
Measured Boot                                Enabled                  Enabled
Verified Boot                                Enabled                  Enabled
```

- In case `Verified Boot` is enabled, you are not allowed to flash a modified ME region, because **your computer won't power on in case you flash the modified ME which won't pass a cryptographic verification,** but it's still safe to reflash the FD with modified `HAP`/`AltMeDisable` bits (see `me_cleaner -s` / `ifdtool -M` flags).

- In case you have only `Measured Boot` enabled, there is a possibility that you won't incur the same problem as with `Verified Boot`, but there is no enough information on this to say whether it's 100% safe or not to flash ME on `Measured Boot`-enabled computers. Please make your own research.

- In case you don't have `Verified Boot` or `Measured Boot` enabled it means you have Intel Boot Guard disabled and it's safe to flash the neutralized ME region.

- **In any case, flashing only the FD with HAP/AltMeDisable is enough to neutralize ME**, since it's a method used by NSA and other US government agencies on their computers that carry sensitive information. Some quotes from various sources combined on [Wikipedia article about Intel ME](https://en.wikipedia.org/wiki/Intel_Management_Engine) demonstrate that disabling Intel ME in the Flash Descriptor serves its purpose:

```
In August 2017, Russian company Positive Technologies (Dmitry Sklyarov) published a method to disable the ME via an undocumented built-in mode. As Intel has confirmed the ME contains a switch to enable government authorities such as the NSA to make the ME go into High-Assurance Platform (HAP) mode after boot. This mode disables most of ME's functions, and was intended to be available only in machines produced for specific purchasers like the US government; however, most machines sold on the retail market can be made to activate the switch ( http://blog.ptsecurity.com/2017/08/disabling-intel-me.html https://www.bleepingcomputer.com/news/hardware/researchers-find-a-way-to-disable-much-hated-intel-me-component-courtesy-of-the-nsa/ )
...
Dell, in December 2017, began showing certain laptops on its website that offered the "Systems Management" option "Intel vPro - ME Inoperable, Custom Order" for an additional fee. Dell has not announced or publicly explained the methods used. In response to press requests, Dell stated that those systems had been offered for quite a while, but not for the general public, and had found their way to the website only inadvertently. The laptops are available only by custom order and only to military, government and intelligence agencies. They are specifically designed for covert operations, such as providing a very robust case and a "stealth" operating mode kill switch that disables display, LED lights, speaker, fan and any wireless technology. (https://www.heise.de/newsticker/meldung/Dell-schaltet-Intel-Management-Engine-in-Spezial-Notebooks-ab-3909860.html)
```

See also:

_Verifying Intel Boot Guard status on Linux:_ [_https://github.com/corna/me_cleaner/wiki/Intel-Boot-Guard_](https://github.com/corna/me_cleaner/wiki/Intel-Boot-Guard)  (`MEInfo` v11 may also be used instead, since it's available for Linux)
_Intel Boot Guard - a detailed description_: [_https://trmm.net/Bootguard_](https://trmm.net/Bootguard) ([mirrored page](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Articles/About_Intel_Boot_Guard_by_Trammell_Hudson.pdf))





## Unlocking FD or/and ME

There are several known and unknown methods of unlocking the Flash Descriptor to have a full access to the flash memory via SPI

### Server / Desktop

- #### Motherboard Jumper / Switch

  Many desktop motherboards have a physical jumper that could be tagged as "Service Mode", "FD/Flash/Security Override", "FD Unlock", "FDO", "ME/TXE/SPS Unlock", "ME/TXE/SPS Service", "Manufacturing Mode" or similar.
	
  This usually unlocks the FD for a full access. Refer to your motherboard service manual for more information.


### Server / Desktop / Laptop


- #### Setting HDA_SDO pin HIGH on _Intel High Definition Audio_ chips ( >= 6 Series chipsets )

  This is an official method described by Intel in the technical chipset documentation distributed for OEMs to provide a service mode on their motherboards.


  <details>
  <summary>It is described as "Flash Descriptor Security Override" or "Intel ME Debug Mode" (click to expand image)</summary>
  
  ![Intel Flash Descriptor Security Override signal](https://raw.githubusercontent.com/mostav02/Remove_IntelME_FPT/master/Misc/Intel_HDA_SDO_secoverride_strap_definition.png)

	Sources:  
	[Intel 6 Series Chipset and Intel C200 Series Chipset - datasheet](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Articles/Intel%20docs/6-chipset-c200-chipset-datasheet.pdf)  
	[Intel 300 Series and C240 Series Chipset Family Platform Controller Hub - datasheet](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Articles/Intel%20docs/300-series-c240-series-chipset-pch-datasheet-vol-1.pdf)

  </details>


  It unlocks the full access to all flash regions (read/write).
	
  You need to find a location of Intel HD Audio (HDA) chip used on your motherboard. You can either go to your vendor's website and search for audio drivers which would tell which chip is used on your model. Dell and Lenovo always mention what manufacturer and chip series are used in the audio driver description.
  The manufacturer is mostly one of the following:  Realtek, Conexant, IDT, VIA, Wolfson Microelectronics, and formerly C-Media. 
		
  Once you found the HDA chip on your motherboard, you have to determine what pin numbers are used for `SDATA_OUT` (or `SDO`, `HDA_SDO`) and `DVDD` (1.5V / 3.3V) by searching for a datasheet for that chip. Most manufacturers define pins 1 and 5 for that, but there are some exceptions like Conexant chips that mainly use pin 18 as `DVDD` and pin 4 as `SDO`.
  Here are few examples that demonstrate the difference among HDA chip manufacturers and models:
	
	DVDD | HDA_SDO | Model
	--- | --- | ---
	1  | 5  | Realtek ALC269
	1  | 5  | Realtek ALC882
	1  | 5  | Realtek ALC3235
	1  | 4  | IDT 92HD87
	1  | 5  | IDT 92HD90
	1  | 5  | VIA VT1705
	1  | 5  | VIA VT1708B
	1  | 5  | VIA VT2021
	18 | 4  | Conexant CX20671/CX20672

  The more updated table is [here](https://github.com/mostav02/Remove_IntelME_FPT/blob/master/Misc/pinouts/README.md)
	
  Once you found the correct pins you must short them during powering on your system (PWROK state), after which the full flash memory address range will be globally unlocked until the next reboot.

  Use [Determining whether the FD is unlocked](#determining-whether-the-fd-is-locked) section to check if the SPI flash is unlocked and in case of success move straight to [Neutralizing ME and Flashing via FPT](#neutralizing-me-and-flashing-via-fpt)


  - **NOTE:** It's recommended to downgrade the BIOS to the lower possible version before performing this method. 

  - **NOTE:** Be careful when shorting the pins, they are so fragile on those chips that making pressure with a needle/clip may deform and damage the pin. 


- #### Setting GPIO33 pin LOW on chipset series <= 5 

  Older motherboards have a strap connected pin called GPIO33 for unlocking the SPI flash read/write access. 
  This can be performed by shorting GPIO33 and Ground(GND) pins during PWROK state. There is no universal pins location for this one so you always must refer to your motherboard manufacturer servicing manuals to find it.
	
  An example of a GPIO33 pin location may be found here https://libreboot.org/docs/hardware/gm45_remove_me.html#early_notes

- #### BIOS options & hidden options

  Many OEMs provide BIOS options to unlock flash memory parts for servicing needs. Some allow all regions to be unlocked (like FD+BIOS+ME) while other may only allow BIOS or ME to be unlocked for writing.
	
  In many cases those options are hidden and you need to find them by dumping your BIOS image (usually it's possible by doing `fptw64.exe -BIOS -D biosimage.bin` since many BIOSes on Intel motherboards are available for reading by default)

  The main utitilies used by BIOS modders are [UEFITool](https://github.com/LongSoft/UEFITool/releases) and the [IFR Extractor](https://github.com/LongSoft/Universal-IFR-Extractor/releases) (there are more, but since most systems run an UEFI BIOS there is often no need of using more specific software from vendors such as AMI, Phoenix or Award)

  `UEFITool` is used for opening BIOS dump images and finding the hidden UI menus. The process is not mentioned here because a simple [DuckDuckGo](https://duckduckgo.com/?q=Using+%22UEFITool%22+for+bios+modding) (or Google) search will provide a lot of guides on that subject. On Dell laptops the UI is called `SetupPrep`, while on other systems it's just `Setup` and may always vary from OEM to OEM.
	
  `IFR Extractor` is used to convert `PE32` or full body images extracted by `UEFITool` into a readable text format, where you can inspect all the BIOS menus and their options.
	
  The common search terms are "Me FW Image Re-Flash", "ME/TXE Disable", "HMRFPO", "Disable SPI/BIOS Protection" but not limited to. You may carefully inspect the extracted information and look for options that intuitively sound related to flash/service/unlocking.
	
  Once you found an option look for its `VarOffset`, which is a variable address of the option.
  For example "Me FW Image Re-Flash" option is present and have the following `VarOffset` values on some laptops: `0x5B` for on Dell E7450, `0x2BC` on Dell M4800, Dell Venue 7130, `0x67A` on Dell XPS9360

  To change the hidden option/variable's value you must boot into a EFI GRUB shell (download [here](https://github.com/mostav02/Remove_IntelME_FPT/raw/master/Misc/bootx64.efi) or [here (alternative version)](https://github.com/datasone/grub-mod-setup_var), create `/efi/boot` directory on a FAT32 formatted USB drive, put the `.efi` file in it then boot it from the BIOS menu_) and set those variables using the `setup_var` command, for example:
	
  `setup_var 0xXX 0x01` to to enable or `setup_var 0xXX 0x00` to disable the option with a variable address `0xXX` (replace XX to the actual hex value)

  type `reboot` and check if this option has unlocked something in the SPI using FPT.

  > **Note:** It is not necessarily that such BIOS options/variables will be named. Sometimes they are unnamed variables or even combination of variables that perform SPI servicing unlock, thus each case is subject to reverse engineering. Search engines may point to BIOS modding forums that may contain some useful information, but be ready to face some misleading opinions too, since regulars of such communities often neglect important security aspects, hence better make your own research.


- #### Flashing an unlocked FD with OEM BIOS upgrades

  This method won't be explained here because it's dangerous and very uncommon, since most OEMs don't include the FD in BIOS upgrades, however some older systems may have BIOS upgrades that contain a full flash.
  

- #### Other methods

  Please **perform a (re)search** in case none of the above worked for you. 
	
  Start with simple search engine requests like "Unlocking Intel FD" or "Unlocking SPI Flash" which may give you some extra methods regarding your specific hardware. There are may be methods not available/disclosed for general public by OEMs and it's always a subject of making an aditional research involving some reverse (or social...) engineering.  
  It is very uncommon that laptop/desktop hardware is completely hardlocked for reflash and if some hardware lack information it doesn't mean there is no way to unlock it.  
  Some OEM/ODM have their own servicing utilities that perform FD unlocking, please make an effort of searching them.

  You could always end by using an external programmer if none worked, but in this case you will need other guides, because this guide doesn't cover any external programming.




## Neutralizing ME and Flashing via FPT

Congratulations on getting here! (This section supposes you have an unlocked flash descriptor)

  - Before proceeding, make a full flash dump first:  
    `fptw64.exe -D FullDump.bin`

### In case you have Intel Boot Guard with Verified Boot enabled ** **_OR_** ** you only want to use `HAP`/`AltMeDisable` bits:

It's only necessary to re-flash the flash descriptor (4kb image).

> `me_cleaner` doesn't work on 4KB Flash Descriptor files for modifying the `HAP`/`AltMeDisable` bits, which is an implementation [issue](https://github.com/corna/me_cleaner/issues/323).
> The `ifdtool` from https://github.com/coreboot/coreboot/tree/master/util/ifdtool does it


- #### Using `ifdtool`

  - Use the `ifdtool` with `-M 1` flag on the FD you dumped with FPT (`fptw64.exe -DESC -D myFD.bin`):  
  `./ifdtool -M 1 myFD.bin`

  - Flash the new file `myFD.bin.new` with FPT:  
  `fptw64.exe -DESC -F myFD.bin.new` [IMPORTANT WARNING](#important-note-on-flashing-anything)

- #### Using `me_cleaner` (same effect as using `ifdtool`)

  In case you don't have ifdtool or just lazy to compile it:

  - Use `me_cleaner` with `-s` on a full flash memory dump made with `fptw64.exe -D FullDump.bin`:  
  `python me_cleaner.py -s FullDump.bin`

  - Flash the full rom back with FPT:  
  `fptw64.exe -F FullDump.bin`

  OR you can extract first 4096 bytes of neutralized `FullDump.bin` which will result into a Flash Descriptor image, then flash it with `fptw64.exe -DESC -F extracted_flash_descriptor_from_full_dump`

In case of a good FPT output, Intel ME will be disabled right after reboot.  Use `MEInfo` to ensure it won't be able to use MEI anymore.


### In case you don't have Intel Boot Guard with Verified Boot enabled ** **_AND_** ** you want to flash a neutralized ME region:

> If you also want to set `HAP`/`AltMeDisable` bits, perform a second part of [THIS STEP](#in-case-you-have-intel-boot-guard-with-verified-boot-enabled--or--you-only-want-to-use-hapaltmedisable-bits) which describes using `me_cleaner` on a full dump. Use `-S` instead of `-s` to also clean the ME region.

  `me_cleaner` is working fine on the ME-only region files without needing a full flash dump.

  - Use the `me_cleaner` without flags on the FD you dumped with FPT (`fptw64.exe -ME -D myME.bin`):  
  `python me_cleaner.py myME.bin`

  - Flash the modified `myME.bin` with FPT:  
  `fptw64.exe -ME -F myME.bin` [IMPORTANT WARNING](#important-note-on-flashing-anything)

In case of a good FPT output, Intel ME will be disabled right after reboot. Use `MEInfo` to ensure it won't be able to use MEI anymore.

### Share your experience

Please consider sharing your experience once you accomplished disabling Intel ME.

- Leave a comment here https://github.com/corna/me_cleaner/issues/3 describing your machine and what flashing method you have used (internal/external)

- If you unlocked a flash descriptor on some machine that is not present here https://github.com/mostav02/Remove_IntelME_FPT/tree/master/Unlocking_Flash_Descriptor please create an issue or a Pull Request to include it in this repo. 

	If your machine is present there but you have used a different method for unlocking a FD please create an issue or a PR in order to update the details.
