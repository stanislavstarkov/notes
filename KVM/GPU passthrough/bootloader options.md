# TLDR  
```
amd_iommu=on iommu=pt kvm_amd.npt=1 kvm_amd.avic=1 console=ttyUSB0 console=tty vfio-pci.ids=10de:1c03,10de:10f1 video=efifb:off
```  
# Ubuntu

Redact `/etc/default/grub`  


- GRUB_CMDLINE_LINUX_DEFAULT - applied to normal boot
- GRUB_CMDLINE_LINUX - applied to all boot (including rescue mode)

```
root@nas-1:~# cat /etc/default/grub

# If you change this file, run 'update-grub' afterwards to update

# /boot/grub/grub.cfg.

# For full documentation of the options in this file, see:

#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0

GRUB_TIMEOUT_STYLE=hidden

GRUB_TIMEOUT=0


GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`


GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt kvm_amd.npt=1 kvm_amd.avic=1 vfio-pci.ids=10de:1c03,10de:10f1,1022:145f video=efifb:off"

GRUB_CMDLINE_LINUX="console=tty console=ttyS0"


# Uncomment to enable BadRAM filtering, modify to suit your needs

# This works with Linux (no patch required) and with any kernel that obtains

# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)

#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"


# Uncomment to disable graphical terminal (grub-pc only)

#GRUB_TERMINAL=console

# The resolution used on graphical terminal

# note that you can use only modes which your graphic card supports via VBE

# you can see them in real GRUB with the command `vbeinfo'

#GRUB_GFXMODE=640x480

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux

#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries

#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start

#GRUB_INIT_TUNE="480 440 1"
```
Run to apply changes:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

# Sources
- https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
