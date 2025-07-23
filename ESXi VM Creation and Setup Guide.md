# ESXi VM Creation and Setup Guide - DNS Lab Environment

## Overview
This guide creates a complete DNS testing environment with two AlmaLinux VMs on VMware ESXi, featuring both internal isolated networking and external connectivity.

## VM Specifications
| Component | DNS Server | DNS Client |
|-----------|------------|------------|
| **OS** | AlmaLinux 9.6 | AlmaLinux 9.6 |
| **vCPUs** | 2 | 2 |
| **RAM** | 4GB | 4GB |
| **Disk** | 20GB | 20GB |
| **External IP** | 10.4.34.45/27 | 10.4.34.46/27 |
| **Internal IP** | 192.168.100.10/24 | 192.168.100.20/24 |
| **Gateway** | 10.4.34.33 | 10.4.34.33 |
| **Hostname** | dns-server | dns-client |

---

## Phase 1: Pre-Installation Setup

### Step 1: Download AlmaLinux ISO
1. Navigate to [https://almalinux.org/get-almalinux/](https://almalinux.org/get-almalinux/)
2. Click on **"AlmaLinux OS 9.6 Minimal ISO"**
3. Wait for download to complete (approximately 2GB)

### Step 2: Upload ISO to ESXi Datastore
1. Open **vSphere Client** in web browser: `https://your-esxi-ip`
2. Login with ESXi credentials
3. Navigate to **Storage** → Select your datastore
4. Click **Datastore browser**
5. Click **Create directory** → Name: `ISOs`
6. Enter the `ISOs` directory
7. Click **Upload files** → Select AlmaLinux ISO
8. Wait for upload completion

### Step 3: Configure External Network
1. Navigate to **Networking** → **Port groups**
2. Select **VM Network**
3. Click **Edit settings**
4. Configure **VLAN ID** to match your network requirements
5. Click **SAVE**

### Step 4: Create Internal Network Infrastructure
**Create Virtual Switch:**
1. Navigate to **Networking** → **Virtual switches**
2. Click **Add standard virtual switch**
3. Configure:
   - **Name**: `vSwitch1`
   - **MTU**: `1500` (default)
4. Click **ADD**

**Create Internal Port Group:**
1. Navigate to **Networking** → **Port groups**
2. Click **Add port group**
3. Configure:
   - **Name**: `Internal-Network`
   - **Virtual switch**: `vSwitch1`
   - **VLAN ID**: `0`
4. Click **ADD**

---

## Phase 2: Virtual Machine Creation

### Step 5: Create DNS Server VM
1. Navigate to **Virtual Machines** → **Create/Register VM**
2. **Select creation type**: Create a new virtual machine → **NEXT**
3. **Select name and guest OS**:
   - **Name**: `AlmaLinux-DNS-Server`
   - **Compatibility**: ESXi 8.0 (or latest available)
   - **Guest OS family**: Linux
   - **Guest OS version**: AlmaLinux 9 (64-bit)
   - Click **NEXT**
4. **Select storage**: Choose your datastore → **NEXT**
5. **Customize settings**:
   - **CPU**: 2 vCPUs
   - **Memory**: 4096 MB
   - **Hard disk 1**: 20 GB, Thin provisioned
   - **Network Adapter 1**: VM Network (external)
   - **Add other device** → **Network Adapter**
   - **Network Adapter 2**: Internal-Network
   - **CD/DVD Drive 1**: 
     - Select **Datastore ISO file**
     - Browse to `ISOs/AlmaLinux-9.6-x86_64-minimal.iso`
     - Enable **Connect at power on**
6. Click **NEXT** → **FINISH**

### Step 6: Create DNS Client VM
Repeat Step 5 with the following change:
- **Name**: `AlmaLinux-DNS-Client`
- All other settings remain identical

---

## Phase 3: Operating System Installation

### Step 7: Install AlmaLinux on DNS Server
1. Select **AlmaLinux-DNS-Server** VM
2. Click **Power on**
3. Click **Launch Web Console**
4. **Boot Screen**: Select "Install AlmaLinux 9.6"

**Installation Configuration:**
1. **Language**: English (United States) → **Continue**
2. **Installation Summary Configuration**:

   **Installation Destination:**
   - Select the 20GB disk
   - Click **Done**

   **Network & Host Name:**
   - **External Interface (ens34)**:
     - Toggle **ON**
     - Click **Configure**
     - **IPv4 Settings** → Manual:
       - **Address**: `10.4.34.45`
       - **Netmask**: `255.255.255.224` (or `/27`)
       - **Gateway**: `10.4.34.33`
       - **DNS servers**: `8.8.8.8, 8.8.4.4`
     - Click **Save**
   
   - **Internal Interface (ens35)**:
     - Toggle **ON**
     - Click **Configure**
     - **IPv4 Settings** → Manual:
       - **Address**: `192.168.100.10`
       - **Netmask**: `255.255.255.0` (or `/24`)
       - **Gateway**: `192.168.100.1`
     - Click **Save**
   
   - **Host name**: `dns-server`
   - Click **Done**

   **Root Password:**
   - Set strong password
   - Click **Done**

   **User Creation:**
   - Create administrative user
   - Enable "Make this user administrator"
   - Click **Done**

3. Click **Begin Installation**
4. Wait for installation completion (10-15 minutes)
5. Click **Reboot System**

### Step 8: Install AlmaLinux on DNS Client
Repeat Step 7 for **AlmaLinux-DNS-Client** with these network changes:
- **External Interface (ens34)**: `10.4.34.46/27`, Gateway: `10.4.34.33`
- **Internal Interface (ens35)**: `192.168.100.20/24`, Gateway: `192.168.100.1`
- **Host name**: `dns-client`

---


## Environment Ready
✅ **DNS Server VM**: AlmaLinux 9.6, dual-homed (external + internal)  
✅ **DNS Client VM**: AlmaLinux 9.6, dual-homed (external + internal)  
✅ **Internal Network**: Isolated 192.168.100.0/24 for DNS testing  
✅ **External Connectivity**: Internet access via 10.4.34.0/27 network  