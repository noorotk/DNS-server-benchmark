# KVM Virtual Machine Setup

## Install the required packegs to run kvm:

```bash
dnf install qemu-kvm libvirt libvirt-python3 libguestfs-tools virt-install virt-manager virt-viewer
```

## First download the iso image:

```bash
wget https://repo.almalinux.org/almalinux/9.6/isos/x86_64/AlmaLinux-9.6-x86_64-minimal.iso
```

## Then make a disk to install the vm on:

```bash
qemu-img create -f qcow2 /var/lib/libvirt/images/almalinux-server.qcow2 20G
```

> qcow2 is the type of storage to run the virtual machine

## Then run on your local pc:

```bash
ssh -L 5901:localhost:5901 user@your-kvm-server-ip
```

## Then from the pc ssh session, run this to create the VM:

```bash
virt-install --name Almalinux-DNS-Server \
--ram 4096 \
--vcpus 2 \
--cpu host-model \
--hvm \
--disk path=/var/lib/libvirt/images/almalinux-server.qcow2,size=20 \
--cdrom /var/lib/libvirt/boot/AlmaLinux-9.6-x86_64-minimal.iso \
--os-variant almalinux9 \
--network network=default \
--graphics vnc,listen=0.0.0.0,port=5901
```

## Then run this from the local pc also to open gui to the installation:

```bash
vncviewer localhost:5901
```

## Then the vm should run succesfully, you can see the ip then ssh to it from the kvm server.

### Change hostname:

```bash
hostnamectl set-hostname [name]
```

Press `ctrl+d` then login again

## The vm is ready now, repeat the steps for the DNSperf VM.

---

## Some commands you need:

```bash
virsh list --all
virsh dominfo [name of the vm]       *****info about the vm
virsh destroy [name of the vm]
virsh undefine [name of the vm] --remove-all-storage
virsh domifaddr [name of the vm]     ****to show the ip of the vm
```
