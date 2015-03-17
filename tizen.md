# Installation
## Download and Installation
* http://download.tizen.org/snapshots/tizen/common/latest/images/
* https://wiki.tizen.org/wiki/Common_Install


## Setting up
### Disable VGA output on VTC1010
The hardware design of the Nexcom VTC1010 reports an active VGA port even if none is connected. Add the following to the boot loader kernel option command line to disable this 'ghost' screen: video=VGA-1:d
Open /boot/extlinux/extlinux.conf for the MBR image
Open /boot/loader/entries/vmlinuz-<VERSION>-x86-ivi.conf for the EFI image


## Links
 * https://wiki.tizen.org/wiki/IVI/Enable_Logging
 * https://wiki.tizen.org/wiki/IVI/Modello
 * https://wiki.tizen.org/wiki/Common
