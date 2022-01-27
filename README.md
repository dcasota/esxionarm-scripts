<p align="center"><img src="https://user-images.githubusercontent.com/14890243/151339098-f9da9453-b3f9-4f00-939a-802f17793f4b.png" width="300" height="200"></p>
    
# What is ESXi-on-Arm scripts ?
This repo is dedicated to ESXi-on-Arm related homelab tinkering.
ESXi on Arm is a VMware project name for a hypervisor system. It runs ESXi on arm64 processors eg. on Raspberry Pi hardware. See https://flings.vmware.com/esxi-arm-edition.

# Setting up the hardware
Actually this content is solely related to Raspberry Pi 4B hardware.

## Hardware

Bill of material
- 1x 32GB microSD for the RPI4 uefi firmware
- 1x 32GB usb stick for the arm64 ESXi install bits
- 1x 128 GB usb stick for the arm64 ESXi installation
- 1x 1 TB usb stick with an existing vmfs6 datastore
- network cable, connectivity eg. to a laptop with installed web browser
- micro hdmi attached monitor
- keyboard
- rpi4 power supply

## UEFI firmware preparation

On your client (eg. laptop) plug-in the microSD.

Download RPi4_UEFI_Firmware_{release}.zip from https://github.com/pftf/RPi4/releases

Download RPi Imager from https://www.raspberrypi.org/software/

Start RPi Imager > own image > select RPi4_UEFI_Firmware_{release}.zip, write on 32GB microSD.

In your RPi4, plugin now the microSD.

## ESXi on Arm ISO preparation

Download the esxi-on-arm ISO from https://flings.vmware.com/esxi-arm-edition.

Download a tool to write the ISO file to the usb stick eg. rufus.

Start Rufus > select VMware-VMvisor-Installer-7.0.0-19076756.aarch64.iso, and write the bits on the 32GB usb stick.

## Bios preparation

Plugin the 32GB usb stick on an USB2 port.
Plugin the 128GB usb stick on an USB3 port.
Power on.

Configure BIOS:
- Disable 3GB limitation.
- Configure fan.
- Remove boot options boot from any network. Configure to boot from USB stick.

Now boot from the usb media.

## Install ESXi
You should now have two plugged-in two usb-sticks, a micro HDMI cable, a keyboard and a network cable.

Install ESXi on the plugged-in 128gb usb stick.

ESXi automatically runs with DHCP if available or with APIPA. You can configure a static ip address as well.
The ip is presented on the screen eg. http://192.168.0.100.

## Configure ESXi

On your web browser, go to the ip address and login.
Navigate to Host > Manage > Tab: Services > start TSM-SSH.

### Configure NTP

### Configure local.sh
In my scenario, the 1TB usb stick contains an existing vmfs6 datastore. It isn't automatically recognized during boot. Additionally, sometimes ntp daemon has to be started manually. Hence, we add following content to /etc/rc.local.d/local.sh .
```
/etc/init.d/ntpd restart
ntpq -p
/etc/init.d/usbarbitrator stop
partedUtil getptbl /dev/disks <your vml.id>
```
The ```<vml.id>``` must contain the id which can be found by analyzing the output from ```ls /dev/disks```.

## Configure native hardware status driver for ESXi-Arm on Raspberry Pi
There is no built-in driver for monitoring the hardware status of the Raspberry Pi.
Having done some research, it seems that there are some open source initiatives, but none has reached a stable and 'install friendly' level.
A promising one is https://github.com/thebel1/thpimon.
Download the .vib from URL and copy it to your datastore eg. /vmfs/volumes/sandisk.
```
[root@localhost:~] esxcli software acceptance set --level=CommunitySupported
Host acceptance level changed to 'CommunitySupported'.
[root@localhost:~] esxcli software vib install -v /vmfs/volumes/sandisk/thgpio-0.1.0-1OEM.701.1.0.40650718.aarch64.vib
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: THX_bootbank_thgpio_0.1.0-1OEM.701.1.0.40650718
   VIBs Removed:
   VIBs Skipped:
[root@localhost:~]
```

## Configure native GPIO driver for ESXi-Arm on Raspberry Pi
To pass the GPIO signal(s) to a virtual machine, there is a community supported solution.
Download from URL https://github.com/thebel1/thgpio and copy the .vib to your datastore eg. /vmfs/volumes/sandisk.
```
[root@localhost:~] esxcli software acceptance set --level=CommunitySupported
Host acceptance level changed to 'CommunitySupported'.
[root@localhost:~] esxcli software vib install -v /vmfs/volumes/sandisk/thgpio-0.1.0-1OEM.701.1.0.40650718.aarch64.vib
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: THX_bootbank_thgpio_0.1.0-1OEM.701.1.0.40650718
   VIBs Removed:
   VIBs Skipped:
[root@localhost:~]
```


# Virtual Machines
Go to the following chapters.
- [arm64 Microsoft Windows 11](https://github.com/dcasota/esxionarm-scripts/tree/main/windows11)
- arm64 VMware Photon OS 4 rev2

