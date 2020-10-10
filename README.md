# Flashlight - Kaloom Fabric Hardware Diagnostic utility

Flashlight is a utility used for gathering fabric hardware node and generates reports.

This utility runs in two modes:

* Post installation data collection - this is the default mode and runs against an installed Kaloom fabric
* Pre installation data collection - this mode is usually used to collect data ahead of Kaloom fabric installation. It requires node-probe OS to be installed on all target nodes.

# Usage

```
check target Fabric hardware resources

Usage:
  flashlight check [flags]

Aliases:
  check, rake

Flags:
  -a, --all                             Collect all hardware info
  -f, --fabric-definition-file string   Path to fabric definition file (required)
  -h, --help                            help for check
  -n, --node-probe                      Collect data from node-probe os platform
  -s, --split                           Generate split table report; smaller screens friendly (default true)

```

Example:

* Simple mode:
```bash
flashlight check -f /opt/kaloom/fabric_definition.yaml
```

* Node-probe mode, requires in memory node-probe OS deployment


```bash
flashlight check --node-probe --fabric-definition-file /opt/kaloom/fabric_definition.yaml
```

![Output](example.jpg)

#  Tools and commands used to collect data

Examples below might not necessary reflect exact options used to get the actual data. For instance,  in many cases a
linux tool is used to get target information where the actual info is just a substring of returned output.
Sometimes linux tools like awk are used to filter out target info, on other occasions golang is used to process output
data as well. 

Please, use description below as reference only.


- Host MAC address:
```bash
sudo ip link show eth0
```

- IPMI availability check:
```bash
ipmi-ping <node-host-ip> -c 1 -t 1
```

- BMC MAC Address:
```bash
onieDisk=$(sudo /sbin/blkid | grep ONIE-BOOT | awk '{print $1}')
sudo mkdir -p /mnt/onie-boot; sudo mount ${onieDisk} /mnt/onie-boot ; sudo cat /mnt/onie-boot/onie-sys* ; sudo umount /mnt/onie-boot; sudo rmdir /mnt/onie-boot
```

- Bios version:
```bash
sudo dmidecode -s bios-version
```

- BIOS release date:
```bash
sudo dmidecode -s bios-release-date
```

- ONIE version:
```bash
onieDisk=$(sudo /sbin/blkid | grep ONIE-BOOT | awk '{print $1}')
sudo mkdir -p /mnt/onie-boot; sudo mount ${onieDisk} /mnt/onie-boot ; sudo cat /mnt/onie-boot/onie-sys* ; sudo umount /mnt/onie-boot; sudo rmdir /mnt/onie-boot
```

- System manufacturer:<br>
Depends on manufacturer there two ways were used, info from BIOS or ONIE

```bash
sudo dmidecode -s system-manufacturer
# When ONIE available 
onieDisk=$(sudo /sbin/blkid | grep ONIE-BOOT | awk '{print $1}')
sudo mkdir -p /mnt/onie-boot; sudo mount ${onieDisk} /mnt/onie-boot ; sudo cat /mnt/onie-boot/onie-sys* ; sudo umount /mnt/onie-boot; sudo rmdir /mnt/onie-boot
```

- Product Name:<br>
Depends on manufactured there two ways were used, from BIOS or ONIE

```bash
sudo dmidecode -s system-product-name
# When ONIE available 
onieDisk=$(sudo /sbin/blkid | grep ONIE-BOOT | awk '{print $1}')
sudo mkdir -p /mnt/onie-boot; sudo mount ${onieDisk} /mnt/onie-boot ; sudo cat /mnt/onie-boot/onie-sys* ; sudo umount /mnt/onie-boot; sudo rmdir /mnt/onie-boot
```

- Boot Mode - UEFI or BIOS:<br>
Checking OS for /sys/firmware/efi presence
```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

- Boot sequence - Current Boot:<br>
Option available when UEFI is enabled only
```bash
sudo efibootmgr
```

- Boot sequence - Second Boot:<br>
Option available when UEFI is enabled only
```bash
sudo efibootmgr
```

- System total RAM:<br>
```bash
sudo grep "MemTotal" /proc/meminfo| awk '{print $2}'
```

- System number of CPUs:
```bash
grep processor /proc/cpuinfo | wc -l
```

- System available disks:
```bash
sudo /usr/sbin/parted -l | grep 'Disk /dev' | grep -v 'Disk /dev/mapper' | awk '{print $2, $3}'
```

- System hardware clock:
```bash
sudo hwclock | awk '{print $1, $2, $3, $4, $5, $6, $7}'
```

- Check Controller's 25GbE NIC availability:
```bash
sudo lspci  | grep Ethernet  | grep 25GbE  | awk '{print $4,$(NF-5),$(NF-3)}' |sort
```

- Check Controller's NIC driver:
```bash
sudo ls /sys/class/net/p*/device/driver/module/drivers/ | grep pci | awk  -F  \":\" '{print $2}' |sort
```

- Barefoot Driver:
```bash
sudo modinfo bf_kdrv
```

- Barefoot SDE:
```bash
telnet <node-host-ip> 9999
> ver
(bf-syslibs/bf-utils/bf-drivers)
```

- BMC version:<br>
OpenBMC OS or ipmitool are used

```bash
sudo cat /etc/issue
# when openBMC is not available:
ipmitool -I lanplus -H <controller-management-IP> -U <username> -P <password> lan print
```