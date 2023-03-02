# VirtualBox + PXE Boot + Kickstart example

[VirtualBox](https://www.virtualbox.org/wiki/Downloads) support PXE (network booting) out of the box with minimal configuration and no extra software or servers.

This repo contains some basic starter files you can use to bootstrap your VirtualBox PXE boot setup faster.

## Requirements

* VirtualBox is installed and works
* Create a VM you will use to test the PXE boot setup

## VM Settings

In order for PXE booting to work with the builtin server in Virtualbox you need to configure your VM's network settings and boot order properly.

* Set the VM to use the NAT networking (Network -> Adapter 1 -> Attached to: NAT).

* Set the VM to boot from network (System -> Motherboard -> Boot Order). Alternatively, you can use F12 in the booting VM to load the boot menu. For testing PXE and kickstart configs, I found that changing the boot order was easier.

## Setup PXE boot files

Unfortunately VirtualBox does not provide any PXE boot configuration files and only provides a way to serve those files.

To make it easier, I have provided all of the files needed in this repo in the TFTP folder. Copy everything in TFTP folder of this repo to the VirtualBox storage directory. Depending on your system it will be in a different location.

In OSX it is `~/Library/Virtualbox/TFTP`  
In Linux it is `~/.config/VirtualBox/TFTP/`
In Windows 10 Powershell it is `~\.VirtualBox\`

Note: You may want to create this directory if it doesn't exist, but our download below will do the same thing for you

One liner to download and extract the TFTP folder
```shell
curl https://codeload.github.com/defunctzombie/virtualbox-pxe-boot/tar.gz/master | tar zx --strip-components 1
```

```shell
foo bar
````

## Symlink pxelinux.0 to vmname.pxe

From the Virtualbox Docs

> 6.3.2. PXE booting with NAT
> PXE booting is now supported in NAT mode. The NAT DHCP server provides a boot file name of the form vmname.pxe if the directory TFTP exists in the directory where the user's VirtualBox.xml file is kept. It is the responsibility of the user to provide vmname.pxe.

What this means is that in order for VirtualBox to actually PXE boot your machine, it will look for a file called vmname.pxe (where vmname is the name of your virtual machine). Avoid spaces or other special characters in vm names.

On my system, I created a vm called `test` so I would then make a symlink called `test.pxe` to `pxelinux.0`

```shell
ln -s ./pxelinux.0 test.pxe
```

In windows, while one can create symlinks as admin in powershell, it is more simple to copy the file:
```shell
copy .\pxelinux.0 test.pxe
```

## Boot the VM

Click boot and if all went well, you will see a menu with two entries:

* Install
* Kickstart Install

Select either one to try it out!

## Further config

At this point the PXE booting works and you see a menu with two example items I have provided. That is the main extent of this guide however here are some more details about the menu items and configuration.

The menu entries are configured in `TFTP/pxelinux.cfg/default`. This is just a text file. Open it and you will find that the menu entry items look similar to grub boot item lines.

The `Install` menu entry simply boots the Ubuntu wily amd64 installer. The kernel and initrd files were both downloaded from the [ubuntu ftp archive][1] and placed in the `installers/ubuntu/wily/amd64/` directory. In this way you could support many different OS versions and vendors.

The `Kickstart Install` is identical to the `Install` entry except that it provides a [kickstart](https://en.wikipedia.org/wiki/Kickstart_(Linux)) config file to automate the install process. Kickstart stuff is outside of the scope of this guide, but suffice to say you can automate basically any part of the install.

NOTE: The kickstart file is retrieved via HTTP. For the examples here I simply put the kickstart file from `kickstart/basic.cfg` on pastebin so that no extra HTTP server is needed. For testing this is usually sufficient but for production setups you will need to serve your kickstart files via some http server.

## References
* https://gist.github.com/jtyr/816e46c2c5d9345bd6c9

[1]: http://ftp.ubuntu.com/ubuntu/dists/wily/main/installer-amd64/current/images/netboot/ubuntu-installer/amd64/
