# Fast Switch  Prime-Ubuntu-18.04 / Mint 19

*** Update October 31, 2018
The work done to by Alberto Milone to backport the 18.10 improvements to 18.04 is working well for me, so this repository may be redundant in 18.04 and it should be un-necessary in 18.10. Functionally the difference is that the code here will immediately kill your session, saving you the trouble of logging out. Alberto's packages require you to log out, but no rebooting, so kernel rebuidling, it's smooth. However, the 18.10 packages are still more recent than the 18.04 packages, so working in 18.10 on your hardware may not mean working in 18.04. IN which case, try the solution here.

If you run the nvidia control panel in intel mode, you get an error and it doesn't start, so you switch back to Nvidia with sudo prime-select nvidia

This Readme has steps to uninstall.

lightdm is required. If gdm3 works with nvidia modesetting and external displays, let me know, but it hasn't worked since at least 16.04. 

So ubuntu users with Optimus should stick with lightdm, even in ubuntu 18.10

Alberto plans to backport the changes to 18.04, at which point the workaround here will be redundant

***



Nvidia Prime for Optimus laptops using Ubuntu & lightdm, based on work done by Matthieu Gras.

Lets you change hybrid & pure Intel modes without rebooting. Based on Ubuntu 18.04's prime-select & bbswitch. I have tested it on Mint 19, it works there too, but I use Ubuntu 18.04 on my Optimus laptops.

Ubuntu 18.10 has a greatly improved version of prime-select which in my testing is very good, no need to install this repo. The 18.10 version of prime-select is reboot-less, finally fixes gdm3 when using nvidia in modeset (necessary for tear free graphics).

All the references below to Ubuntu's standard prime select are specifically to the 18.04 version.

I am using the long term support nvidia driver, I am currently on 390.87.

Ubuntu's prime-select method (fixed here) is different to bumblebee. If you don't care about external monitors, you may find bumblebee better (I have never used bumblebee). The bumblebee project is responsbible for the excellent bbswitch tool (powers off the nvidia card), which Ubuntu 18.04 removed and which this Matthieu Gras approach restores.

This Matthieu Gras method is not very invasive. It requires that you change your display manager, it installs a script and it installs a small background service that does nothing until you change modes. It doesn't touch the kernel, boot arguments, grub or your nvidia drivers, although it does require you to put back the bbswitch-dkms package to ubuntu, which is a kernel module.

**Requires lightdm. And forget about wayland** 

