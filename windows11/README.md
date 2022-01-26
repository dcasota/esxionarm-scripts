# Windows 11 virtual machine on ESXi on Arm on a Raspberry Pi 4B
This is a step-by-step guide how to install a Microsoft Windows 11 virtual machine on ESXi on Arm on a Raspberry Pi 4B.
For ESXi on Arm on a Raspberry Pi 4B have a look to the main chapter.

January 2022:
Actually, Windows 10 and Windows 11 arm64 on ESXi on a raspberry pi4 hardware do not include driver support for network adapter, graphic card, mouse, etc.
There are no VMware Tools aarch64 for Microsoft Windows arm64.  
Not sure why, but the recipes and drivers of 3rd party vendors the all didn't work with the downloaded ISO. Here the links just in case:  
https://rpi4-uefi.dev/windows-10-drivers/, https://github.com/worproject/RPi-Windows-Drivers, https://github.com/SuperJMN/Deployer  
In addition, Windows 11 ISO setup on ESXi on Arm v1.8 does not start successfully.

A functioning, networkless setup is first to install a Windows 10 virtual machine, and do an inplace update to windows 11 afterwards.

# Setting up the ISOs
Windows 11 on Arm64 is available as preview, see https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewARM64.
The download bits as .vhdx image format isn't supported on ESXi on Arm, hence we cannot use this download method.

Go to [uupdump.net], enter "windows 10 arm64" on the search bar, specify arm64 as architecture and specify to download the content as ISO.
You will download a zip file eg. 19044.1499_arm64_en-us_professional_55c91dc2_convert.zip. Unzip it.
Now start uup_download_windows.cmd, or uup_download_macos.sh or uupd_download_linux.sh.
The script takes a while, but at the end you get a dynamically created ISO eg. 19041.1.191206-1406.VB_RELEASE_CLIENTPRO_OEMRET_A64FRE_EN-US.ISO.

Now do the same with "windows 11 arm64". You will download a zip file eg. 22538.1000_arm64_en-us_professional_5813c380_convert.zip. Unzip it.
Now start uup_download_windows.cmd, or uup_download_macos.sh or uupd_download_linux.sh.
The script takes again a while, but at the end you get a dynamically created ISO eg. 22538.1000.220114-1500.RS_PRERELEASE_CLIENTPRO_OEMRET_A64FRE_EN-US.ISO.

Windows 11 arm64 wouldn't start because of compatibility checks.
To have a modified Windows 11 installation ISO to bypass compatibility checks both pre and post installation, see https://github.com/JosephM101/Force-Windows-11-Install.

Copy both ISO to an ESXi datastore.

# Create and start the Windows guest virtual machine
The specs are:
virtual hardware ESXi 6.7 or higher (v13 ++), guest os Windows, guest os version: microsoft windows 10 (64-bit).
2 CPU, 4GB RAM, 55GB harddisk on SATA controller 0:0, SATA Controller 0, USB controller 1 with USB3.1 support, Network adapter with vmxnet3 adapter type,
CD/DVD driver 1 on SATA controller 0 as SATA 0:1 with connected (+ connect at power on) 19041.1.191206-1406.VB_RELEASE_CLIENTPRO_OEMRET_A64FRE_EN-US.ISO,
Video card (default settings).

Start your virtual machine, and go through the Windows 10 setup process.
Do use the configuration options of 'no network'.

# In-Place update to Windows 11

In windows registry add following entries:  
```
HKEY_LOCAL_MACHINE\SYSTEM\Setup\MoSetup
HKEY_LOCAL_MACHINE\SYSTEM\Setup\MoSetup\AllowUpgradesWithUnsupportedTPMOrCPU, type dword32, value 1
HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig
HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig\BypassTPMCheck, type dword32, value 1
HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig\BypassSecureBootCheck, type dword32, value 1
HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig\BypassRAMCheck, type dword32, value 1
HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig\BypassStorageCheck, type dword32, value 1
HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig\BypassCPUCheck, type dword32, value 1
```

Detach the Windows 10 ISO, and attach the Windows 11 ISO.
In the vm, go to the CD/DVD drive, and start in \sources\setup.exe.
Choose customize during the presetup phase.

# Post configuration

Download and install [7-zip 64-bit Windows arm64](7-zip.org).
