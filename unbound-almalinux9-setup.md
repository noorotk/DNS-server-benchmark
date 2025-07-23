# Unbound Installation for AlmaLinux 9

```bash
dnf update -y
dnf install epel-release -y
dnf install unbound vim htop bind-utils -y
```

## turn off firewalld and selinux

```bash
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```

## download the query file from github

```bash
mkdir query-file
cd query-file/
curl -o unbound-local-queries https://raw.githubusercontent.com/noorotk/queryfiletest/main/unbound-local-queries
cd
```

## backup unbound original config

```bash
cp /etc/unbound/unbound.conf /etc/unbound/unbound.conf.backup
```

## Add this in `/etc/unbound/unbound.conf`

```
interface: 192.168.122.139
access-control: 0.0.0.0/0 refuse
access-control: 127.0.0.0/8 allow
access-control: 192.168.122.0/24 allow
include: "/etc/unbound/local-zones/*.conf"
```

```bash
systemctl restart unbound
```

## create the local records file

```bash
mkdir /etc/unbound/local-zones
cp query-file/unbound-local-queries /etc/unbound/local-zones/unbound-local-queries.conf
```

## try to dig some domain found in unbound-local-queries to test if unbound is working

```bash
dig google.com @192.168.122.139
```

## turn off the internet access to the VM and start the benchmark

```bash
ip route del default
```