It looks like this: [YouTube: Optimus fast switch Ubuntu 18.04](https://youtu.be/RfB_IWw7pl4)

## Why does this code exist?

Why does this exist? It restores the pre-18.04 'bbswitch' approach; it's much faster to change profiles and more reliable. However, it also takes advantage of recent improvements, so you can swap modes without rebooting.

## Who should try this?

This is not for Ubuntu beginners. If things go wrong, you need to know about virtual consoles and recovery mode and some basic systemd admin. If you want support, you'll need to read the debugging steps in this document.

To install it, you need to know about `git clone` and you need to change your display manager to lightdm.

Good nvidia technical support comes from this thread:
[Optimus on Ubuntu 18.04 is a step backwards ... but I found the first good solution](https://devtalk.nvidia.com/default/topic/1032482/linux/optimus-on-ubuntu-18-04-is-a-step-backwards-but-i-found-the-first-good-solution)

Note: the Ubuntu developer who works so hard on this, delivering Ubuntu and Mint the best Optimus experience of any Linux distribution, is working on a new approach to switching, which is close to the old pre 18.04 method. He has a harder task than unofficial solutions like this code, because he needs to find a solution that works with gdm3. 

## Tear-free prime sync
Please see the tip below about activating prime-sync for nvidia optimus tear free graphics on the laptop's panel when in nvidia mode. With this script and that fix, you can look forward to a decent Optimus experience.
Note that the changes to prime-select in pre-release Ubuntu 18.10 currently default to mode-setting being activated (the first linux distribution to do this, as far as I know). 

# Dependencies and preparation:

* This is tested mostly with Ubuntu 18.04, but should work in Mint 19 and xubuntu.

* You need the programming language rust, install from apt: `sudo apt install rustc cargo`

* properly install the nvidia drivers the standard ubuntu way, from Additional Drivers.

* If you have done this already, make sure you do 
```sudo /usr/bin/prime-select nvidia ``` 
to ensure that nvidia drivers are installed in your initramfs. That is, use the standard Ubuntu script to force nvidia mode. Do this after upgrading nvidia drivers too, if you upgraded while in intel mode.

* install bbswitch (via `sudo apt install bbswitch-dkms`)

* lightdm as the display manager
```
sudo apt install lightdm
```

## Changing display-managers
You can swap between installed display managers with `sudo dpkg-reconfigure lightdm`
I don't know why gdm3 doesn't work with nvidia drivers in modeset mode. No one seems to know how to fix it. Hence lightdm

The ubuntu install of the nvidia driver will also install nvidia-prime, Ubuntu's optimus module. This code supersedes that (but does not overwrite it, it puts modified scripts elsewhere in your path). You should leave the ubuntu package installed though.

Note: while testing this in a reinstall of Ubuntu 18.04, lightdm did not install properly on one laptop. Work-around: install xubuntu-desktop which relies on lightdm, but still kept ubuntu as the log-in session.


# How to build & install

Pay fanatically detailed attention to the Dependencies and Preparation steps above. Each step is important. Make sure you are booted in a working nvidia mode!

Naturally, make sure you have git and git clone this repository. After you have done that...

```
cd prime_socket/src
sudo make install
```
The first time you do that, 90% of the total time will be downloading rust dependencies.

# Upgrading to a new nvidia driver: do this when you are already in nvidia mode
Tip: `sudo prime-select nvidia` before updating nvidia.

If you upgrade in intel mode, probably the nvidia driver won't be added to your kernel image, since 18.04 removes the nvidia driver from the kernel when in intel mode. This is guaranteed not to work: this script assumes the nvidia drivers are always in the kernal, as it was before 18.04. 
Therefore, if that happens, you won't be able to change to nvidia mode until you fix it. 
Force it into nvidia mode (`/usr/bin/prime-select nvidia`, that is, use the official Ubuntu 18.04 script).


This script should work again but you can try 
```sudo update-initramfs -u```
for good luck.


# Usage (changing modes)
Usage is the same as the standard script, but this one takes about five seconds. Note: it will immediately kill your gnome session and log you out, by terminating the display manager with root privileges.

```
sudo prime-select intel|nvidia|query
```


Note: the modified script is installed into /usr/local/bin. By default, scripts in this path will be found before the official script in /usr/bin. This means that after you do the install, you have the original, standard ubuntu script untouched, but the modified one is earlier in the path.

# Cautions (how to break it)

Don't use the graphical switcher of the nvidia-control panel to change modes. It uses the standard debian prime-select script, which will remove the nvidia driver and rebuild your kernel image when you go to intel mode, which will stop this fast-switch method from working, because this Matthieu Gras approach assumes the nvidia drivers are always present in the kernel image (it removes them when you are in intel mode, prior to powering off the nvidia card with bbswitch).

If you remove the nvidia modules using Ubuntu's standard (slow) method, you will need to use the standard method to put them back (by using the nvidia control panel to swap back to nvidia or from a shell).
If you want to use the standard prime-select script, it is untouched at
/usr/bin/prime-select

The modified version at /usr/local/bin has priority in the path. To use the standard script, be explicit about the path.

## Optional/Experimental: get the nvidia control panel to use the modified prime-select code

Ubuntu has added a section to the nvidia control panel which lets you change profiles; it's referred to above. These modifications run /usr/bin/prime-select.

So you can make the gui tool use the modified version of prime-select once you're happy that it's working.

```
sudo mv /usr/bin/prime-select /usr/bin/prime-select_orig
sudo ln -s /usr/local/bin/prime-select /usr/bin/prime-select
```
Of course, this means if you need to use the standard script (for example, to force nvidia mode if you upgrade nvidia drivers without reading the notes above), you have to explicitly use the official script).


# Did it work? Is the nvidia card powered off?

after entering intel mode, start a sudo shell and test if the card is off:
```
sudo -i
modprobe bbswitch
cat /proc/acpi/bbswitch
```
and good output (for intel mode) looks like this
```
tim@w520-mint ~ $  cat /proc/acpi/bbswitch 
0000:01:00.0 OFF

```
## What services should be running in intel mode?
In intel mode, the service nvidia-prime-boot.service should execute before the display manager starts, to remove nvidia and nouveau modules, and then it uses bbswitch to power-off the card. The modified prime-select script activates this service when doing prime-select intel
and it is obviously deactivated when prime-select nvidia


This is typical output in intel mode:
```
$ systemctl status nvidia-prime-boot.service 
● nvidia-prime-boot.service - Unload nvidia modules and turn dGPU off during boot
   Loaded: loaded (/etc/systemd/system/nvidia-prime-boot.service; enabled; vendor preset: enabled)
   Active: inactive (dead)

```
## What services should be running in nvidia mode?
The nvidia-prime-boot.service should be disabled.

```
$ systemctl status nvidia-prime-boot.service 
● nvidia-prime-boot.service - Unload nvidia modules and turn dGPU off during boot
   Loaded: loaded (/etc/systemd/system/nvidia-prime-boot.service; disabled; vendor preset: enabled)
   Active: inactive (dead)

```

# Notes

You must have the nvidia drivers installed in your initramfs.
This will be true if you have installed the standard Ubuntu nvidia-drivers but it will not be true if you did the standard 

```
prime-select intel
```

See notes above. 



# Uninstall

This code doesn't really disturb your system much. 
You could rename /usr/local/bin/prime-select to /usr/local/bin/prime-select-fast so that the standard script is no longer masked by the modified one.


purge and reinstall the package nvidia-prime
```
sudo apt purge nvidia-prime; sudo apt install nvidia-prime
```

Disable services:

```
systemctl disable nvidia-prime-boot.service
systemctl disable prime-socket.service
```

And then

```
sudo /usr/bin/prime-select nvidia
```

and to revert to gdm3, install and select it as the default:

```
sudo apt install gdm3
sudo dpkg-reconfigure gdm3
```


You should be back to standard ubuntu now. 

## Uninstall bbswitch-dkms
You installed the bbswitch-dkms module to get this working.
The standard Ubuntu approach doesn't use bbswitch (the decision which causes all the problems). I wouldn't expect any problems by leaving it installed, but it is unnecessary if you want to use the standard Ubuntu 18.04 approach to Optimus.


# Nvidia Optimus Prime sync for tear-free laptop panel

This tip applies to standard Ubuntu/Mint too.

In nvidia mode, you'll get tearing on the laptop unless you enable prime sync.
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

This is not for Ubuntu beginners. You need to know about virtual consoles and recovery mode and some basic systemd admin.he 
The easiest way to screw this up is not to set ligthdm as your default display-manager. You probably won't be able to login.
So...

First, you need access to a virtual console. 
Depending on what has gone wrong, you may be able to access a virtual console.
`sudo apt install openssh-server` is always helpful too.

Starting in recovery mode usually works to get a GUI login. (choose resume boot twice during the boot process). 
make sure the systemctl lines in the makefile worked by using systemctl status prime-socket.
This is what it should look like: the service should be active and running.
```
● prime-socket.service - Socket service for on the fly prime switching
   Loaded: loaded (/etc/systemd/system/prime-socket.service; enabled; vendor pre
   Active: active (running) since Fri 2018-06-15 08:41:00 AEST; 31min ago
 Main PID: 808 (prime_socket)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/prime-socket.service
           └─808 /usr/local/bin/prime_socket

Jun 15 08:41:00 raffles systemd[1]: Started Socket service for on the fly prime
```


## Display manager doesn't start in intel mode

If you swap to intel, reboot and can't get the display manager working, this is probably because the nvidia drivers were not unloaded. 

## Intel-mode fix attempt 1

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

You should not see an error telling you that bbswitch is not installed, because that means you didn't read the instructions above. Also, you should not see errors that no nvidia modules are installed, because that means you either did not install the nvidia drivers, or you removed them (perhaps by 18.04-standard `prime-select intel`, in which case `sudo /usr/bin/prime-select nvidia` and reboot. Please carefully read the installation instructions above ...

## Intel-mode fix attempt 2
if you can't get to a graphical session even with recovery boot,
 then try to get to a virtual console and 
check with `lsmod|grep nvidia`. 
and to be safe, check `lsmod|grep nouveau`
You should never see the nouveau driver, this would be a nasty bug, please open an issue.

If the nvidia drivers are present:
then from the virtual terminal:

```
sudo systemctl stop lightdm
sudo rmmod nouveau #in case it is loaded
sudo rmmod nvidia_drm
sudo rmmod nvidia_modeset
sudo rmmod nvidia_uvm
sudo rmmod nvidia
sudo systemctl start lightdm
```
but you will have to work out why the nvidia-prime-boot.service did not do its job.

## Intel-mode fix attempt 3
As a last resort you can try to rename/remove your Xorg configuration file. Xorg file misconfiguration can prevent the desktop manager from starting. Such scenario could happen after major dist-upgrades from 16.04 to 17.10/18.04 or if you have custom Xorg settings, which by default in 18.04 should go to ```/usr/share/X11/xorg.conf.d```. So do this only if all other options/workarounds doesn't work.

```
sudo mv /etc/X11/xorg.conf /etc/X11/xorg.conf.backup
reboot
```

This should help if your X server can't find screens or devices.
You can check the X server logs with:
```
sudo cat /var/log/Xorg.0.log

# OR the rootless way
cat ~/.local/share/xorg/Xorg.0.log
```

If this doesn't help, you can restore your original Xorg config with:
```
sudo mv /etc/X11/xorg.conf.backup /etc/X11/xorg.conf
```


## Display manager doesn't start in nvidia mode?

You probably don't have the nvidia drivers installed in your kernel image, which can happen even if think you have the nvidia modules installed, because the standard 18.04 optimus logic uninstalls the drivers when you choose intel mode. We don't want that. 

Try ```sudo /usr/bin/prime-select nvidia```. If it complains that you are already in nvidia mode, do ```sudo /usr/bin/prime-select intel``` and then ```sudo /usr/bin/prime-select nvidia```

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

It uses a modified version of prime-select.

The modified version is installed into /usr/local/bin which comes first in the standard path, so it masks the version of the nvidia-prime package

This version uses bbswitch to disable the nvidia card, which was the standard Ubuntu method until 18.04

There are virtually no reports of bbswitch not working in ubuntu 18.04 and there are many reports of the new way not working. 

The script calls a background service which kills lightdm, takes a few steps to change state, and restarts lightdm. Killing the display manager is necessary to remove the nvidia drivers.

The steps to change state:

* create or delete an xorg config file, 
* and remove or add the nvidia drivers to the running kernel. It never adds nvidia drivers which are missing; it assumes they are always in a booting-kernel, and unloads them & tunrs off the card if you are in intel mode. Therefore, it doesn't need to do much at all if you want nvidia mode; nvidia mode is basically the default situation. 

The nvidia drivers are always present in the kernel image when you start the machine as a consequence of the standard ubuntu install of the nvidia drivers as long as you have not removed them by standard prime-select intel

The rust code prepares the state change. 
 nvidia-prime-boot.service is what removes the nvidia drivers and powers off the card; it obviously only runs if you selected intel mode.

# How is this different to the standard 18.04 approach?

Two things are different. Firstly, nvidia-prime in Ubuntu 18.04 does not use bbswitch to power-off the nvidia card when you are in intel-only mode. Instead, the developers swapped to an officially-supported kernel feature, which apparently only works when the nouveau driver is present. 
Unfortunately, this means the nvidia drivers have to be removed. So prime-select intel goes through an elaborate process of removing the nvidia drivers, rebuilding the initramfs image and rebooting, solely to load nouveau so the nvidia card can be turned off. 

Swapping back to nvidia then requires the basically the same process to repeat, except this time the nvidia modules are re-added to the kernel image. 

It is a very time consuming approach, mandating a reboot. Also, quite a few users have trouble getting the nouveau-power-off to work, so for those users it is slow, intrusive and broken.

bbswitch is not officially in the kernel. However, it is well used in just about all other distribution; it has a significant user-base, and there is no sign that it will stop working. Right now, it works with kernel 4.17 even as Ubuntu 18.04 is based on 4.15.
