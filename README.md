# Ruuvitag Reader for Raspberry Pi Zero W

This is the repository for preparing a Raspberry Pi Zero W for reading Ruuvitag messages.

## Prerequisites

* Ruuvitag
* Raspberry Pi Zero W
* Micro SD -cardreader

## Quickstart

### Writing a pre-made image

On OSX run the following command to find out which disk the SD card is `diskutil list`. Check which disk is the SD card (e.g. */dev/disk2*). **NOTICE: If you write on a wrong disk, you will lose data.**

Then unmount that disk `diskutil unmountDisk /dev/diskX`, where *diskX* is the disk you picked from the previous command.

Download the ready-made Raspberry Pi Zero -image `wget https://www.dropbox.com/s/egl2t7r0143natx/ruuvitag-raspberrypi-zero-20200126.img.gz`.

Finally write it one the SD card `gunzip -c ruuvitag-raspberrypi-zero-20200126.img.gz | sudo dd bs=1m of=/dev/rdisk2`.

**Now you are ready to unmount the SD card, insert it into your Raspberry Pi and Boot.**

### Reading Ruuvitag

SSH into the Pi with `ssh -i keys/pi.pem pi@<ip address>`, where *ip address* is the address of your Raspberry Pi.

Create a directory and cd into it, e.g. `mkdir ruuvi && cd ruuvi`.

Link the globally installed [node-ruuvitag](https://github.com/pakastin/node-ruuvitag) with `npm link node-ruuvitag`.

Open node console (with `node`) and copy-paste the following code:

```node
const ruuvi = require('node-ruuvitag');

ruuvi.on('found', (tag) => {
  console.log(`Found RuuviTag with id: ${tag.id}`);

  tag.on('updated', (data) => {
    console.log(`${tag.id}: ${JSON.stringify(data)}`);
  });
});
```

Once you press enter you will see the logging from nearby ruuvitags.

**NOTICE:** You can also copy `index.js` into the directory you created and run it with `node index.js`.

## Creating the image

### Prerequisites

* [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Writing Raspbian on the SD card

Download [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) Lite (Buster at the time of writing).

On OSX run the following command to find out which disk the SD card is `diskutil list`. Check which disk is the SD card (e.g. */dev/disk2*). **NOTICE: If you write on a wrong disk, you will lose data.**

Then unmount that disk `diskutil unmountDisk /dev/diskX`, where *diskX* is the disk you picked from the previous command.

Format the disk with `sudo newfs_msdos -F 32 /dev/diskX`, where *diskX* is the same disk as in the previous command.

Finally write the raspbian image on the disk with the command below. Notice that the disk is prepended with a letter *r*. Using the *raw disk* for writing makes writing the data much faster.
    
    sudo dd bs=1m if=2019-09-26-raspbian-buster-lite.img of=/dev/rdisk2

### Enabling Wifi and SSH

To enable SSH on the Raspberry on boot time, add an empty file called *ssh* into the */boot* partition of the SD card. You will find such file in the *boot*-directory of this repository, but you can also create it with e.g. `touch /boot/ssh`.

To enable Wifi, modify the *boot/wpa_supplicant.conf* with your locale and Wifi information, and add it to the */boot* partition of the SD card. **NOTICE: Raspberry Pi Zero does not support 5GHz wifi.**

To disable *ipv6*, add `ipv6.disable=1` to the end of the first line of the *cmdline.txt*-file in the */boot* partition of the SD card. You will find a pre-made *cmdline.txt* for the current Buster (2019-09-26 at the time of writing) in the *boot*-directory.

### Generating an SSH key for accessing the Raspberry Pi

To generate an SSH secret key, run `openssl genrsa 4096 > pi.pem`. Then change the access control so that its not accessible to others `chmod 600 pi.pem`.

Next generate a public key from the secret key `ssh-keygen -y -f pi.pem > key.pem.pub`.

**Now you are ready to unmount the SD card, insert it into your Raspberry Pi and Boot.**

**NOTICE: Remember to change the default password on the first login.**

### Provisioning

Once the Raspberry Pi has booted and you know its ip-address (either attach a screen, do an ip scan or see your router admin), it is time to provision the Raspberry Pi Zero. This guide will refer to this address as `<ip address>`.

#### Add your SSH key to authorized keys

First add the key you generated previously as a trusted key by running `ssh-copy-id -i pi.pem pi@<ip address>` and using the default password. **Remember to change your password.** You can also remove password login from SSH altogether by setting `PasswordAuthentication no` in the configuration file `/etc/ssh/sshd_config` and either rebooting your Pi, or the SSH service (using `sudo service ssh restart`).

You can also login to your Pi with ssh and add the contents of `pi.pem.pub` to `~/.ssh/authorized_keys`, but `ssh-copy-id` will do that for you easier.

#### Provision your Raspberry Pi

Copy the `hosts.example` -file to e.g. `hosts.local` and replace `<ip address>` with the ip address of your Pi.

Run `ansible-playbook provisioning/playbook.yml -i provisioning/hosts.local`

