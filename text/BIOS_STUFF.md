# BIOS Stuff
What can we do?
* Update the microcodes, this is the most important modification to make.
* Update RST firmware, this is responsible for SATA/RAID.
* Update iGPU VBIOS.
* Add KVM/AMT to the SPI region, enables vPro stuff like remote desktop even if system is off.
* Modify DSDT tables, in theory we could add all our ACPI patches needed for macOS to the BIOS itself.
* Modify Dell boot logo, replace it with a fruity pineapple if you're so inclined.

## Things to download
* Windows 10, see below for a guide for those who have left Windows behind long ago
* UBU
* Intel ME Platform Tools 9.1
* Intel firmware files
* MMTool

All those we need can be found the first post of [this](https://www.win-raid.com/t154f16-Tool-Guide-News-quot-UEFI-BIOS-Updater-quot-UBU.html) thread. Search for ```SoniX's MEGA link``` in there and open it up. We also need to download and install ```Binary Modification Program``` found that forum thread.

The things you need to download from the Mega repo are:
* UBU_v1_xxxxx.rar
* Tools -> mmt.rar
* Files_xxxxx -> Intel GOP/RST/VBIOS *.7z files

We also need Intel ME System Tools 9.1, a download link can be found [here](https://www.win-raid.com/t596f39-Intel-Management-Engine-Drivers-Firmware-amp-System-Tools.html), search for ```9.1 r7```.

## Extracting

Intel for some reason doesn't make their ME System Tools public, but now that we have a copy we can unpack it and start using some of the tools. First lets check the versions.

Open a new PowerShell as admin and go to the MEInfo\WIN64 folder and run ```.\MEInfoWin64.exe```:

```
PS C:\Intel ME System Tools v9.1 r7\MEInfo\WIN64> .\MEInfoWin64.exe

Intel(R) MEInfo Version: 9.1.45.3000
Copyright(C) 2005 - 2017, Intel Corporation. All rights reserved.

Intel(R) ME code versions:

BIOS Version:                           A18
MEBx Version:                           9.0.0.0029
Gbe Version:                            1.3
VendorID:                               8086
PCH Version:                            4
FW Version:                             9.1.45.3000 H
LMS Version:                            Not Available
MEI Driver Version:                     11.7.0.1032
```

I removed the rest as its not important for this check. Simply make sure your MEBx is at version 9.0.x and the firmware is on 9.1.x. If you're on the latest Dell BIOS these should match up.

We can only upgrade MEBx and the firmware within its major release. This means that we have to stay on 9.0.x for MEBx and 9.1.x for firmware related things. This is no problem and we're not touching those.

Now lets extract the current BIOS, don't use any other tools do this, we can't just extract the .rom file from the Dell BIOS executable.

Navigate to the ```Flash Programming Tool\WIN64``` folder and execute ```.\fptw64.exe -bios -d bios_backup.bin```:

```
PS C:\Intel ME System Tools v9.1 r7\Flash Programming Tool\WIN64> .\fptw64.exe -bios -d bios_backup.bin

Intel (R) Flash Programming Tool. Version:  9.1.10.1000
Copyright (c) 2007 - 2014, Intel Corporation. All rights reserved.

Platform: Intel(R) Q87 Express Chipset
Reading HSFSTS register... Flash Descriptor: Valid

    --- Flash Devices Found ---
    MX25L6405D    ID:0xC22017    Size: 8192KB (65536Kb)
    MX25L3205A    ID:0xC22016    Size: 4096KB (32768Kb)


- Reading Flash [0xC00000] 6144KB of 6144KB - 100% complete.
Writing flash contents to file "bios_backup.bin"...

Memory Dump Complete
FPT Operation Passed
```

We now have a backup of the BIOS and the file we need to apply our modifications to. We can move on to the next section.

## Modifying
Download and extract UBU somewhere simple. I picked C:\UBU. Now you'll need to find a copy of MMTool. We downloaded a file called mmt.rar, extract it and you'll find a bunch of versions of the MMTool utility. I used v5.0.0.7 and renamed it to ```mmtool_a4.exe``` and placed in the UBU root folder. At this point also copy your extracted BIOS to this folder and rename it to ```bios.bin```.

First we will apply microcode patches. These fix processor related bugs and security issues. It is a must to keep these up to date, don't expect much from Dell at this point.

Still in the UBU root folder right click ```UBU.bat``` and run it as admin. It will find the extracted bios and starts to analyse it and taking it apart. Let it do its thing, when its done press any key to continue and you'll be presented with a menu (the file not found message can be ignored).

Next enter ```5``` to enter the microcode section. It will print a table showing the current versions and if there are any updates for it. On mine it looked like this:

```
╔═════════════════════════════════════════╗
║        MC Extractor v1.42.0 r140        ║
╚═════════════════════════════════════════╝
╔═════════════════════════════════════════════════════════════════╗
║                              Intel                              ║
╟─┬─────┬───────────┬────────┬──────────┬────┬──────┬────────┬────╢
║#│CPUID│Platform ID│Revision│   Date   │Type│ Size │ Offset │Last║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼────────┼────╢
║1│40661│ 32 (1,4,5)│   1B   │2019-02-26│PRD │0x6400│0x3CB150│ No ║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼────────┼────╢
║2│40660│ 32 (1,4,5)│FFFF0011│2012-10-12│PRE │0x6400│0x3D1950│Yes ║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼────────┼────╢
║3│306C3│ 32 (1,4,5)│   27   │2019-02-26│PRD │0x5C00│0x3D8150│ No ║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼────────┼────╢
║4│306C2│ 32 (1,4,5)│FFFF0006│2012-10-17│PRE │0x5800│0x3DE150│Yes ║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼────────┼────╢
║5│306C1│ 32 (1,4,5)│FFFF0013│2012-06-14│PRE │0x6000│0x3E3950│ No ║
╚═╧═════╧═══════════╧════════╧══════════╧════╧══════╧════════╧════╝
```

We'll end up in another menu, all we want to enter here is ```F```. It will find and replace anything that can be updated, like this:

```
Choice:F
CPUID 306C3 found.
Files\Intel\mcode\1150\cpu306C3_plat32_ver00000028_2019-11-12_PRD_DBD4CFD1.bin
Checksum correct.
CPUID 306C2 found.
Files\Intel\mcode\1150\cpu306C2_plat32_verFFFF0006_2012-10-17_PRE_30531EB4.bin
Checksum correct.
CPUID 306C1 found.
Files\Intel\mcode\1150\cpu306C1_plat32_verFFFF0014_2012-07-25_PRE_E86E3EB1.bin
Checksum correct.
Generate FFS with Microcode

╔═════════════════════════════════════════╗
║        MC Extractor v1.42.0 r140        ║
╚═════════════════════════════════════════╝
╔═══════════════════════════════════════════════════════════════╗
║                             Intel                             ║
╟─┬─────┬───────────┬────────┬──────────┬────┬──────┬──────┬────╢
║#│CPUID│Platform ID│Revision│   Date   │Type│ Size │Offset│Last║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼──────┼────╢
║1│306C3│ 32 (1,4,5)│   28   │2019-11-12│PRD │0x5C00│ 0x18 │Yes ║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼──────┼────╢
║2│306C2│ 32 (1,4,5)│FFFF0006│2012-10-17│PRE │0x5800│0x5C18│Yes ║
╟─┼─────┼───────────┼────────┼──────────┼────┼──────┼──────┼────╢
║3│306C1│ 32 (1,4,5)│FFFF0014│2012-07-25│PRE │0x6000│0xB418│Yes ║
╚═╧═════╧═══════════╧════════╧══════════╧════╧══════╧══════╧════╝
        These microcodes will be entered into your BIOS file

R - Start replacement
0 - Cancel
Choice:
```

Now we enter ```R``` to replace the old microcodes with the new ones in the BIOS file.

```
Choice:R
        [Preparing for replacement]
BIOS file backup
        [Replacement]
mCode FFS: File replaced
                       Real    Pointer
    _FIT_ Offset - FFA90740 == FFA90740

                       Real    _FIT_
01  mCode Offset - FFDCB150 != FFDC9610
           Fixed -             FFDCB150
      mCode Size -     5C00
02  mCode Offset - FFDD0D50 != FFDCFE10
           Fixed -             FFDD0D50
      mCode Size -     5800
03  mCode Offset - FFDD6550 != FFDD6610
           Fixed -             FFDD6550
      mCode Size -     6000
Changes _FIT_ saved
Press any key to continue . . .
```

After pressing any key we're back in the menu with the table on top. You will notice the table now shows the updated microcodes. We are now done here. Press ```0``` to return to the main menu.

Now there are more things we can update here, one could be very useful but not really required; we can update the SATA/RAID (RST) drivers, though unfortunately it doesn't magically add a RAID controller to 7020 boards. If only!

I have tried [a few versions](https://www.win-raid.com/t596f39-Intel-Management-Engine-Drivers-Firmware-amp-System-Tools.html) and didn't notice any differences, but if you do have a RAID controller there could be benefits. For example using a modified version that allows for RAID0 booting and TRIM from the BIOS itself. You can find a lot more detailed information on the Win-Raid forums. Also note that the latest versions don't always mean best performance.

What I did update were the ethernet firmware and iGPU VBIOS. Remember those Intel GOP/VBIOS/RST files we grabber earlier? We're going to extract the GOP one. Copy the 2 files found in ```\Intel_GOP_VBT_r2\HSW\189``` to ```C:\UBU\Files\Intel\VBIOS```.

UBU should still be open and if you enter ```2``` in the menu now and new menu will appear displaying a current and available section. Verify the available version is newer than your current one and press ```1``` to update those in the BIOS file.

At the time of writing version ```5.5.1034``` was the newest and probably will be forever. Somehow despite the folder the updates were in being called ```189``` the RAW GOP VBT is left at version ```184```. It appears the files only update the GOP driver.

Press any key to return to the main menu and now press ```3``` another current and available menu will appear. Here I could update my ethernet firmware from ```0.0.17``` to ```0.0.27```. PRess ```1``` to update the drivers inside the BIOS.

If you wish you can also try RST drivers, but for now I left those alone until I know a bit more about the impact they may or may not have on performance. I left mine at the default.

We're now done modifying the BIOS, wew.

Press ```0``` to return to the main menu and then press ```0``` again then press ```1``` to save the BIOS as ```mod_bios.bin```.

Time to move on to the next section!

## Flashing
This is the "dangerous" part, it's not really though. But you do have to pay close attention if you want to prevent having to recover from a bad flash.

Enabling service mode requires a jumper on the servcie pins on the motherboard. It disables the write protections to the flash regions. This not needed for extracting, only when flashing.

I think it also possible to remove these restrictions in a modified Grub shell but I wouldn't leave this kind of protections disabled if I were you.

Before you can write anything you have to short the service pins on the motherboard. They are clearly labeled. Use a jumper or some breadboard cables.

Turn the machine off and short the service jumper and turn it back on. You'll get a notice about it and have to press F1 to resume booting.

Once back in Windows it is time to flash the modified BIOS file. Copy ```mod_bios.bin``` from C:\UBI to the WIN64 folder of the Intel ME System Tools Flash Programming Tool.

Navigate to the same folder in a PowerShell running as admin, and execute ```.\fptw64.exe -bios -f mod_bios.bin```. It will start the process right away. Once it's finished execute ``` .\fptw64.exe -greset```. The machine will now reboot and it is recommended to enter the BIOS and load the factor defaults. Make sure you set things up correctly for your hackingtosh config too. Like legacy roms etc. Save and exit the BIOS.

Enjoy your newly modified and updated BIOS. Feels good right?

There is a lot more to update and modify here, but for now we're done here.

> Note: None of these actions will clear the modifications we made in the modified Grub shell. Clearing those can probably only be done with a jumper on the motherboard or with the Intel tools we used earlier or the recovery methods that are there to recover from bad or corrupt updates.

## Windows for those that don't want it
So you're like me and not spend any time in Windows and also don't want to install it on your machine. Well you're in luck! There is a way to create a Windows install that will work on pretty much any computer and runs from a usb drive. Don't use anything too slow or you will be in a serious world of lag and pain because the system keeps using 100% of the disk. Use an old ssd for best results.

You'll need:
* [WinToUSB](https://www.easyuefi.com/wintousb/) - To create the Windows To Go disk
* Windows 10 media, you can grab a eval version directly from Microsoft using [this](https://tb.rg-adguard.net/public.php) website

Create the disk with WinToUSB and boot from it, run Windows Update until there are no more updates.

## KVM/AMT/SPI
Most Dell OptiPlex system have an Intel feature called vPro, this allows for remote management. Even when the computer is turned off! When I went into the MEBx the first to change the password I noticed sometimes KVM/AMT was not an option or could not be enabled. The BIOS was missing some parts. This is not an issue because we can update MEBx so these options become available again.

Being able to login remotely when the machine is turned off and to be able to turn it on, change BIOS settings or install a new OS is pretty cool and if you need manage lots of computers at work much easier on the legs.

Before we get started it we need to make a backup. Navigate to the ```Flash Programming Tool\WIN64``` folder and execute ```.\fptw64.exe -d spi.bin```. This file contains the BIOS and other flash regions, we will not touch the BIOS this time. What we want to do is verify the current MEBx in the image we just made and then if needed update the ME region and only flash the ME region back.

We will also need to download the latest MEBx compatible with our machine. You can update from the 9.0.x to 9.1.x so we are stuck with 9.0.x MEBx and a 9.1.x firmware for it. It can sound a bit confusing, but look at it as MEBx being an operating system. The OS is running version 9.0.x and can only be upgraded within that branch. The firmware the OS uses is at 9.1.x and can only be upgraded within that branch. Maybe it is possible to upgrade but it would involve more risks than benefits. Reprogramming chips in specialised tools are not worth the gains.

1. Go to [this](https://www.win-raid.com/t832f39-Intel-Engine-Firmware-Repositories.html) forum thread and look for the ```B. Intel (Converged Security) Management Engine Firmware Repository``` section, download the file thats linked for 9.1. We'll also need to download the last release of [ME Analyzer](https://github.com/platomav/MEAnalyzer/releases).
2. Open the ```\Intel ME System Tools v9.1 r7\Flash Image Tool\WIN32``` folder and double click ```fitc.exe```. Once open drag your extracted ```spi.bin``` file in it. 
3. From the build menu open build settings and disable ```Generate intermediate build files```. Leave the window open and navigate to ```\Intel ME System Tools v9.1 r7\Flash Image Tool\WIN32\spi\Decomp``` in the folder copy the new firmware downloaded (in my case this file was called *9.1.45.3000_5MB_PRD_RGN.bin*) in step 1 to this folder and rename it to ```ME Region.bin```. If such a file already exists, remove it.
4. Go back to Flash Image Tool ( ```fitc.exe```) that should still be open. Press F5 to build a new image, press yes when asked about a boot profile. Navigate to ```\Intel ME System Tools v9.1 r7\Flash Image Tool\WIN32\Build``` and copy ```outimage.bin``` to ```\Intel ME System Tools v9.1 r7\Flash Programming Tool\WIN64```. Once again navigate to that folder in a PowerShell.
5. We're ready flash the new image execute ```.\fptw64.exe -me -f outimage.bin ```. We tell it to only flash the ME region, this speeds thigns up and leaves the BIOS untouched. A good safety measure as well.
6. Once done flashing execute ```.\fptw64.exe -greset``` and for it to reboot. When it's reboot press F12 to the boot menu and enter MEBx and check if KVM/AMT options are there and can be enabled. If not go back to step 5 and execute the same command minus the -me flag. This will replace everything in the flash. It should not be needed to follow that step.
7. With KVM/AMT enabled you can manage the machine with something like [Meshcommander](https://www.meshcommander.com/meshcommander). And some basics from the a web browser. I'll leave that all to you to explore.

It's a good idea to verify the current settings once done with ```MEInfoWin64.exe```:
```
PS C:\Intel ME System Tools v9.1 r7\MEInfo\WIN64> .\MEInfoWin64.exe

Intel(R) MEInfo Version: 9.1.45.3000
Copyright(C) 2005 - 2017, Intel Corporation. All rights reserved.

Intel(R) Manageability and Security Application code versions:

BIOS Version:                           A18
MEBx Version:                           9.0.0.0029
Gbe Version:                            1.3
VendorID:                               8086
PCH Version:                            4
FW Version:                             9.1.45.3000 H
LMS Version:                            Not Available
MEI Driver Version:                     11.7.0.1032
Wireless Hardware Version:              Not Available
Wireless Driver Version:                Not Available

FW Capabilities:                        0x4DFE5947

    Intel(R) Active Management Technology - PRESENT/ENABLED
    Intel(R) Capability Licensing Service - PRESENT/ENABLED
    Protect Audio Video Path - PRESENT/ENABLED
    Intel(R) Dynamic Application Loader - PRESENT/ENABLED
    Service Advertisement & Discovery - PRESENT/ENABLED

Intel(R) AMT State:                     Enabled
TLS:                                    Enabled
Last ME reset reason:                   Global system reset
Local FWUpdate:                         Enabled
BIOS Config Lock:                       Enabled
```
All looks good, firmware has been updated and after I changed the password from the ```admin``` default to ```uHhvd!sD^8``` the options to enable remote management could be enabled (KVM/AMT). Network by default is unconfigured, you'll need to turn that on too. You can configure the IP manually or using DHCP. I've had mixed resutls with DHCP. Manual setup is best. Pick an IP outside the range your router/DHCP server serves.

## Recovery
It is possible to recover from a bad flash. I will detail the process here in the future.
