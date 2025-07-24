# DNSPerf installation for almaLinux 9
-----------------------------------------------------------

## update system

```bash
dnf update -y
```

## install basic tools

```bash
dnf install -y epel-release git vim wget curl net-tools bind-utils
```

## disable firewall for testing

```bash
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```

## disable SELinux for initial testing

```bash
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
```

## enable codeready builder (CRB) repository and install build dependencies

```bash
dnf config-manager --set-enabled crb
dnf makecache
dnf install -y openssl-devel ldns-devel ck-devel libnghttp2-devel make gcc libtool autoconf libcap-devel libpcap-devel
```

## clone and build DNSPerf

```bash
git clone https://github.com/DNS-OARC/dnsperf.git
cd dnsperf
./autogen.sh
./configure
make
make install
```

## verify installation

```bash
dnsperf -v
```

## download query file for testing

```bash
cd
mkdir query-file
cd query-file/
curl -o dnsperf-queries https://raw.githubusercontent.com/noorotk/queryfiletest/main/dnsperf-queries
```

## run a basic benchmark test

```bash
dnsperf -s 192.168.xx.xx -d dnsperf-queries -t 10
```

## turn off the internet access to the VM and start the benchmark

```bash
ip route del default
```
