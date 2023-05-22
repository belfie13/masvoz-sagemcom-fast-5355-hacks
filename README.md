# masvoz-sagemcom-fast-5355-hacks

Research on Sagemcom F@ST 5355 router


## Table of contents

1. [Introduction](#introduction%C3%B3n)
2. [Operators that offer the router](#operators-that-offer-the-router)
3. [Router manuals](#router-manuals)
4. [Firmware](#firmware)
5. [Hardware](#hardware)
6. [SSH access](#ssh-access)
7. [USB Storage](#usb-storage)
8. [GPON Password](#gpon-password)
9. [Initial and current configuration](#initial-and-current-configuration)
10. [SIP Configuration](#configuraci%C3%B3n-sip)
11. [Admin password](#contrase%C3%B1a-admin)
12. [DHCP](#dhcp)
12. [References](#references)


## Introduction

I am no hacker or security expert. This little research is a little part of me (point 3 is entirely mine) and a lot of user research put in order (you can see the sources used in chapter 4. References)


## Operators offering the router

This router is offered to all operators working on MasVoz networks. Currently they are:

- MasVoz
- Pepephone
- Yoigo


## Router manuals

There is no official user manual for MasVoz and cia so don't go crazy looking for it (if someone is encouraged to do it, I'll post it here to help other users).


## Firmware

```
    root@home:/# cat /proc/version
    Linux version 3.4.11-rt19 (g503707@shz-p0000665fl) (gcc version 4.6.2 (GCC) ) #21 SMP PREEMPT Wed Oct 17 14:51:30 CST 2018
```

```
    root@home:/# cat /proc/mtd
    dev: size erasesize name
    mtd0: 00020000 00020000 "nvram"
    mtd1: 000a0000 00020000 "bcm"
    mtd2: 00500000 00020000 "data"
    mtd3: 07a40000 00020000 "ubi
    mtd4: 00086800 0001f000 "secondaryboot" mtd5: 00104340 00020000 "secondaryboot"
    mtd5: 00104340 0001f000 "uboot
    mtd6: 00104340 0001f000 "uboot-rescue
    mtd7: 000019e8 0001f000 "permanent_param"
    mtd8: 018d3000 0001f000 "rescue"
    mtd9: 0175f000 0001f000 0001f000 "operational
    mtd10: 0001f000 0001f000 0001f000 "firm_header"
    mtd11: 0028b000 0001f000 0001f000 "kernel"
    mtd12: 014d4000 0001f000 0001f000 "rootfs"
    mtd13: 007c0000 0001f000 "filesystem1"
    mtd14: 009b0000 0001f000 "filesystem2"
```


## Hardware

```
    root@home:/# cat /proc/cpuinfo
    system type : F@ST5655_V2
    processor : 0
    cpu model : Broadcom BMIPS4350 V8.0
    BogoMIPS : 598.01
    wait instruction : yes
    microsecond timers : yes
    tlb_entries : 32
    extra interrupt vector : no
    hardware watchpoint : no
    ASEs implemented :
    shadow register sets : 1
    kscratch registers : 0
    core : 0
    VCED exceptions : not available
    VCEI exceptions : not available

    processor : 1
    cpu model : Broadcom BMIPS4350 V8.0
    BogoMIPS : 606.20
    wait instruction : yes
    microsecond timers : yes
    tlb_entries : 32
    extra interrupt vector : no
    hardware watchpoint : no
    ASEs implemented :
    shadow register sets : 1
    kscratch registers : 0
    core : 0
    VCED exceptions : not available
    VCEI exceptions : not available
```

```
    root@home:/# cat /proc/meminfo
    MemTotal: 206932 kB
    etc...
```


## SSH access

1. Login as normal user with Chrome / Firefox (user: 1234 and password: 1234 according to the faq on the Yoigo web)
2. Open the developer console (F12 in Chrome)
3. Click on console. This is the Chrome / Firefox command console in which we can use javascript to program or touch the existing code on any web. Then we will be writing a series of xmo commands to modify the configuration.
4. We look for which is the uid of the user with which we have logged in my case is the uid=3. After executing this command:
```
    > $.xmo.getValuesTree("Device/UserAccounts/Users");

    (4) [{...}, {...}, {...}, {...}]
    0: {uid: 1, Enable: true, Login: "internal", Password: "", SecretQuery: "", ...}
    1: {uid: 2, Enable: true, Login: "acs", Password: "", SecretQuery: "", ...}
    2: {uid: 3, Enable: true, Login: "1234", Password: "", SecretQuery: "", ...}
    3: {uid: 4, Enable: true, Login: "admin", Password: "", SecretQuery: "", ...}
```
Once we have the UID we give permissions to the user to use the SSH console.
```
    > $.xmo.getValuesTree("Device/UserAccounts/Users/User[@uid='3']");
    > $.xmo.getValuesTree("Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses");
    > $.xmo.getValuesTree( "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='3']/Enabled");
    > $.xmo.getValuesTree( "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='4']/Enabled");
```


6. Finally we enable SSH and telnet which are disabled by default.
```
    $.xmo.setValuesTree("ACCESS_ENABLE_ALL", "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='3']/LANRestriction");
    $.xmo.setValuesTree("ACCESS_ENABLE_ALL", "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='4']/LANRestriction");
    $.xmo.setValuesTree(22, "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='3']/Port");
    $.xmo.setValuesTree(23, "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='4']/Port");
    $.xmo.setValuesTree(true, "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='3']/Enabled");
    $.xmo.setValuesTree(true, "Device/UserAccounts/Users/User[@uid='3']/RemoteAccesses/RemoteAccess[@uid='4']/Enabled");
```
7. We restart the router, it is recommended to do it from the router interface. In my tests it does not matter, these changes persist after a physical reboot of the router.
8. We connect by means of SSH/Telnet with the user 1234. I am not going to go into how to connect or what program to use, just say that in the previous step we enabled both the standard telnet port and the standard SSH port.
Once inside the rooter we can gain root access with "su" (the password is usually "root"):
```
    1234@home:/tmp$ su
    Password:

    BusyBox v1.17.3 (2018-10-17 14:24:21 CST) built-in shell (ash).
    Enter 'help' for a list of built-in commands.
    root@home:~#
```


## USB Storage

By default the router USB is enabled as storage but from the normal user interface it can be enabled.
Through Telnet/SSH if you put a USB flash drive it will recognize it immediately and mount it in /mnt/sda1.

This way you can copy files from the router to the pendrive or from the pendrive to the pc to pass some bin, some file that you are working with on the router. The latter is highly recommended for the steps to get the GPON or the SIP password.
```
    root@home:/# cp /source_file.txt /mnt/sda1
    root@home:/# cp /mnt/sda1/origin_file.txt /tmp
```

## GPON Password

The GPON password of the fiber is in hexadecimal. You will have to use some converter if instead of hexadecimal for the new router you want it in plain. For example from: https://codebeautify.org/hex-string-converter
```
    cp /opt/filesystem2/data/optical_conf.txt /mnt/sda1
```


## Initial and current configuration

The xml with the current configuration is in XML is located in: /tmp/cfg.xml I do not know because I have not tested if any direct change in the xml after reboot survives I understand by the folder in which it is not, so it must be stored somewhere else that I have not located yet.

On the other hand the initial configuration of the router that you have just after reset is in: /etc/start-cfg.xml


## SIP configuration

The SIP configuration is located in /tmp/cfg.xml . The block we are interested in is this one:
```
    <SIP>
        <AuthUserName>e34910000000@ims.yoigo.com</AuthUserName>
        <AuthPassword>12345678</AuthPassword>
        <AuthPasswordCrypt/>
        <URI>+34910000000</URI>
    </SIP>
```

In my case with yoigo it seems that the user is *"e34 + my_fix_phone "*. On the other hand the password is random and alphanumeric.


## Password admin

In the MasVoz/Yoigo firmware it seems that it comes with a higher permissions user "admin" that allows you to do many more things on the router. Apparently this user is only active after resetting and entering the GPON key or for the tech support guys to connect to your router for external tele-assistance.

Still by default this user can be accessed but after the first reboot it does not disable what it does is change the password (on first reboot it usually defaults to "admin") and changes it to a random password. In my tests it seems that it is numerical, which comes in handy to obtain it and thus to be able to access the advanced configuration of the router.

1. To obtain it, we have it in the file with the current configuration in: /tmp/cfg.xml
2. The password we have to look for it in the block of the admin user. For example:
```
    <User uid="4">
        <Enable>true</Enable>
        <Login>admin</Login>
        <Password>6b2a8b2864a82a58032a848f87b4a0d5</Password>
```
3. As we can guess the password is not this one but an MD5 of it. As it is numerical it is easily found on the internet by searching in a dictionary to obtain the equivalence of the MD5 with the plain text. I leave this Google search to you.


## DHCP

To get the DHCP window to save in the debug console of your browser you have to type the following commands since the router GUI is bugged:

```
    $('#dhcpForm').scope().dhcpForm.$valid=true
    $('#dhcpForm').scope().dhcpForm.$invalid=false
```


## References

- SAGEMCOM-FAST-5370e-TELIA](https://github.com/wuseman/SAGEMCOM-FAST-5370e-TELIA) Great research on another router not entirely applicable since in Spain the firmware has changed, updated and customized. But it is so similar that even in the firmware there are files that talk about felia and townki.

- Foro Banda Ancha sobre MasMovil](https://bandaancha.eu/foros/extraer-gpon-router-sagemcom-fast-5655v2-1731346) Here I really started to see what the users were saying since my idea was to put my own router at home and the impossibility or impassivity of the MasVoz technical service to help you with the necessary to configure another router or access to a more advanced configuration of the same.
