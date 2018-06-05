# Prime-Ubuntu-18.04
Nvidia Prime without rebooting

# Dependencies:
bbswitch\
Nvidia 390\
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
