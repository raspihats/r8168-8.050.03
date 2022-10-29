
# Linux device driver for Realtek Ethernet Gigabit controllers

This driver, GBE Ethernet LINUX driver r8168 for kernel up to 5.17, was downloaded from: https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software

The following paragraph is from the download page:

Network Interface Controllers > 10/100/1000M Gigabit Ethernet > PCI Express
 - RTL8111B/RTL8111C/RTL8111D/RTL8111E/RTL8111F/RTL8111G(S)/RTL8111H(S)//RTL8118(A)(S)/RTL8119i/RTL8111L/RTL8111K
 - RTL8168B/RTL8168E/RTL8168H
 - RTL8111DP/RTL8111EP/RTL8111FP
 - RTL8411/RTL8411B

 This is the Linux device driver released for RealTek RTL8168B/8111B, RTL8168C/8111C, RTL8168CP/8111CP, RTL8168D/8111D, RTL8168DP/8111DP, and RTL8168E/8111E Gigabit Ethernet controllers with PCI-Express interface.

 This driver was modified to work with the LEDs on the SGII-CM4CB.

 After building and installing this driver, see below, you may be needing to backlist the r8169 driver that comes with the latest Raspberry Pi OS kernel like so:

 `sudo nano /etc/modprobe.d/raspi-blacklist.conf`

 And add the following line:

 `backlist r8169`

## Requirements

- Kernel source tree (supported Linux kernel 2.6.x and 2.4.x)
- For linux kernel 2.4.x, this driver supports 2.4.20 and latter.
- Compiler/binutils for kernel compilation

## Quick install with proper kernel settings

Unpack the tarball :

`tar vjxf r8168-8.aaa.bb.tar.bz2`

Change to the directory:

`cd r8168-8.aaa.bb`

If you are running the target kernel, then you should be able to do :

`./autorun.sh (as root or with sudo)`

You can check whether the driver is loaded by using following commands.

`lsmod | grep r8168`
`ifconfig -a`

If there is a device name, ethX, shown on the monitor, the linux
driver is loaded. Then, you can use the following command to activate
the ethX.

`ifconfig ethX up`
,where X=0,1,2,...

## Set the network related information
1. Set manually:
 a. Set the IP address of your machine.
	`ifconfig ethX "the IP address of your machine"`
 b. Set the IP address of DNS.
   Insert the following configuration in /etc/resolv.conf.
   nameserver "the IP address of DNS"
 c. Set the IP address of gateway.
   route add default gw "the IP address of gateway"

2. Set by doing configurations in /etc/sysconfig/network-scripts /ifcfg-ethX for Redhat and Fedora, or /etc/sysconfig/network /ifcfg-ethX for SuSE. There are two examples to set network configurations.
 a. Fixed IP address:
  DEVICE=eth0
  BOOTPROTO=static
  ONBOOT=yes
  TYPE=ethernet
  NETMASK=255.255.255.0
  IPADDR=192.168.1.1
  GATEWAY=192.168.1.254
  BROADCAST=192.168.1.255
	
 b. DHCP:
  DEVICE=eth0
  BOOTPROTO=dhcp
  ONBOOT=yes

## Modify the MAC address

There are two ways to modify the MAC address of the NIC.

1. Use ifconfig:
`ifconfig ethX hw ether YY:YY:YY:YY:YY:YY`
,where X is the device number assigned by Linux kernel, and YY:YY:YY:YY:YY:YY is the MAC address assigned by the user.

2. Use ip:
`ip link set ethX address YY:YY:YY:YY:YY:YY`
,where X is the device number assigned by Linux kernel, and YY:YY:YY:YY:YY:YY is the MAC address assigned by the user.

## Force Link Status

1. Force the link status when insert the driver.
If the user is in the path ~/r8168, the link status can be forced to one of the 5 modes as following command.
`insmod ./src/r8168.ko speed=SPEED_MODE duplex=DUPLEX_MODE autoneg=NWAY_OPTION`
,where:
	SPEED_MODE = 
	- 1000 for 1000Mbps
	- 100 for 100Mbps
	- 10 for 10Mbps

	DUPLEX_MODE =
	- 0 for half-duplex
	- 1 for full-duplex
	
	NWAY_OPTION = 
	- 0 for auto-negotiation off (true force)
	- 1 for auto-negotiation on (nway force)

	For example:
	`insmod ./src/r8168.ko speed=100 duplex=0 autoneg=1`
	will force PHY to operate in 100Mpbs Half-duplex(nway force).

2. Force the link status by using ethtool.
a. Insert the driver first.
b. Make sure that ethtool exists in /sbin.
c. Force the link status as the following command.
`ethtool -s ethX speed SPEED_MODE duplex DUPLEX_MODE autoneg NWAY_OPTION`
,where:
	SPEED_MODE = 
	- 1000 for 1000Mbps
	- 100 for 100Mbps
	- 10 for 10Mbps
	
	DUPLEX_MODE = 
	- half for half-duplex
	- full for full-duplex
	
	NWAY_OPTION = 
	- off for auto-negotiation off (true force)
	- on for auto-negotiation on (nway force)

	For example:
	`ethtool -s eth0 speed 100 duplex full autoneg on`
	will force PHY to operate in 100Mpbs Full-duplex(nway force).

## Jumbo Frame
Transmitting Jumbo Frames, whose packet size is bigger than 1500 bytes, please change mtu by the following command.

`ifconfig ethX mtu MTU`
, where X=0,1,2,..., and MTU is configured by user.

RTL8168B/8111B supports Jumbo Frame size up to 4 kBytes.
RTL8168C/8111C and RTL8168CP/8111CP support Jumbo Frame size up to 6 kBytes.
RTL8168D/8111D supports Jumbo Frame size up to 9 kBytes.

