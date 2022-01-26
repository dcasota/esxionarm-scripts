# What is ESXi-on-Arm scripts ?
This repo contains a bunch of ESXi-on-Arm related scripts.
ESXi on Arm is a VMware hypervisor system. It runs on arm64 processors eg. on Raspberry Pi hardware. See https://flings.vmware.com/esxi-arm-edition.

# Setting up the hardware
Actually the content is solely related to Raspberry Pi 4B hardware.

## Hardware
Assembled 1x RPi4 8GB, 1x 32GB microSD, 1x 1TB usb stick, 1x 128GB usb stick, 1x 32GB usb stick


## UEFI firmware preparation

On your client (eg. laptop) plug-in the microSD.

Download RPi4_UEFI_Firmware_{release}.zip from https://github.com/pftf/RPi4/releases

Download RPi Imager from https://www.raspberrypi.org/software/

Start RPi Imager > own image > select RPi4_UEFI_Firmware_{release}.zip, write on 32GB microSD.

In your RPi4, plugin now the microSD.

## ESXi on Arm ISO preparation

Download the esxi-on-arm ISO from https://flings.vmware.com/esxi-arm-edition.

Download a tool to write the ISO file to the usb stick eg. rufus.

Start Rufus > select VMware-VMvisor-Installer-7.0.0-19076756.aarch64.iso, write on 32GB usb stick

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
Install ESXi on the plugged-in 128gb usb stick.
### Configure network.
### Configure ssh.

## Configure ESXi

### Configure NTP.

### Configure local.sh
To use the 1TB usb stick as vmfs6 datastore, it isn't automatically recognized during boot. Additionally, sometimes ntp daemon has to be started manually.  

Hence, we add following content to /etc/rc.local.d/local.sh .
```
/etc/init.d/ntpd restart
ntpq -p
/etc/init.d/usbarbitrator stop
partedUtil getptbl /dev/disks <your vml.id>
```
The ```<vml.id>``` must contain the id which can be found by analyzing the output from ```ls /dev/disks```.

# Virtual Machines
Go to the following chapters.
- [arm64 Microsoft Windows 11](https://github.com/dcasota/esxionarm-scripts/tree/main/windows11)
- arm64 VMware Photon OS 4 rev2

