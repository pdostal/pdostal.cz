---
title: Installing openSUSE MicroOS and Tumbleweed in QEMU
date: 2022-02-19
tags:
  - linux
---

This is a step by step guide for installing openSUSE MicroOS within QEMU.
This can be easy to customize or automate. It may help you on other platforms too.

<!--more-->

## Installation
 1) Download the QEMU openSUSE MicroOS image.
    ```
    # curl https://download.opensuse.org/tumbleweed/appliances/openSUSE-MicroOS.x86_64-kvm-and-xen.qcow2 -o microos1.qcow2
    ```
 2) Create your combustion script on your local computer as: `/mnt/combustion/combustion/script` 
    ```
    #!/bin/bash
    # combustion: network

    # Redirect output to the console
    exec > >(exec tee -a /dev/tty0) 2>&1

    # Set a password for root, generate the hash with "openssl passwd -6"
    echo 'root:EchlU.h2tvfOpOki54NaHpGYKwdNhjaBuSpDotD7' | chpasswd -e

    # Add a public ssh key and enable sshd
    mkdir -pm700 /root/.ssh/
    cat << EOF >> /root/.ssh/authorized_keys
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCW1AFnflfnPfKg12oAlESachI/BKlMXQ4CpAHBqrVBqlyb4bbma9fNNAlomD+qzKkorxRGIkxK/6WwIwTZpaiz84H+FlpIrPJeiz1jDeSqW+apP/trh95CvwKgCocXZQeA8O0lRy1RtPZYwKNONR427urIYTavrFcxpqeEYD/C2A+v4QdGlSJQHXsMbgZ2egFJlG7n/b0o4tJAH4YmiiKIECq7S03SHWzd8d5ZKzSxwGq35C/HBLBcgvi47jSQZC9H4eyuFY8xSb+fvi4jPXaZlbDlDaJJ9GrwgWbjyJIiwyMAhrxlaG31DmcJoMecxZ0f/ZXIDCSzSG02N93SsBH7 pavel@laptop.pdostal.cz
    EOF
    systemctl enable sshd.service

    # Install nmap
    zypper --non-interactive install nmap

    # Leave a marker
    echo "Configured with combustion" > /etc/issue.d/combustion
    ```
 3) If you install MicroOS on real hardware or other hypervisor, you need to create a Combustion disk image.
    ```
    # dd if=/dev/zero of=/dev/sdX  bs=1M  count=10
    # mkfs.ext4 -F /dev/sdX -L combustion
    # mkdir /mnt/combustion
    # mount /dev/sdX /mnt/combustion
    # mkdir /mnt/combustion/combustion
    ```
    Please use something like `/tmp/combustion.img` instead of `/dev/sdX` in case you install on another hypervisor.
 4) Create the QEMU VM:
    ```
    # I=01; virt-install --boot hd -n microos$I --os-type=Linux --os-variant=opensusetumbleweed --network network=default,mac=52:54:00:f2:fb:$I --ram=4096 --vcpus=4 --disk path=microos$I.qcow2 --noautoconsole --graphics vnc,listen=0.0.0.0,port=59$I --qemu-commandline="-fw_cfg name=opt/org.opensuse.combustion/script,file=/var/lib/libvirt/images/combustion"
    ```
 5) The VM can be accessed over the SSH, by `virsh console microos01` or via VNC.

## Installing Tumbleweed or MicroOS from the network via autoYAST
```
# virt-install -n microos --os-variant=opensusetumbleweed --network network=default --ram=8192 --vcpus=4 --disk path=microos.qcow2,size=40 --location http://download.opensuse.org/tumbleweed/repo/oss/ --extra-args 'autoyast=https://pdostal.cz/microos-autoyast.xml' --nographics --extra-args "console=ttyS0,115200 textmode=1" --boot uefi
```
The above example installs MicroOS but when switching or skipping the autoyast profile we can install Tumbleweed as well.

If you wanna boot from PXE then the `kernel` and `initrd` files are in the same location under `boot/x86_64/loader/` path.

## Serial console installation with UEFI and secure boot enabled
```
# virt-install -n tumbleweed --os-variant=opensusetumbleweed --network network=default,mac=52:54:00:f2:fb:67 --ram=8192 --vcpus=4 --disk path=tumbleweed.qcow2,size=40 --nographics --location http://download.opensuse.org/tumbleweed/repo/oss/ --extra-args "console=ttyS0,115200 textmode=1" --boot uefi,loader_secure=yes
```

There are two magical boot parameters: `console=ttyS0,115200` which outputs the console to the serial terminal and `textmode=1` which spawns the YaST in the text mode. Let's also add `--nographics` as we don't really need it.

To enable UEFI and secureboot please use the `--boot uefi,loader_secure=yes` parameter. With this option openSUSE can also be installed with trusted boot enabled.

To install openSUSE Leap instead use `http://download.opensuse.org/distribution/leap/15.5/repo/oss/` as `--location` parameter.

## Removal
```
virsh destroy microos1 # Stop the VM
virsh undefine microos1 # Undefine the VM
rm microos1.qcow2 # Remove the system image
```

## Resources:
 * [the openSUSE portal](https://en.opensuse.org/Portal:MicroOS/Downloads)
 * [openSUSE Portal/MicroOS/Combustion](https://en.opensuse.org/Portal:MicroOS/Combustion)
 * [MicroOS autoYAST profile](/microos-autoyast.xml)

