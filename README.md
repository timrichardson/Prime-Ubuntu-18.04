# Prime-Ubuntu-18.04
Nvidia Prime without rebooting

# Dependencies:

You need rust.
visit http://www.rust-lang.org
and follow the installation instructions
(or can install from apt)

also, properly install the nvidia drivers the standard ubuntu way, from Additional Drivers


bbswitch (via package bbswitch-dkms)\
Ubuntu 18.04 (might work with other distros if you change some paths)

lightdm as the display manager
sudo apt install lightdm

The ubuntu install of the nvidia driver will also install nvidia-prime, Ubuntu's optimus module. The code supersedes that but you should leave the ubuntu package installed. 


# How to build & install
First build the rust part with cargo\
cd prime_socket\
cargo build


then:\
cd src\
make install\
sudo systemctl enable prime-socket\
sudo systemctl start prime-socket


# Notes

The first time you use sudo prime-select nvidia to change, you may get an error about a missing file
/usr/share/X11/xorg.conf.d/20-intel.conf
which the script tries to delete. 
Do: `sudo touch /usr/share/X11/xorg.conf.d/20-intel.conf`
and repeat `sudo prime-select nvidia`


Reinstalling may need you to 
```
sudo rm /usr/local/bin/prime_socket
```
and then reattempt
`make install`

## Prime sync for tear free laptop panel
In nvidia mode, you'll get tearing on the laptop unless you enable prime sync.\
`sudo vi /etc/modprobe.d/zz-nvidia-modeset.conf`
and include this:
```
#enable prime-sync
options nvidia-drm modeset=1
```
and \
`sudo update-initramfs -u`



## Display manager doesn't start?
If you swap to intel, reboot and can't get the display manager working, this is probably because the nvidia drivers were not unloaded. 
get to a virtual console and 
check with `lsmod|grep nvidia`. 
If this is the problem, then

```
sudo rmmod nvidia_drm
sudo rmmod nvidia_modeset
sudo rmmod nvidia_uvm
sudo rmmod nvidia
sudo systemctl start lightdm
```
but you will have to work out why the nvidia-prime-boot.service did not do its job.


# Usage

```
sudo prime-select intel|nvidia|query
```

# How does it work?

It uses a modified version of prime-select.\
The modified version is installed into /usr/local/bin which comes first in the standard path, so it masks the version of the nvidia-prime package

This version uses bbswitch to disable the nvidia card, which was the standard Ubuntu method until 18.04

There are virtually no reports of bbswitch not working in ubuntu 18.04 and there are many reports of the new way not working. 

The script calls a background service which kills lightdm, takes a few steps to change state, and restarts lightdm. Killing the display manager is necessary to remove the nvidia drivers.

The steps to change state:

* create or delete an xorg config file, 
* and remove or add the nvidia drivers to the running kernel. 

This work is done in the rust code.


The nvidia drivers are always present in the kernel image when you start the machine (as a consequence of the standard ubuntu install of the nvidia drivers). 
So at startup, they have to be removed before the display manager starts, if you are in intel mode. At this point, the card is turned off. Removing the drivers and turning off the nvidia card is the job of the nvidia-prime-boot.service. 

