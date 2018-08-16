# Fast Switch  Prime-Ubuntu-18.04

Nvidia Prime for Optimus laptops using Ubuntu & lightdm. Change hybrid & pure Intel modes without rebooting. Based on Ubuntu 18.04's prime-select & bbswitch.

It is not invasive. It requires that you change your display manager, it installs a script and it installs a small background service that does nothing until you change modes. It doesn't touch the kernel, boot arguments, grub or your nvidia drivers.

Requires lightdm.

It looks like this: https://www.youtube.com/watch?v=RfB_IWw7pl4&feature=youtu.be

Why does this exist? It restores the pre-18.04 'bbswitch' approach; it's much faster to change profiles and more reliable. However, it also takes advantage of recent improvements, so you can swap modes without rebooting.

This is not for Ubuntu beginners. If things go wrong, you need to know about virtual consoles and recovery mode and some basic systemd admin.

To install it, you need to know about git clone and you need to be able to change your display manager to lightdm.

Good support comes from this thread:
https://devtalk.nvidia.com/default/topic/1032482/linux/optimus-on-ubuntu-18-04-is-a-step-backwards-but-i-found-the-first-good-solution/

Note: the Ubuntu developer who works so hard on this, delivering Ubuntu and Mint the best Optimus experience of any Linux distribution, is working on a new approach to switching, which is close to the old pre 18.04 method. 

He has a harder task than unofficial solutions like this code, because he needs to find a solution that works with gdm3, but for sure standard Ubuntu will deliver a better experience at some point. 

# Dependencies:

You need rust.

* install from apt: `sudo apt install rustc cargo`

* properly install the nvidia drivers the standard ubuntu way, from Additional Drivers.

If you have done this already, make sure you do 
```sudo /usr/bin/prime-select nvidia ``` 
to ensure that nvidia drivers are installed in your initramfs. 

* Ubuntu 18.04 and siblings, most of which use lightdm anyway (might work with other distros of similar age which are based on the vendor-neutral library approach, if you change some paths)

* bbswitch (via `sudo apt install bbswitch-dkms`)

* lightdm as the display manager
```
sudo apt install lightdm
```
You can swap between installed display managers with `sudo dpkg-reconfigure lightdm`
I don't know why gdm3 doesn't work with this. I literally don't know. Hence lightdm

The ubuntu install of the nvidia driver will also install nvidia-prime, Ubuntu's optimus module. This code supersedes that (but does not overwrite it, it puts modified scripts elsewhere in your path). You should leave the ubuntu package installed though.

Note: while testing this in a reinstall of Ubuntu 18.04, lightdm did not install properly on one laptop. Work-around: install xubuntu-desktop which relies on lightdm, but still kept ubuntu as the log-in session.


# How to build & install

Pay attention to the tip above: make sure you are in a working nvidia mode.
Naturally, make sure you have git and git clone this repository. After you have done that...

```
cd prime_socket/src
sudo make install
```

# Usage

```
sudo prime-select intel|nvidia|query
```

The first time you use sudo prime-select nvidia to change, you may get an error about a missing file
/usr/share/X11/xorg.conf.d/20-intel.conf
which the script tries to delete. 
Do: `sudo touch /usr/share/X11/xorg.conf.d/20-intel.conf`
and repeat `sudo prime-select nvidia`

* todo: fix this, it's a paper-cut.


Note: the modified script is installed into /usr/local/bin. By default, scripts in this path will be found before the official script in /usr/bin

# Cautions (how to break this)

Don't use the graphical switcher of the nvidia-control panel. It uses the standard debian way, which will rebuild your kernel image: it does this to remove the nvidia drivers with extreme prejudice when you swap to intel mode, which will stop this fast-switch method from working, because it assumes the nvidia drivers are present.

If you remove the nvidia modules using Ubuntu's standard (slow) method, you will need to use the standard method to put them back (by using the nvidia control panel to swap back to nvidia or from a shell).
If you want to use the standard prime-select script, it is untouched at
/usr/bin/prime-select

The modified version at /usr/local/bin has priority in the path. To use the standard script, be explicit about the path.

## Optional: get the nvidia control panel to use the modified prime-select code

Ubuntu has added a section to the nvidia control panel which lets you change profiles; it's referred to above. These modifications run /usr/bin/prime-select.

So you can make the gui tool use the modified version of prime-select once you're happy that it's working.

```
sudo mv /usr/bin/prime-select /usr/bin/prime-select_orig
sudo ln -s /usr/local/bin/prime-select /usr/bin/prime-select
```



# Did it work?

after entering intel mode, start a sudo shell and test if the card is off:
```
sudo -i
modprobe bbswitch
cat /proc/acpi/bbswitch
```


# Notes

You must have the nvidia drivers installed in your initramfs.
This will be true if you have installed the standard Ubuntu nvidia-drivers but it will not be true if you did the standard ```prime-select intel```.
See notes above. 



# Uninstall

This code doesn't really disturb your system much. 
You could rename /usr/local/bin/prime-select to /usr/local/bin/prime-select-fast so that the standard script is no longer masked by the modified one.

purge and reinstall the package nvidia-prime
`sudo apt purge nvidia-prime; sudo apt install nvidia-prime`
And then

`sudo /usr/bin/prime-select nvidia`

and to revert to gdm3, install and select it as the default:
```
sudo apt install gdm3
sudo dpkg-reconfigure gdm3
```


You should be back to standard ubuntu now. 

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

This is not for Ubuntu beginners. You need to know about virtual consoles and recovery mode and some basic systemd admin.

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

## Intel-mode fix attempt 1:

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

## Intel-mode Fix attempt 2
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
