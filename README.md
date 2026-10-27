# CYBERSECURITY-LAB-SETUP
Isolated virtual lab built with VirtualBox and Kali Linux for cybersecurity testing and penetration testing practice.
# 🔐 Cybersecurity Home Lab: VirtualBox + Kali Linux

> A self-contained virtual lab for practising ethical hacking and penetration testing, built on an isolated private network.

---

## 📌 Overview

This repository documents how I built a virtual cybersecurity lab on my own computer using **VirtualBox** and **Kali Linux**. The lab gives me a safe, repeatable place to run security tools without touching any real-world systems.

The lab sits on a private virtual network, so I can later add vulnerable target machines next to my Kali attacker machine and practise authorised testing against them.

## 🎯 Goals

- Install and set up VirtualBox as the hypervisor
- Import Kali Linux as a virtual machine
- Build a private **NAT Network** for the lab
- Give the Kali VM a fixed, predictable IP address
- Confirm connectivity (gateway, internet, DNS) and that tools work
- Save a clean snapshot as a restore point
- Document everything, including the problems I hit and how I fixed them

## 🛡️ What the Lab Is For

Once built, the lab supports hands-on practice in:

- Network reconnaissance and host discovery
- Port scanning
- Vulnerability assessment
- Packet capture and analysis
- Web application security testing
- Exploitation practice against deliberately vulnerable machines

> ⚠️ **Ethical use only.** Use this lab and its tools only on systems you own or have written permission to test. Attacking anything else is illegal and against the purpose of this project.

---

## 🏗️ Lab Architecture

```
![perma link](https://github.com/awanlaiba/CYBERSECURITY-LAB-SETUP/blob/1d425bd643f8635646ca211e803fa1f8eb5c587a/Network%20Architecture.png)
```

## ⚙️ Configuration Summary

| Item | Value |
|---|---|
| Host operating system | Windows 11 |
| Host RAM | 32 GB |
| Host CPU | AMD Ryzen 7 PRO  |
| Hypervisor | VirtualBox 7.2.22 |
| Guest OS | Kali Linux 2026.2 |
| Kali VM memory | 8 GB |
| Network type | VirtualBox NAT Network |
| Network range | 10.0.0.0/24 |
| Kali IP | 10.0.0.2/24 |
| Default gateway | 10.0.0.1 |
| DNS server | 8.8.8.8 |
| Reserved range for future VMs | 10.0.0.3 – 10.0.0.99 |

---

## 🪜 Setup Walkthrough

### Step 1: Install VirtualBox

**What I did:** Installed VirtualBox on Windows.
**Why:** VirtualBox is the hypervisor. It lets one physical computer run several isolated virtual machines, which is the foundation of the whole lab.

### Step 2: Create the NAT Network

**What I did:** In VirtualBox's network settings, I created a new NAT Network with these settings:

| Setting | Value |
|---|---|
| Name | NatNetwork |
| IPv4 prefix | 10.0.0.0/24 |
| DHCP | Enabled |
| IPv6 | Disabled |

📷 `<img width="959" height="566" alt="image" src="https://github.com/user-attachments/assets/c6b586f9-a00a-4611-bd23-beb28c46773b" />`

**Why:** A plain "NAT" adapter isolates each VM from the others, so VMs can reach the internet but not each other. A **NAT Network** puts multiple VMs on the same private subnet so they can talk to one another and still reach the internet through the host. That is exactly what an attacker-and-target lab needs.

### Step 4: Import Kali Linux

**What I did:** Downloaded the official Kali VirtualBox image, import it into VirtualBox and then adjust the VM:

```
Adapter 1
  Attached to : NAT Network
  Name        : NatNetwork
  Memory        : 8192 MB
```
📷 `<img width="959" height="521" alt="image" src="https://github.com/user-attachments/assets/5e7550b7-a7fa-4c31-93af-ddfa9bfe6d39" />`

**Why:** Attaching the adapter to the NAT Network places Kali inside the lab's private subnet. 

I also set up a shared folder so I can move files between Windows and the Kali VM.

📷 `<img width="581" height="382" alt="image" src="https://github.com/user-attachments/assets/cf13408e-0ec6-46b1-81d7-9eef49b00b44" />
`

### Step 5: Configure Kali's Network

**What I did:** Set a fixed IPv4 configuration on Kali:

```
IP address  : 10.0.0.2
Subnet mask : 255.255.255.0
Gateway     : 10.0.0.1
DNS         : 8.8.8.8
```

