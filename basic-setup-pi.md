# Setup Raspberry Pi

## Image OS
Download Raspberry Pi Image Builder

https://www.raspberrypi.com/software/

Select Other OS-> Ubuntu Server

Configure Settings when imaging:
- Wifi SSID and password
- Password for user "Ubuntu"
- Public SSH key from your computer to be able to login with no password `ubuntu@<IP>`

Login using keyboard with `ubuntu` and password you setup in Image Builder, or set new password

## IP Reservation
Get the IP Address from welcome message or running `ip a` it will be something like `192.168.x.y`

Configure your router to always give the same IP address to the device by creating a reservation

## SSH setup

As convienience setup your local ssh config by adding a new entry in `$HOME/.ssh/config` for example
```
Host pi5
    HostName 192.168.1.231
    User ubuntu
```

Now test the ssh connection
```bash
ssh ubuntu@pi5
```
To setup ssh client create a key pair in your raspberry pi OS
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
Copy the public key content and add the ssh key
```bash
cat ~/.ssh/id_rsa.pub
```
Add entry in Github to be able to git clone and push https://github.com/settings/ssh/new
More info in https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent


## Setup Hostname

If you used the Image Builder settings, the system is using cloud-init to setup the OS.
If you want to change the hostname afterwards you need to create a new file
```bash
sudo sh -c 'echo "preserve_hostname: true" > /etc/cloud/cloud.cfg.d/99_custom_hostname.cfg'
```
Now you can set the hostname:
set your hostname
```bash
sudo hostnamectl set-hostname ${NEW_HOSTNAME}
```
Then reboot the raspberry pi using `sudo reboot now`


### Enable cgroups for the kernel

Enable cgroups for the kernel by appending to the `cmdline.txt` file
- Append to the one line, do not add a new line, a backup copy `cmdline.txt.bak` is made.

```bash
BOOT_DRIVE=/boot/firmware
sudo sed -i.bak 's/$/ cgroup_memory=1 cgroup_enable=memory/' ${BOOT_DRIVE}/cmdline.txt
cat ${BOOT_DRIVE}/cmdline.txt
```
Then reboot the raspberry pi using `sudo reboot now`


## Install GH CLI
Follow the instructions here https://github.com/cli/cli/blob/trunk/docs/install_linux.md

Clone this repository
```bash
gh repo clone csantanapr/k8s-raspberry-pi
```
Setup your name and email for git
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

## VSCode
You can use VSCode and connect to your raspberry pi. Then you can install extensions.

Select "Remote Explorer" from the left side menu tasks, then select your raspberry pi

Open the directory `~/k8s-raspberry-pi` as workspace, open a new terminal, this will be a prompt in your raspberry pi

You can install extentions like "Amazon Q" and use the Free tier, This give you an AI assitance in VSCode

## SSD
If you have a NVME SSD using an adapter you can configure like this

Assume this is the output of your `lsblk`
```
loop0         7:0    0  33.7M  1 loop /snap/snapd/21467
loop1         7:1    0  33.7M  1 loop /snap/snapd/21761
mmcblk0     179:0    0  59.5G  0 disk
├─mmcblk0p1 179:1    0   512M  0 part /boot/firmware
└─mmcblk0p2 179:2    0    59G  0 part /
nvme0n1     259:0    0 238.5G  0 disk
```

The SSD device name is `nvme0n1`

Create a filesystem on the NVMe disk and Create a directory to mount the NVMe disk:
```bash
sudo mkfs.ext4 /dev/nvme0n1
sudo mkdir /mnt/nvme
```

Add an entry to the `/etc/fstab` file to automatically mount the NVMe disk on boot:
```bash
sudo nano /etc/fstab
```
Add the following line to the file:
```
/dev/nvme0n1 /mnt/nvme ext4 defaults,nofail 0 2
```
Mount the NVMe disk:
```bash
sudo systemctl daemon-reload
sudo mount /mnt/nvme
```
Verify
```bash
df -h /dev/nvme0n1
```
Expected output (ie. 256GB SSD)
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1    234G   28K  222G   1% /mnt/nvme
```