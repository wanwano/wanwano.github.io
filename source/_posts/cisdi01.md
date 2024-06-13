---
title: cisdi01
date: 2024-05-24 17:08:56
tags: cisdi
---

# env
- [scoop](https://github.com/ScoopInstaller/Scoop) yyds!
- [Jetbrains](https://www.jetbrains.com/) + tool

# server
Unbox a Hikvision server, **DS-VE22S-B**.

### Hikvisionos

After going through looking for VGA, looking for VGA to HDMI adapter, looking for adapter power supply line, finally boot.

The server is pre-installed Hikvisionos, the rip-off is that the staff do not know the initial account password, after trying to reset the password.

Enter single-user mode to reset:
![01](/img/cisdi01/hikServer01.png)

Press ```e``` and then modify as shown:
![02](/img/cisdi01/hikServer02.png)

Then run ```exec /sbin/init``` to return to multi-user mode and log in again with the new password.

However, I was asked to reinstall a non-custom CentOS system, which was done for nothing.

### USB not recognized

After the production of the USB boot disk, once again encountered an outrageous problem: *dracut-initqueue timeout.*

USB flash drive is not recognized, use ```blkid``` to query the current mounting directory of USB flash drive, and press ```e``` again in installation mode to enter editing mode:
Modified ```hd:LABEL=CentOS\... quiet``` to the ```hd:/mount directory quiet```.

### Network configuration

The purpose is to connect to the same switch computer can remotely log in to the server, the switch is not connected to the router, that is, can not connect to the external network.

So instead of using DHCP, we manually configured the static ip addresses of the servers and computers.```nmtui```,```nmcli```, or directly modify the configuration file.
```
addr: 192.168.1.2
mask: 255.255.255.0
gateway: 192.168.1.1
```

Ahh... Sudden change, and touch out an AC650 wireless USB, to let the server connect to the external network. Damn TP Link doesn't provide Linux drivers. After searching, really let me find, thanks to the [elder brother](https://github.com/brektrou/rtl8821CU) to provide the drive.

This driver can only be used on Linux 4.4 kernel, according to my tests, too high or too low version will not work. Damn officials don't maintain old RPMS.

Re-create the kernel configuration after installation. Run ```grub2-mkconfig -o /boot/grub2/grub.cfg```, and then ```reboot``` to see kernel version changes using ```uname -r```.

Also, run ```rpm -qa | grep kernel``` to see which kernels are installed, and use ```yum remove``` to remove the useless ones.

After downloading the driver, we execute

```
make
sudo make install
sudo modprobe 8821cu
```

Run the ```lsusb``` command to view the number of the NIC and switch the mode of the USB NIC to the wireless NIC mode:
```sudo usb_modeswitch -KW -v 0bda -p 1a2b```.

# knowledge

- [Apollo](https://www.apolloconfig.com/#/zh/design/apollo-introduction)
- [Live streaming protocol](https://www.cnblogs.com/yangchin9/p/18153222)
- Principles of lambda, method references, closures, and stream 
    *   **Generate classes, objects, and methods**


