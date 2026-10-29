# Week 1 Project: Cybersecurity Lab Setup (Kali Linux on VirtualBox)

Week 1 internship project: an isolated cybersecurity and ethical hacking practice lab built with **VirtualBox** and **Kali Linux** on a private **NAT Network** (`10.0.0.0/24`).

> This lab is for learning and practice in a safe, isolated environment only. Use it only on systems you own or have permission to test.

---

## Objectives

- Use VirtualBox as the base (v7.2.16)
- Set up Kali Linux as the attacker machine
- Create a NAT Network with subnet `10.0.0.0/24`
- Enable shared clipboard and drag-and-drop
- Enable a shared folder (host `Downloads` folder)
- Kali IP address: `10.0.0.2/24`
- Full Internet access from Kali

---

## Lab Configuration

| Component | Details |
|---|---|
| Host OS | Windows |
| Hypervisor | Oracle VirtualBox 7.2.16 |
| Attacker VM | Kali Linux (installed from `kali-linux-2025.4-installer-amd64.iso`) |
| VM Resources | 6 GB RAM (6269 MB), 2 CPU cores |
| Network | NAT Network `NatNetwork` - `10.0.0.0/24` (DHCP enabled) |
| Kali IP | `10.0.0.2/24` (netmask `255.255.255.0`) |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` (fallback: `10.0.0.1`) |

```
 Windows Host (VirtualBox)
          |
   NatNetwork 10.0.0.0/24
          |
   Kali Linux  10.0.0.2
   (Gateway 10.0.0.1)
          |
       Internet
```

---

## Setup Steps

### Step 1: 7-Zip
Skipped, because WinRAR was already installed (7-Zip is only needed to extract archives).

### Step 2: Install VirtualBox
Installed VirtualBox **7.2.16** from https://virtualbox.org/wiki/Downloads

![VirtualBox version](screenshots/01-virtualbox-version.png)

### Step 3: Create the NAT Network
1. Open VirtualBox, then **File -> Tools -> Network**
2. Open the **NAT Networks** tab and click **Create**
3. Name: `NatNetwork`
4. **IPv4 Prefix: `10.0.0.0/24`** (see the troubleshooting section below)
5. Keep **Enable DHCP** checked and click **Apply**

![Tools -> Network menu](screenshots/02-tools-network-menu.png)
![NAT Network settings](screenshots/03-nat-network.png)

### Step 4: Create the Kali Linux VM
I used the Kali Linux ISO from https://kali.org/get-kali

- Type: Linux / Debian (64-bit)
- Memory: **6 GB**
- CPU: **2 cores**
- Network: **Adapter 1 -> Attached to: NAT Network -> Name: NatNetwork**
- Start the VM and verify that it boots

![New VM](screenshots/04-new-vm.png)
![Memory](screenshots/05-vm-memory.png)
![CPU](screenshots/06-vm-cpu.png)
![Network adapter](screenshots/07-vm-network-adapter.png)
![Kali desktop](screenshots/08-kali-desktop.png)

### Step 5: Configure the IP address in Kali
1. Open the network settings of the wired connection in Kali
2. Go to the **IPv4** tab and set the method to **Manual**
3. Enter:
   - Address: `10.0.0.2`
   - Netmask: `255.255.255.0`
   - Gateway: `10.0.0.1`
   - DNS: `8.8.8.8` (use `10.0.0.1` if there are Internet problems)
4. Click **Apply**, then test connectivity with `ping` and Firefox.

![Kali IPv4 settings](screenshots/09-kali-ip-config.png)
![Ping test](screenshots/10-ping-test.png)
![Internet in Firefox](screenshots/11-internet-firefox.png)

### Step 6: Clipboard and Drag-and-Drop
VM **Settings -> General -> Features**:
- Shared Clipboard: **Bidirectional**
- Drag-and-Drop: **Bidirectional**

![Clipboard and drag-drop](screenshots/12-clipboard-dragdrop.png)

### Step 7: Shared Folder
VM **Settings -> Shared Folders -> Add**:
- Folder Path: host `Downloads` folder
- Check **Auto-mount** and **Make Global**

![Shared folder](screenshots/13-shared-folder.png)

### Step 8: Snapshot
Took a snapshot of the clean, working VM so I can roll back at any time.

![Snapshot](screenshots/14-snapshot.png)

---

## Lab Verification

| Test | Command | Expected Result |
|---|---|---|
| Check IP address | `ip a` | `10.0.0.2/24` shown |
| Test Internet | `ping -c 3 8.8.8.8` | 0% packet loss |
| Test DNS / browser | Open `google.com` in Firefox | Page loads |

![ip a output](screenshots/15-ip-a.png)

---

## Troubleshooting: Issue I Faced

### Problem
After setting up the VM, the Kali IP was not working and there was no Internet connection.

### Root Cause
While creating the NAT Network, I entered the wrong IPv4 prefix: **`10.0.2.0/24`** instead of **`10.0.0.0/24`**. The NAT Network's subnet and gateway (`10.0.2.x`) did not match the static IP I gave Kali (`10.0.0.2/24`, gateway `10.0.0.1`). The VM and its gateway were on different subnets, so no traffic could be routed.

### Fix
1. Open **File -> Tools -> Network -> NAT Networks**
2. Select `NatNetwork` and open its properties
3. Change **IPv4 Prefix** from `10.0.2.0/24` to **`10.0.0.0/24`**
4. Click **Apply**
5. Shut down the Kali VM completely and start it again
6. Verify:
   ```bash
   ip a
   ping -c 3 10.0.0.1
   ping -c 3 8.8.8.8
   ping -c 3 google.com
   ```

### Lesson Learned
The IP, gateway and netmask configured inside the VM must always belong to the same subnet as the NAT Network created in VirtualBox. A small typo in the prefix (`10.0.2.0` vs `10.0.0.0`) breaks the whole network.

---

## General Troubleshooting Checklist (No Internet in Kali)

1. Check that the NAT Network was created correctly (`10.0.0.0/24`, DHCP enabled)
2. Check that the VM adapter is attached to **NAT Network -> NatNetwork**
3. Check IP / netmask / gateway / DNS inside Kali
4. Make sure no other VM on the same NAT Network uses `10.0.0.2`
5. Try DNS `10.0.0.1` instead of `8.8.8.8`
6. If the issue persists (common on VirtualBox 7 with newer Kali), run:
   ```bash
   sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
   sudo nmcli connection down "Wired connection 1"
   sudo nmcli connection up "Wired connection 1"
   ```
   (The connection name may differ on your system; check it with `nmcli connection show`.)
7. Restart the VM, and if needed the host OS

---

## What I Learned

- A NAT Network lets multiple VMs talk to each other and still reach the Internet
- Static IP, gateway and netmask must match the virtual network's subnet
- Snapshots give a clean recovery point before experiments
- Documenting problems and fixes is part of professional work

---

## Documentation

Full step-by-step document with screenshots: `Lab Setup Week 1 Project.docx`

## Next Steps (Phase 2 - Optional)

- Add Windows 10/11 and other target VMs to the same NAT Network
- Ping test between all machines
- Take snapshots of each VM

---

## Author

Malik Muhammad Faheem Badar
Cybersecurity Intern, Week 1 Project
[LinkedIn](https://linkedin.com/in/your-profile) | [GitHub](https://github.com/your-username)
