# How to setup a personal desktop with Ubuntu 18.04 and connect to it remotely

## Ubuntu installation
Assuming that use have a bootable usb with Ubuntu 18.04 and a clean machine.

0. Insert the usb and reboot the machine into usb.
Note: When you click F11 and the list of bootable options will be prompted, try to boot into UEFI (EFI) rather than USB (BIOS).
1. Click Try Ubuntu
2. Connect to internet
TUNI note: connect to 'roam.fi'. it will ask your credentials, enter those and click 'No certificate is required' and click connect.
3. Click Install Ubuntu on desktop and proceed with installation.
4. uncheck box with 3rd party drivers and download updates. 
5. when you will be ask about something like Automatic installation or 'Something else', select something else. 
6. Allocate the space
The window will ask you to partition your disks. First, select the disk from which you want your Ubuntu to boot (bottom of the window), then free up the space on all drives if you have something on your drives (use minus icon), and proceed with allocating space for `/`, `/home`, `/boot`, etc (if from TUNI, please see the next TUNI note).
TUNI note: the desktops that we had so far had: 512GB SSD, 1TB NVMe, 8TB HDD. We recommend to use SSD for OS, NVMe for the data/model-logs for your current project, and HDD for archiving the data that you don't want to download or process once more. Therefore, select SSD for booting. On SSD allocate: 1024MB to EFI Partition (just enter the 1024 and select EFI partition from the menu, rest is by default. If you don't have EFI, only BIOS, you boot live usb into bios mode instead of efi, see Note to the first step), RAM+10GB allocate to swap area, 256GB allocate for the `/` and the rest for `/home`. On NVMe allocate: the entire space to `/home/nvme/`. On HDD allocate: 8TB to `/home/hdd`.

7. Click Continue or whatever (check once again that you selected the correct boot disk)
8. Name your machine and select username. Select password
TUNI note: ask the group first how to name your desktop because we all may use this desktop in the future. Username should be `esa`.
9. Click install. Wait. Reboot. 
10. `sudo apt-get update && sudo apt-get upgrade`
11. Reboot
12. Download the latest drivers from nvidia web site (runfile)
13. `chmod +x <driver-file>`
_following some steps from https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07_
14. Install dependencies `sudo apt-get install build-essential gcc-multilib dkms`
15. Creat Blacklist for Nouveau Driver `sudo nano /etc/modprobe.d/blacklist-nouveau.conf` and add these lines there
```
blacklist nouveau
options nouveau modeset=0
```
16. Run `sudo update-initramfs -u`
17. Reboot. login into text mode instead of graphical interface (ALT+CTRL+F2)
18. Install drivers `sudo ./NVIDIA-Linux-x86_64-384.69.run --dkms -s` (note the `dkms` and `-s` at the end)
19. Check the installation `nvidia-smi`
20. Reboot. You shuld be able to enter the ubuntu graphical interface.
21. Prohibit updating the drivers in Software updates uncheck 'proprietary drivers for devices'
TUNI note: Currently you cannot write and edit files in `/home/hdd` and `/home/nvme` if your not a root. To fix this, open Nautilus with sudo rights `sudo nautilus`. Go to hdd. Click right mouse and go to Properties > Permissions. Change both Owner and Group to the current username (`esa`). Also, click Change Permission for Encosed files at the bottom on the window and give Read and write and Create and delete files permissions to Owner and Group. Repeat for nmve.
22. Adjust sleeping behaviour (from https://askubuntu.com/questions/47311/how-do-i-disable-my-system-from-going-to-sleep) `sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target`
23. install neccessary packages `sudo apt-get install htop tmux cmake git elinks net-tools`. (optionally) install nvtop (htop for gpus, follow instructions here https://github.com/Syllo/nvtop dont dorget sudo before `make install`). 
24. install openssh-server: `sudo apt-get install openssh-server` (if throws errors with packages see https://askubuntu.com/questions/760378/can-not-install-openssh-server-on-ubuntu-16-04/795062)
25. install miniconda/anaconda (just search in google how to install one of those but basically you need to `wget` the `.sh` file and run it with bash). During installation select `yes` when it will prompt whether to add into to `.bashrc`. Don't dorget to `source .bashrc` afterwards.
26. Setup VPN connection. See info below.
27. Turn on automatic boot into text-mode (no graphical interface). `sudo systemctl set-default multi-user.target` (see also https://askubuntu.com/questions/1056363/how-to-disable-gui-on-boot-in-18-04-bionic-beaver).


## Setup VPN
In order to connect to the desktop remotely you need to connect the desktop to the VPN. Your laptop (or any computer) should be also connected to the same VPN (or to public wifi in the company). Don't forget to install `openssh-server` before you will try to connect through `ssh`. On 16.04 you may need to install pptp things: `sudo apt-get install network-manager-pptp pptp-linux`.

The setup is inspired by https://www.youtube.com/watch?v=nKP6M0xIuqE. The specific info can be found from IT helpdesk in your company (TUNI note: https://info2019.tuni.fi/wireless-networks/#Roam.fi).

1. go to vpn settings. and follow the steps from the video mentioned above and use the info from IT helpdesk in your company (TUNI note: `staff-ras.vpn.tut.fi` gateway and the credentials from TUT). In the password form select that it will be applicable for all users.
2. Then we need to prohibit Wired connections to connect automatically after the startup. Type `nm-connection-editor` in terminal (or just to to Edit connection on 16.04). Click on wired connections and go to Preferences (setting icon at the bottom) > General > uncheck the box. Repeat for another connection.
3. Also, in the same editor, make sure to allow your desktop to connect to internet automatically using wired connection (turn this back on see the previous step). TUNI note: we use 'roam.fi' (pass for all users as well). 
4. After the automatic connection is turned on, bind the VPN to this particular connections. So it will automatically connect to VPN after it is connected to the internet.

TUNI note: useful command: `nmap -sn 130.230.89.0/24`, `nmcli con show`

## Problem with `torch.nn.DataParallel()` and ThreadRipper processors

Sometimes you run your code in `torch.nn.DataParallel()` on 1+ GPUs and it just hangs. Take a look at the solution provided by one of the ?NVidia? developers: https://github.com/pytorch/pytorch/issues/1637#issuecomment-338268158
