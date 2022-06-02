# ubuntu-jumpstart-solaris
Guide for installing Solaris (Sparc or Intel) from an Ubuntu Server (20.04 LTS) via Jumpstart

* Credits:
This is taken heavily from Scott Howard's work here:
https://www.docbert.org/Solaris/Jumpstart/linux.html

But, we're Ubuntu and not Redhat. I'm running 20.04 LTS for this, in a VirtualBox VM. I'm thinking 22.04 or newer Ubuntu should be nearly identical for instructions. Eventually, as I want to learn Docker, I'll "dockerize" this like what was done by others for Irix, here: https://github.com/frankeverdij/docker-irixinstall/

For my example the Sun client we're trying to install is called "ultra10" while my Ubuntu VM is named "cockfosters"

1) Add the required packages.
```
sudo apt install rarpd tftp tftpd bootparamd nfs-server nfs-client 
```
2) Copy over the CD or DVD Image. In my case, I'm using Solaris 10 as pilfered from archive.org:
https://archive.org/details/sunsolaris10operatingsystem1106x86sparc

3) Mount the iso as a loopback image to the Linux host:
```
sudo mount -o loop -t iso9660 /media/ISO/SOL_10_1106_SPARC.mdf /media/solaris
```
4) Set up the media to be exported via NFS; add the following line to /etc/exports:
/media/solaris *(ro,no_root_squash)

5) Use the "exportfs" command to export the filesystem and then the "showmount" command to show what's exported:
```
$ sudo exportfs -a
$ sudo showmount -e 
```

6) On your Sun system, use the "banner" command to tell us the MAC address (Ethernet hardware address) of the system. You can also catch this info when turning on the system and typing "Stop + A" to drop the "ok" prompt. Chances are you already know this iif you're reading this document.
```
ok banner
Sun Ultra 5/10 UPA/PCI (UltraSPARC-IIi 440Mhz), Keyboard Present
OpenBoot 3.25, 256 MB (50 ns) memory installed, Serial #16777215
Ethernet address 8:0:20:c0:ff:33, Host ID: 08fffffff.
```
7) Next, we need to tell our Linux system "who" the Sun system is. We do this by
* adding the MAC address to */etc/ethers 
* and the hostname to */etc/hosts
* and the boot parameters to */etc/bootparams

* Add to /etc/ethers:
```
8:0:20:c0:ff:33 ultra10
```
* Add to /etc/hosts
```
192.168.178.9 ultra10
```

* Add to /etc/bootparams
```
ultra10  root=cockfosters:/media/solaris/Solaris_10/Tools/Boot install=cockfosters:/media/solaris boottype=:in
```
8) Now is the more challenging part, we have to prep some boot blocks (a network-based bootloader) for our Sun system. This is done using the tftp software we installed earlier. We have to copy the bootloader (a flat file) to the tftp directory. Then we create a symbolic link with the IP address of the client in HEX to the bootloader so it will be found.
```
sudo mkdir -p /srv/tftp; sudo cp /media/solaris/Solaris_10/Tools/Boot/usr/platform/sun4u/lib/fs/nfs/inetboot /srv/tftp/inetboot.sol10.sun4u
```
Since my IP is 192.168.178.9 we need to convert that to hex. 192 -> C0 168 -> A8 178 -> B2 and 9 is 09. Then we make the symbolic link, and case is important!

```
cd /srv/tftp; sudo ln -s inetboot.sol10.sun4u C0A8B209
```

Some systems require the architecture to be a part of the filename. So the command may need to be "sudo ln -s inetboot.sol10.sun4u C0A8B209.SUN4U" This is mainly for older systems that probably won't support Solaris 10.

In many modern Ubuntu installations "rpcbind" is locked-down. This means that it won't respond to requests sent over the network. To fix this, edit the /etc/default/rpcbind file so it will respond to network requests. Change the "OPTIONS" line to read:
```
OPTIONS="-w -r -i"
```

Then restart the rpcbind daemon itself:
```
sudo service rpcbind restart
```
Now, on the Sun system's console, type
```
ok boot net -- install
```
... and then the network install should complete.

