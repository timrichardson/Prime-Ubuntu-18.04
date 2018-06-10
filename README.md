# Prime-Ubuntu-18.04
Nvidia Prime without rebooting. Requires lightdm (although gdm3 seems to work much better in Ubuntu 18.04 and possibly this code could work with it).

# Dependencies:

You need rust.
visit http://www.rust-lang.org
and follow the installation instructions. It's one line. 
The rust website urges you to install it this way and it's what I did. 
However, I did test the build once using the Ubuntu rust pacages, and it worked fine. 
(install from apt: `sudo apt install rustc`)

also, properly install the nvidia drivers the standard ubuntu way, from Additional Drivers

* This code assumes the nvidia drivers are installed. They are installed by the standard Ubuntu module, but if you did the standard 18.04 `prime-select intel`, they are no longer installed in the kernel. So do `prime-select nvidia` and reboot. Make sure you have the nvidia drivers running before proceeding.


* Ubuntu 18.04 (might work with other distros of similar age which are based on the vendor-neutral library approach, if you change some paths)

* bbswitch (via package bbswitch-dkms)\

* lightdm as the display manager
```
sudo apt install lightdm
```

The ubuntu install of the nvidia driver will also install nvidia-prime, Ubuntu's optimus module. The code supersedes that but you should leave the ubuntu package installed. 


# How to build & install

First build the rust part with cargo
```
cd prime_socket
cargo build
```


then:
```
cd src
sudo make install
sudo systemctl enable prime-socket
sudo systemctl start prime-socket
```


# Usage

```
sudo prime-select intel|nvidia|query
```

Don't use the graphical switcher of the nvidia-control panel. 

It uses the standard debian way, which will rebuild your kernel. It goes to the effort of actually removing the nvidia drivers if you go to intel mode, which will stop this method from working, since it uses the much faster of approach of not uninstalling anything, but unloading modules from memory if we don't want them. 
If you remove the nvidia modules using Ubuntu's standard (slow) method, you will need to use the standard method to put them back (by using the nvidia control panel to swap back to nvidia).
If you want to use the standard prime-select script, it is untouched at
/usr/bin/prime-select

The modified version at /usr/local/bin has priority in the path so if you need to use the standard script, be explicit about the path.


# Notes

The first time you use sudo prime-select nvidia to change, you may get an error about a missing file
/usr/share/X11/xorg.conf.d/20-intel.conf
which the script tries to delete. 
Do: `sudo touch /usr/share/X11/xorg.conf.d/20-intel.conf`
and repeat `sudo prime-select nvidia`

* todo: fix this, it's a paper-cut.


Reinstalling may need you to 
```
sudo rm /usr/local/bin/prime_socket
```
and then reattempt
`make install`

# Uninstall

This code doesn't really disturb your system much. 
You could rename /usr/local/bin/prime-select to /usr/local/bin/prime-select-fast


If you are in intel mode, then nvidia-prime-boot.service is enabled, and it will unload the nvidia drivers. The standard Ubuntu method does not expect this; if shouldn't affect you booting in intel mode, but it can't be good if you are trying to use the standard Ubuntu method to boot into hybrid mode. 
So disable the service.

`sudo systemctl disable nvidia-prime-boot.service`

## Uninstall bbswitch-dkms
You installed the bbswitch-dkms module to get this working.
The standard Ubuntu approach doesn't use bbswitch (the decision which causes all the problems). I wouldn't expect any problems by leaving it installed, but it is unnecessary if you want to use the standard Ubuntu 18.04 approach to Optimus.


# Prime sync for tear free laptop panel

This tip applies to standard Ubuntu too. 

In nvidia mode, you'll get tearing on the laptop unless you enable prime sync.\
`sudo vi /etc/modprobe.d/zz-nvidia-modeset.conf`
and include this:
```
#enable prime-sync
options nvidia-drm modeset=1
```
and \
`sudo update-initramfs -u`

Tearing you see on non-laptop panels won't be fixed by prime sync. For that problem, you need to turn on pipeline-composition on the affected screens (via the nvidia control panel). Learn more on the nvidia developer linux forums.



# Troubleshooting: Display manager doesn't start?

First, make sure you did the systemctl lines of the the install instructions.


If you swap to intel, reboot and can't get the display manager working, this is probably because the nvidia drivers were not unloaded. 

## Fix attempt 1:

boot in recovery mode, and choose "resume boot" (possibly twice)
This will probably get lightdm started, allowing you to log in.

Check if the service which unloads the nvidia drivers is working:
```
sudo -i
systemctl status nvidia-prime-boot.service
```

Here is an example of healthy output:
```
root@raffles:~# systemctl status nvidia-prime-boot.service
● nvidia-prime-boot.service - dGPU off during boot
   Loaded: loaded (/etc/systemd/system/nvidia-prime-boot.service; disabled; vendor preset: enabled)
   Active: inactive (dead)

Jun 10 10:24:09 raffles systemd[1]: Starting dGPU off during boot...
Jun 10 10:24:09 raffles systemd[1]: Started dGPU off during boot.
```

you may also find something useful in 
```
journalctl -e
```

You should not see an error the bbswitch is not installed, because that means you didn't read the instructions above. Also, you should not see errors that no nvidia modules are installed, because that means you either did not install the nvidia drivers, or you removed them (perhaps by 18.04-standard `prime-select intel`, in which case `sudo /usr/bin/prime-select nvidia` and reboot. Pleaes carefully read the installation instructions above ...

## Fix attempt 2
if you can't get to a graphical session even with recovery boot,
 then try to get to a virtual console and 
check with `lsmod|grep nvidia`. 
If the nvidia drivers are present:
then from the virtual terminal:

```
sudo systemctl stop lightdm
sudo rmmod nvidia_drm
sudo rmmod nvidia_modeset
sudo rmmod nvidia_uvm
sudo rmmod nvidia
sudo systemctl start lightdm
```
but you will have to work out why the nvidia-prime-boot.service did not do its job.

These two methods have solved any problems I have encountered. 

## Still stuck?

An idea: 
turn off nvidia-prime-boot.service
`systemctl disable nvidia-prime-boot.service`

swap to a virtual terminal (eg ctrl-alt F4)

run `sudo /usr/local/bin/prime_socket`

now go back to your GUI session, or some other virtual terminal, and do 
`prime-select intel`
and see what you see in the prime_socket VT

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

# How is this different to the standard 18.04 approach?

Ubuntu 18.04 does not use bbswitch to power-off the nvidia card when you are in intel-only mode. Instead, the developers swapped to an officially-supported kernel feature, which only works when the nouveau driver is present. 
Unfortunately, this means the nvidia drivers have to be removed. So prime-select intel goes through an elaborate process of removing the nvidia drivers, rebuilding the initramfs image and rebooting, solely to load nouveau so the nvidia card can be turned off. 

Swapping back to nvidia then requires the basically the same process to repeat, except this time the nvidia modules are re-added to the kernel image. 

It is a very time consuming approach, mandating a reboot. Also, quite a few users have trouble getting the nouveau-power-off to work. 

bbswitch is not officially in the kernel. However, it is well used in just about all other distributions and there is no sign that it will stop working.
