[[_TOC_]]  
>The best source of info: https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
###### Check virtualization support

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```
output:

```
16
```

Other command

```bash
kvm-ok
```

```
INFO: /dev/kvm exists
KVM acceleration can be used
```  

###### Check IOMMU support
```bash
dmesg | grep -i -e DMAR -e IOMMU

[    0.000000] ACPI: DMAR 0x00000000BDCB1CB0 0000B8 (v01 INTEL  BDW      00000001 INTL 00000001)
[    0.000000] Intel-IOMMU: enabled
[    0.028879] dmar: IOMMU 0: reg_base_addr fed90000 ver 1:0 cap c0000020660462 ecap f0101a
[    0.028883] dmar: IOMMU 1: reg_base_addr fed91000 ver 1:0 cap d2008c20660462 ecap f010da
[    0.028950] IOAPIC id 8 under DRHD base  0xfed91000 IOMMU 1
[    0.536212] DMAR: No ATSR found
[    0.536229] IOMMU 0 0xfed90000: using Queued invalidation
[    0.536230] IOMMU 1 0xfed91000: using Queued invalidation
[    0.536231] IOMMU: Setting RMRR:
[    0.536241] IOMMU: Setting identity map for device 0000:00:02.0 [0xbf000000 - 0xcf1fffff]
[    0.537490] IOMMU: Setting identity map for device 0000:00:14.0 [0xbdea8000 - 0xbdeb6fff]
[    0.537512] IOMMU: Setting identity map for device 0000:00:1a.0 [0xbdea8000 - 0xbdeb6fff]
[    0.537530] IOMMU: Setting identity map for device 0000:00:1d.0 [0xbdea8000 - 0xbdeb6fff]
[    0.537543] IOMMU: Prepare 0-16MiB unity mapping for LPC
[    0.537549] IOMMU: Setting identity map for device 0000:00:1f.0 [0x0 - 0xffffff]
[    2.182790] [drm] DMAR active, disabling use of stolen memory
```  

###### Check IOMMU groups

Only full group could be passed through
```bash
#!/bin/bash
shopt -s nullglob
for g in /sys/kernel/iommu_groups/*; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```  
Example output:
```bash
IOMMU Group 0:
	00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 1:
	00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 10:
	00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B [1022:1454]
IOMMU Group 11:
	00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 59)
	00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 12:
	00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 0 [1022:1460]
	00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 1 [1022:1461]
	00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 2 [1022:1462]
	00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 3 [1022:1463]
	00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 4 [1022:1464]
	00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 5 [1022:1465]
	00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 6 [1022:1466]
	00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 7 [1022:1467]
IOMMU Group 13:
	01:00.0 Non-Volatile memory controller [0108]: Phison Electronics Corporation E12 NVMe Controller [1987:5012] (rev 01)
IOMMU Group 14:
	02:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset USB 3.1 XHCI Controller [1022:43d5] (rev 01)
	02:00.1 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset SATA Controller [1022:43c8] (rev 01)
	02:00.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Bridge [1022:43c6] (rev 01)
	03:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
	03:01.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
	03:04.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
	04:00.0 USB controller [0c03]: Renesas Technology Corp. uPD720201 USB 3.0 Host Controller [1912:0014] (rev 03)
	05:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)
	06:00.0 Serial Attached SCSI controller [0107]: Broadcom / LSI SAS2008 PCI-Express Fusion-MPT SAS-2 [Falcon] [1000:0072] (rev 03)
IOMMU Group 15:
	07:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP106 [GeForce GTX 1060 6GB] [10de:1c03] (rev a1)
	07:00.1 Audio device [0403]: NVIDIA Corporation GP106 High Definition Audio Controller [10de:10f1] (rev a1)
IOMMU Group 16:
	08:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Zeppelin/Raven/Raven2 PCIe Dummy Function [1022:145a]
IOMMU Group 17:
	08:00.2 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Platform Security Processor [1022:1456]
IOMMU Group 18:
	08:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Zeppelin USB 3.0 Host controller [1022:145f]
IOMMU Group 19:
	09:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Zeppelin/Renoir PCIe Dummy Function [1022:1455]
IOMMU Group 2:
	00:01.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 20:
	09:00.2 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
IOMMU Group 21:
	09:00.3 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) HD Audio Controller [1022:1457]
IOMMU Group 3:
	00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 4:
	00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 5:
	00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 6:
	00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 7:
	00:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 8:
	00:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B [1022:1454]
IOMMU Group 9:
	00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
```
###### Show CPU topology

`lstopo` is part of `hwloc` package  

```bash
root@nas-1:~# lstopo --of ascii -p
┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Machine (63GB total)                                                                                                                       │
│                                                                                                                                            │
│ ┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐ │
│ │ Package P#0                                                                                                                            │ │
│ │                                                                                                                                        │ │
│ │ ┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐ │ │
│ │ │ NUMANode P#0 (63GB)                                                                                                                │ │ │
│ │ └────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘ │ │
│ │                                                                                                                                        │ │
│ │ ┌────────────────────────────────────────────────────────────────┐  ┌────────────────────────────────────────────────────────────────┐ │ │
│ │ │ L3 (8192KB)                                                    │  │ L3 (8192KB)                                                    │ │ │
│ │ └────────────────────────────────────────────────────────────────┘  └────────────────────────────────────────────────────────────────┘ │ │
│ │                                                                                                                                        │ │
│ │ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │ │
│ │ │ L2 (512KB)  │  │ L2 (512KB)  │  │ L2 (512KB)  │  │ L2 (512KB)  │  │ L2 (512KB)  │  │ L2 (512KB)  │  │ L2 (512KB)  │  │ L2 (512KB)  │ │ │
│ │ └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │ │
│ │                                                                                                                                        │ │
│ │ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │ │
│ │ │ L1d (32KB)  │  │ L1d (32KB)  │  │ L1d (32KB)  │  │ L1d (32KB)  │  │ L1d (32KB)  │  │ L1d (32KB)  │  │ L1d (32KB)  │  │ L1d (32KB)  │ │ │
│ │ └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │ │
│ │                                                                                                                                        │ │
│ │ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │ │
│ │ │ L1i (64KB)  │  │ L1i (64KB)  │  │ L1i (64KB)  │  │ L1i (64KB)  │  │ L1i (64KB)  │  │ L1i (64KB)  │  │ L1i (64KB)  │  │ L1i (64KB)  │ │ │
│ │ └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │ │
│ │                                                                                                                                        │ │
│ │ ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │ │
│ │ │ Core P#0    │  │ Core P#1    │  │ Core P#2    │  │ Core P#3    │  │ Core P#4    │  │ Core P#5    │  │ Core P#6    │  │ Core P#7    │ │ │
│ │ │             │  │             │  │             │  │             │  │             │  │             │  │             │  │             │ │ │
│ │ │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │ │ │
│ │ │ │ PU P#0  │ │  │ │ PU P#1  │ │  │ │ PU P#2  │ │  │ │ PU P#3  │ │  │ │ PU P#4  │ │  │ │ PU P#5  │ │  │ │ PU P#6  │ │  │ │ PU P#7  │ │ │ │
│ │ │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │ │ │
│ │ │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │ │ │
│ │ │ │ PU P#8  │ │  │ │ PU P#9  │ │  │ │ PU P#10 │ │  │ │ PU P#11 │ │  │ │ PU P#12 │ │  │ │ PU P#13 │ │  │ │ PU P#14 │ │  │ │ PU P#15 │ │ │ │
│ │ │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │ │ │
│ │ └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │ │
│ └────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                                                                            │
│ ├┤╶─┬─────┼┤╶───────┬───────────────────┐                                                                                                  │
│     │3.9       3.9  │ PCI 01:00.0       │                                                                                                  │
│     │               │                   │                                                                                                  │
│     │               │ ┌───────────────┐ │                                                                                                  │
│     │               │ │ Block nvme0n1 │ │                                                                                                  │
│     │               │ │               │ │                                                                                                  │
│     │               │ │ 931 GB        │ │                                                                                                  │
│     │               │ └───────────────┘ │                                                                                                  │
│     │               └───────────────────┘                                                                                                  │
│     │                                                                                                                                      │
│     ├─────┼┤╶─┬─────┬─────────────┐                                                                                                        │
│     │3.9      │3.9  │ PCI 02:00.1 │                                                                                                        │
│     │         │     └─────────────┘                                                                                                        │
│     │         │                                                                                                                            │
│     │         └─────┼┤╶─┬─────┼┤╶───────┬────────────────┐                                                                                 │
│     │          3.9      │0.2       0.2  │ PCI 05:00.0    │                                                                                 │
│     │                   │               │                │                                                                                 │
│     │                   │               │ ┌────────────┐ │                                                                                 │
│     │                   │               │ │ Net enp5s0 │ │                                                                                 │
│     │                   │               │ └────────────┘ │                                                                                 │
│     │                   │               └────────────────┘                                                                                 │
│     │                   │                                                                                                                  │
│     │                   └─────┼┤╶───────┬─────────────────────────────────────────────┐                                                    │
│     │                    2.0       2.0  │ PCI 06:00.0                                 │                                                    │
│     │                                   │                                             │                                                    │
│     │                                   │ ┌───────────┐  ┌───────────┐  ┌───────────┐ │                                                    │
│     │                                   │ │ Block sdd │  │ Block sdb │  │ Block sde │ │                                                    │
│     │                                   │ │           │  │           │  │           │ │                                                    │
│     │                                   │ │ 3726 GB   │  │ 186 GB    │  │ 3726 GB   │ │                                                    │
│     │                                   │ └───────────┘  └───────────┘  └───────────┘ │                                                    │
│     │                                   │                                             │                                                    │
│     │                                   │ ┌───────────┐  ┌───────────┐                │                                                    │
│     │                                   │ │ Block sdc │  │ Block sda │                │                                                    │
│     │                                   │ │           │  │           │                │                                                    │
│     │                                   │ │ 3726 GB   │  │ 3726 GB   │                │                                                    │
│     │                                   │ └───────────┘  └───────────┘                │                                                    │
│     │                                   └─────────────────────────────────────────────┘                                                    │
│     │                                                                                                                                      │
│     ├─────┼┤╶───────┬─────────────┐                                                                                                        │
│     │4.0       4.0  │ PCI 07:00.0 │                                                                                                        │
│     │               └─────────────┘                                                                                                        │
│     │                                                                                                                                      │
│     └─────┼┤╶───────┬─────────────┐                                                                                                        │
│      16        16   │ PCI 09:00.2 │                                                                                                        │
│                     └─────────────┘                                                                                                        │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Host: nas-1                                                                                                                                │
│                                                                                                                                            │
│ Indexes: physical                                                                                                                          │
│                                                                                                                                            │
│ Date: Fri Dec 25 20:36:22 2020                                                                                                             │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```  
