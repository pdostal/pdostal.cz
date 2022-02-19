---
title: Installing openSUSE MicroOS in QEMU
date: 2022-02-19
tags:
  - linux
---

This is step by step guide for installing openSUSE MicroOS in QEMU.
This can be easy to customize or automate. It may help you on other platforms too.

<!--more-->

## Installation
 1) Download the QEMU openSUSE MicroOS image.
    ```
    # curl https://download.opensuse.org/tumbleweed/appliances/openSUSE-MicroOS.x86_64-kvm-and-xen.qcow2 -o microos1.qcow2
    ```
 2) Create 10MB Combustion image:
    ```
    # dd if=/dev/zero of=combustion.img  bs=1M  count=10
    # mkfs.ext4 -F combustion.img -L combustion
    # mkdir /mnt/combustion
    # mount combustion.img /mnt/combustion
    # mkdir /mnt/combustion/combustion
    ```
    If you install on real hardware, create the file system on formated disk instead.
 3) Save Combustion script as: `/mnt/combustion/combustion/script`:
    ```
    #!/bin/bash
    # combustion: network
    # Redirect output to the console
    exec > >(exec tee -a /dev/tty0) 2>&1
    # Set a password for root, generate the hash with "openssl passwd -6"
    echo 'root:EchlU.h2tvfOpOki54NaHpGYKwdNhjaBuSpDotD7' | chpasswd -e
    # Add a public ssh key and enable sshd
    mkdir -pm700 /root/.ssh/
    cat id_rsa.pub >> /root/.ssh/authorized_keys
    systemctl enable sshd.service
    # Install vim-small
    zypper --non-interactive install vim-small
    # Leave a marker
    echo "Configured with combustion" > /etc/issue.d/combustion
    ```
 4) Copy `id_rsa.pub` to the image and `umount` it.
 5) Create the QEMU VM:
    ```
    # virt-install --boot hd -n microos1 --os-type=Linux --os-variant=opensusetumbleweed --network network=default --noautoconsole --ram=8096 --vcpus=8 --disk path=microos1.qcow2 --autoconsole text --disk combustion.img
    ```
 6) You can undefine the combustion disk now but it is not necessary.

## Removal
```
virsh destroy microos1 # Stop the VM
virsh undefine microos1 # Undefine the VM
rm microos1.qcow2 # Remove the system image
```

## Resources:
 * [the openSUSE porta](https://en.opensuse.org/Portal:MicroOS/Downloads)
 * [openSUSE Portal/MicroOS/Combustion](https://en.opensuse.org/Portal:MicroOS/Combustion)

