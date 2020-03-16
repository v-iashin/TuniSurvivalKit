# How to setup a personal GPU desktop with Ubuntu 18.04

This is a guide which I wrote for myself when I had to install a similar enviroment on several machines. This instruction are quite shallow and there is must be a better way but it works.

The goal is to install Ubuntu 18.04 on a clean machine with a custom disk space allocation, VPN, booting into text mode rather than a GUI.

- [How to setup a personal GPU desktop with Ubuntu 18.04](#how-to-setup-a-personal-gpu-desktop-with-ubuntu-1804)
  - [Ubuntu installation](#ubuntu-installation)
  - [Set of packages](#set-of-packages)
  - [conda super-short introduction](#conda-super-short-introduction)
    - [Installation](#installation)
    - [Creating and Clonning an Enviroment](#creating-and-clonning-an-enviroment)
  - [Problem with `torch.nn.DataParallel()` and ThreadRipper processors](#problem-with-torchnndataparallel-and-threadripper-processors)
  - [Mouting the unformatted disks](#mouting-the-unformatted-disks)

## Ubuntu installation
Assuming that use have a bootable usb with Ubuntu 18.04 and a clean machine.

0. Make a bootable USB with the linux distribution. For a Mac you may use a free app `balenaEtcher`.
0. Insert the usb and reboot the machine into usb. Note: When you click `F11` and the list of bootable options will be prompted, try to boot into UEFI (EFI) rather than USB (BIOS) by selecting the entry with your USB and UEFI.
1. Click `Try Ubuntu` to boot into live version of Ubuntu.
2. Connect to internet. (TUNI note: connect to 'roam.fi'. it will ask your credentials, enter those and click `No certificate is required` and click connect.)
3. Click Install Ubuntu on desktop and proceed with installation.
4. Uncheck box with 3rd party drivers and download updates but check `minimal install` (available in 18.04). (if checked it sometimes fails when installing `16.04`, but I tried with it on 18.04 and it was okay)
5. When you will be ask about something like Automatic installation or 'Something else', select `Something else` to allocate disk space. 
5. Select the drive to boot from at the bottom of the window first.
6. Allocate the space. The window will ask you to partition your disks. First, select the disk from which you want your Ubuntu to boot (bottom of the window), if you don't want to save data from your home and other disks, then, free up the space on all drives if you have something on your drives (use minus icon), and proceed with allocating space for `/`, `/home`, `/boot`, etc. If you want to save your data on other disks which are on other disks which are not used for your system partition, then, only deal with the disk which you will use for the system. TUNI note: the desktops that we had so far had: 512GB SSD, 1TB NVMe, 8TB HDD. We recommend to use SSD for OS, NVMe for the data/model-logs for your current project, and HDD for archiving the data that you don't want to download or process once more. Therefore, select SSD for booting. On SSD allocate: 1024MB (512 MB) to EFI Partition (just enter the 1024 and select EFI partition from the menu, primary drive etc are by default. If you don't have EFI, only BIOS, you boot live usb into bios mode instead of UEFI, see Note to the first step), RAM+10GB allocate to swap area, 256 GB (128 GB) allocate for the `/` and the rest for `/home`. Keep in mind that other collegues might use the machine with you so they need some space on `/home`. So make sure you have enough on `/home`. (The next note about allocating space on HDD and NVMe. If you want to keep your data on the disks after reinstallation skip the next sentence and after installation use a guide how to mount the disks below ->) On NVMe allocate: the entire space to `/home/nvme/`. On HDD allocate: 8TB to `/home/hdd`.
7. (check once again that you selected the correct boot disk). Click Continue.
8. Name your machine and select username. Select password. TUNI note: ask the group first how to name your desktop because we all may use this desktop in the future (usually the convention is: `3x2080-12443` for a machine with three 2080 Ti and number `ws-12443-npc` - the code from the machine sticker).
9. Click install. Wait. Once done, remove the USB and Reboot. 
10. `sudo apt-get update && sudo apt-get upgrade`
10. (optionally) you may want to disable updates and do them manually. Go to update-manager and select switch the period of updates to `Never`.
11. Reboot
12. Download the latest drivers from nvidia web site (runfile)
13. `chmod +x <driver-file>` makes the drivers file to be executable.
14. Install dependencies `sudo apt-get install build-essential gcc-multilib dkms`
15. (Important to do now. The drivers will do it for you but you will need to restart, if you didn't blacklist nouveau) Create Blacklist for Nouveau Driver `sudo nano /etc/modprobe.d/blacklist-nouveau.conf` and add these lines there
```
blacklist nouveau
options nouveau modeset=0
```
16. Run `sudo update-initramfs -u`
17. Reboot. login into text mode instead of graphical interface (`ALT+CTRL+F2`).
17. Try to stop the graphical interface (you will be able to start it later): `sudo systemctl stop gdm3.service`.
18. Install drivers `sudo ./NVIDIA-Linux-x86_64-384.69.run --dkms -s` (note the `--dkms`, and `-s` at the end (`dkms` means to register dkms module into the kernel so that update of the kernel will not require a reinstallation of the driver; `-s` means to perform installation silently))
19. Check the installation `nvidia-smi`. You should see a list of GPUs and the drivers version on top of the table.
20. Reboot. You shuld be able to enter the ubuntu graphical interface.
21. (If you want to mount the disks which you haven't formated during the system installation, look at the end of this paragraph.) Allow your user to Read and Write to the created disks partitions (`/home/hdd` and `/home/nvme`). To fix this, open Nautilus with sudo rights `sudo nautilus`. Go to `/home/hdd`. Click right mouse and go to `Properties > Permissions`. Change both `Owner` and `Group` to your current username. Also, click `Change Permission for Encosed files` at the bottom on the window and give `Read` and `Write` and `Create and delete files permissions` to `Owner` and `Group`. Repeat for NVMe. If you didn't format the drives then you just need to mount them to your system (follow the guide provided later in this document).
22. (optional, this is not really required if you are using wired connection instead of a VPN. If your VPN disconnects _several times a day_ (not once a week which is normal) you may consider this option) Adjust sleeping behaviour (from https://askubuntu.com/questions/47311/how-do-i-disable-my-system-from-going-to-sleep) `sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target`
26. Setup VPN connection. See info below.
27. Turn on automatic boot into text-mode. `sudo systemctl set-default multi-user.target` (see also https://askubuntu.com/questions/1056363/how-to-disable-gui-on-boot-in-18-04-bionic-beaver).

## Set of packages
- `sudo apt install htop tmux cmake git`
- `sudo apt install openssh-server` (if throws errors with packages see https://askubuntu.com/questions/760378/can-not-install-openssh-server-on-ubuntu-16-04/795062)
- install `nvtop` (htop for gpus, follow instructions here https://github.com/Syllo/nvtop don't forget sudo before `make install`
- conda (see installation instructions later)
- if you need CUDA just for TensorFlow or PyTorch and not for `nvcc` (compiler) then just use conda to install CUDA and CuDNN binaries and you will not have any headache 

## conda super-short introduction
It is a good style to use conda virtual environments rather than installing your packages system-wise. Another cool thing about conda is that you don't need to worry about CUDA and CuDNN installation if you expect to use CUDA only for TensorFlow or PyTorch -- conda has binaries for CUDA and CuDNN which is enough to use these libraries without installing CUDA and CuDNN system-wise.

### Installation
1. Go to [miniconda](https://docs.conda.io/en/latest/miniconda.html#linux-installers) (or anaconda) and copy a link for Linux 64-bit.
2. Use `curl -O your_link` to download it.
3. During installation select `yes` when it will prompt whether to add into to `.bashrc`. 
4. Don't dorget to `source .bashrc` afterwards to be able to run conda command.

### Creating and Clonning an Enviroment
Creating
```
conda create -n my_environment_name
conda activate my_environment_name
conda install python=3.6
```
Clonning
```
conda activate your_env_name
conda env export > environment.yml
```
to create the same enviroment on another machine, transfer `environment.yml` there and run
```
conda env create -f environment.yml
```

More on [manage-environments](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html).

## Problem with `torch.nn.DataParallel()` and ThreadRipper processors
Sometimes you run your code in `torch.nn.DataParallel()` on 1+ GPUs and it just hangs. Take a look at the solution provided by one of the ?NVidia? developers: https://github.com/pytorch/pytorch/issues/1637#issuecomment-338268158

## Mouting the unformatted disks
If you didn't format the drives then you just need to mount them to your system. We are going to follow the guide from [askubuntu.com](https://askubuntu.com/a/125277/583662). 
1. Run `sudo fdisk -l` and find out the partition you want to additionally mount (in my case it is something like `/dev/sdb1` or `/dev/nvme0n1p1` -- please note `1` and `p1` at the end of partitions -- you will need this one and NOT SOMETHING LIKE: `/dev/sdb` and `/dev/nvme0n1`; also make sure you _haven't_ selected your system partition which is usually `/dev/sda*`); 
2. create a mounting points for your disks (for example `sudo mkdir /home/hdd` and `sudo mkdir /home/nvme`); 
3. Open `/etc/fstab` file with root permissions `sudo nano /etc/fstab`; 
4. To mount the disks permanentaly (after every boot), append two lines (or whether number of extra disks you want to add): `/dev/sdb1    /home/hdd    ext4    defaults    0    0` and `/dev/nvme0n1p1    /home/nvme    ext4    defaults    0    0` to the file and save it; 
5. Mount the disks for this boot: `sudo mount /home/hdd` and `sudo mount /home/nvme`.