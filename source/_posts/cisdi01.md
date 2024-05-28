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
![01](./img/cisdi01/hikServer01.png)

Press ```e``` and then modify as shown:
![02](./img/cisdi01/hikServer02.png)

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

# conf

- [Apollo](https://www.apolloconfig.com/#/zh/design/apollo-introduction)