📷 `<img width="350" height="273" alt="image" src="https://github.com/user-attachments/assets/6650a124-d991-4af1-9a80-1066b613aadd" />
`

**Why:** A static address means Kali is always found at the same place. That makes lab notes, scan commands, and future exercises consistent, and DHCP can't shuffle the address between reboots.

### Step 6: Take a Clean Snapshot

**What I did:** Once everything worked, I created a VirtualBox snapshot named `Clean Kali - Network Setup`.

**Why:** Security testing often changes or breaks a system. The snapshot is a known-good baseline I can roll back to in seconds instead of rebuilding the lab.

---

## 🔎 Verification

| Check | Command | Expected outcome |
|---|---|---|
| 🌐 IP address is correct | `ip a` | Shows 10.0.0.2/24 |
| 📡 Gateway reachable | `ping 10.0.0.1` | Replies received |
| 🌍 Internet reachable | `ping 8.8.8.8` | Replies received |
| 🔎 DNS works | `nslookup amazon.com` | Domain resolves to an address |
| 🧰 Nmap installed | `nmap --version` | Version information shown |
| 🔄 Snapshot restores | Restore snapshot, then `ip a` | Baseline configuration returns |

📷 `![ip a output](<img width="332" height="264" alt="image" src="https://github.com/user-attachments/assets/9847c54f-fd54-4baf-b830-048191ebdf0d" />)`
📷 `![ping 10.0.0.1](<img width="329" height="161" alt="image" src="https://github.com/user-attachments/assets/de336194-0dc0-42c8-9f1a-0d086d073eac" />)`
📷 `![nslookup amazon.com](<img width="323" height="139" alt="image" src="https://github.com/user-attachments/assets/ee0b8a28-a7f8-4e69-9d23-6b738f8e7251" />)`
📷 `![nslookup nmap --version](<img width="324" height="88" alt="image" src="https://github.com/user-attachments/assets/f56b4702-8704-4b6e-8cf8-2245cad958da" />)`

Each test confirms a different layer: addressing, local network, routing out to the internet, name resolution, tooling, and recoverability.

---

## 🐞 Problems and Fixes

### Problem 1: No internet after setting a static IP

**Symptom:** After manually setting the IP, connectivity sometimes failed, depending on how NetworkManager handled the connection.

**Fix I used:**

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

I then restarted the connection (or rebooted) and re-tested.

**Why it helps:** Duplicate Address Detection (DAD) makes the system check whether another host is already using the IP before it brings the connection up. Setting the timeout to 0 skips that wait, which can avoid a stalled connection on a small, private lab network.

> 💡 Connection names vary between systems. Run `nmcli connection show` first and substitute your own connection's name.

---

## 💡 Key Takeaways

1. **NAT vs NAT Network.** Standard NAT isolates VMs from each other; a NAT Network lets them share a private subnet while still reaching the internet. This is what makes a multi-machine lab possible.
2. **Virtual networking.** Which adapter type and network a VM is attached to decides who it can talk to.
3. **Static addressing.** I practised setting and verifying IP, subnet mask, gateway, and DNS on Linux.
4. **Snapshots.** Take one before anything risky; it turns mistakes into a quick rollback.
5. **Troubleshooting.** Both problems were solved by reading symptoms, finding the underlying cause, and fixing that rather than guessing.
6. **Documentation.** Recording commands, settings, screenshots, and fixes is part of professional security work and makes the lab reproducible.

## 🌟 Why This Matters

A well-built lab is the foundation for nearly every later security skill. It lets me:

- Practise offensive and defensive techniques legally and safely
- Experiment freely, because a snapshot undoes any damage
- Reproduce results consistently, thanks to fixed addressing and documented settings
- Grow the environment over time with vulnerable targets, monitoring tools, or a firewall VM

It also mirrors how real security teams use isolated test environments, so the habits here carry over to professional work.

## 🔮 Next Steps

- Add a deliberately vulnerable target VM (for example Metasploitable or DVWA) in the 10.0.0.3–99 range
- Run Nmap discovery and service scans against it
- Capture traffic with Wireshark and analyse it
- Build on the lab with web testing and vulnerability scanning exercises

## 🔐 Security and Ethics

This lab is for education only. Test only systems you own or are explicitly authorised to test.

## 🔗 Tools and Resources

- VirtualBox: https://virtualbox.org/wiki/Downloads
- Kali Linux: https://kali.org/get-kali

## 👤 Author

**Laiba Awan**

LinkedIn: [www.linkedin.com/in/laiba-awan-98466723b]
