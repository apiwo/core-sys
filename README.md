# core-sys
Core-sys is my repo for my linux system packages.
# core-init
To install core-init git clone the project or just download the init file (any flavor).
Then put it into a directory for example mine is 

"/sbin/core-init" (might need to change core init config to match this)

Go to your grub or whatever bootloader and update the config to use the core-init.

For example: init=/sbin/core-init (replace core-init with whatever flavor you downloaded)

Update your grub or other bootloader configuration.

# How to set up
By default the files come with core processes and random filesystem locations.
Replace the file system locations with wherever yours are mounted "nvme0n1p1 for example"
Edit the processes you want to start. 



/usr/lib/elogind/elogind &  **process**
sleep 2  **wait time**
echo -e "  \e[1;32m[ OK ]\e[0m Started elogind" **message and color code**
