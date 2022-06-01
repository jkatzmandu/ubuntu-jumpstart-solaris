# ubuntu-jumpstart-solaris
Guide for installing Solaris (Sparc or Intel) from an Ubuntu Server (20.04 LTS) via Jumpstart

* Credits:
This is taken heavily from Scott Howard's work here:
https://www.docbert.org/Solaris/Jumpstart/linux.html

But, we're Ubuntu and not Redhat. I'm running 20.04 LTS for this, in a VirtualBox VM. I'm thinking 22.04 or newer Ubuntu should be nearly identical for instructions. Eventually, as I want to learn Docker, I'll "dockerize" this like what was done for Irix, here: https://github.com/frankeverdij/docker-irixinstall/

1) Add the required packages.
* $ sudo apt install rarpd tftpd bootparamd nfs-server nfs-client mdf2iso

2) Copy over the CD or DVD Image. In my case, I'm using Solaris 10 as pilfered from archive.org:
https://archive.org/details/sunsolaris10operatingsystem1106x86sparc

Because someone used Alcohol 120% to make the image, we need to use the Linux utility mdf2iso and then mount the iso as a loopback image to the Linux host.

3) 
