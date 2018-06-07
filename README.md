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


# How to build:
First build the rust part with cargo\
cd prime_socket
cargo build


then:\
cd src\
make install\
sudo systemctl enable prime-socket\
sudo systemctl start prime-socket


#Notes

The first time you use sudo prime-select nvidia to change, you may get an error about a missing file
/usr/share/X11/xorg.conf.d/20-intel.conf
which the script tries to delete. 
do
sudo touch /usr/share/X11/xorg.conf.d/20-intel.conf
and repeat


Reinstalling may need you to 
sudo /usr/local/bin/prime_socket
and then reattempt
make install

#How does it work?
The script calls the a background service which kills lightdm, takes a few steps to change state, and restarts lightdm
The steps to change state create or delete an xorg config file, and remove or add the nvidia drivers to the running kernel. This work is done in the rust code.

The nvidia drivers are always present in the kernel image when you start the machine (as a consequence of the standard ubuntu install of the nvidia drivers). 
So at startup, they have to be removed before the display manager starts, if you are in intel mode. At this point, the card is turned off. That's the file of the nvidia-prime-boot.service. 

