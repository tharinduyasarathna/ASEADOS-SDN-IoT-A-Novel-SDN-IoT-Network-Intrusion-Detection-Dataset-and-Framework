
# ASEADOS-SDN-IoT: A Novel SDN–IoT Intrusion Detection Dataset and Testbed Framework

## Overview
ASEADOS–SDN–IoT is a publicly available and fully documented intrusion detection dataset built from a hybrid SDN–IoT testbed integrating physical IoT devices, virtual nodes, multi-segment routing, and ONOS controller telemetry.  
It contains **457,044 labeled flows** with **84 statistical features**, capturing both benign traffic and four major attack classes: **DoS, DDoS, Botnet, and Probe**.

## Testbed Architecture
- **Physical IoT devices**: Raspberry Pi boards, Amazon Echo Show, Echo Dot.
- **Virtual IoT nodes**: Flask/Python-based sensors (temperature, humidity, pressure, light, motion).
- **SDN Infrastructure**: ONOS controller + OVS with 5 logical bridges.
- **Attack VMs**: Kali Linux, Metasploitable2.
- **Hybrid Layer 2/Layer 3 routing** for cross-subnet communication.

## Dataset Composition
### Traffic Classes
- **Benign**: IoT telemetry, service traffic (HTTP/HTTPS, FTP, DNS, SSH)
- **DoS** attacks (HULK, LOIC, Slowloris)
- **DDoS** attacks (Hping3 SYN/UDP/ICMP floods)
- **Botnet** activity (BoNeSI)
- **Probe** scans (Nmap, Metasploit reconnaissance/exploitation)

### Labeled Flow Distribution
| Class | Flows | % |
|-------|-------|-----|
| Benign | 260,797 | 57.06 |
| DoS | 127,772 | 27.96 |
| DDoS | 51,465 | 11.26 |
| Bot | 9,090 | 1.99 |
| Probe | 7,920 | 1.73 |
| **Total** | **457,044** | **100%** |

## 🔧 Complete Testbed Setup 

This section provides the full SDN–IoT testbed setup used to generate the ASEADOS–SDN–IoT dataset.  
It includes VM configuration, ONOS installation, OVS routing, Mininet IoT topology, and attack machine setup.

---

# 🖥️ **VM2 — Ubuntu 16.04 (OVS + Mininet + Flask)**

### 1. Update Packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### 2. Install OVS + Mininet
```
sudo apt-get install openvswitch-switch mininet
sudo ufw disable
```

# 🖥️ **VM4 — ONOS Controller Setup**

### 1. Create ONOS User
```
sudo adduser sdn --system --group
```

### 2. Install Dependencies
```
sudo apt install git zip curl unzip python2-minimal:i386 openjdk-11-jdk -y
```

### 3. Install ONOS 2.6.0
```
cd /opt
sudo wget <ONOS-TAR-GZ-LINK>
sudo tar xzzf onos-2.6.0.tar.gz
sudo mv onos-2.6.0 onos
sudo chown -R sdn:sdn onos
```

### 4. Configure ONOS Options
In `/opt/onos/options`:
```
export ONOS_USER=sdn
export ONOS_APPS=drivers,openflow,gui2
```

### 5. Enable Service
```
sudo systemctl enable onos
sudo systemctl start onos
```

### 6. Access
- UI → http://127.0.0.1:8181/onos/ui  
- CLI → `ssh -p 8101 onos@localhost` (pass: rocks)

---

# 🖥️ **VM1 — Kali Linux (Attack Host)**

Assign IP:
```
sudo ifconfig eth1 200.175.2.130/24 up
sudo ip route add default via 200.175.2.129
```

Tools used in the dataset:
- Nmap  
- Hydra  
- Slowloris  
- Hping3  
- Metasploit  

---

# 🖥️ **VM3 — Metasploitable2 (Victim)**

Assign IP:
```
sudo ifconfig eth0 192.168.3.130/24 up
sudo ip route add default via 192.168.3.129
```

---

# 🔀 **OVS Layer‑3 Routing (VM2)**

### Add IPs:
```
auto ens38
iface ens38 inet static
address 192.168.3.129
netmask 255.255.255.0
```
(similarly for ens39, ens40, ens41)

### Create Bridges
```
sudo ovs-vsctl add-br br1
sudo ovs-vsctl add-br br2
```

### Attach Interfaces
```
sudo ovs-vsctl add-port br1 ens39
sudo ovs-vsctl add-port br2 ens37
```

### Assign IPs
```
sudo ip addr add 200.175.2.129/24 dev br1
sudo ip addr add 192.168.3.129/24 dev br2
```

### Connect to ONOS
```
sudo ovs-vsctl set-controller br1 tcp:192.168.8.128:6653
sudo ovs-vsctl set-controller br2 tcp:192.168.8.128:6653
```

---

# 🌐 **Mininet IoT Simulation (VM2)**

`topology.py`:
```
from mininet.net import Mininet
from mininet.node import OVSBridge, Host
from mininet.link import Link

net = Mininet()
s1 = net.addSwitch('s1', cls=OVSBridge)
s1_ip = '192.168.20.129/24'
s1.cmd('ifconfig s1 ' + s1_ip)

h1 = net.addHost('h1', ip='192.168.20.131/24', defaultRoute='via ' + s1_ip)
h2 = net.addHost('h2', ip='192.168.20.132/24', defaultRoute='via ' + s1_ip)
h3 = net.addHost('h3', ip='192.168.20.133/24', defaultRoute='via ' + s1_ip)
h4 = net.addHost('h4', ip='192.168.20.134/24', defaultRoute='via ' + s1_ip)

Link(h1, s1)
Link(h2, s1)
Link(h3, s1)
Link(h4, s1)

net.start()
net.pingAll()
net.interact()
```

Run:
```
sudo python topology.py
```

Attach S1 to OVS:
```
sudo ovs-vsctl add-port s1 ens41
sudo ip addr add 192.168.20.129/24 dev s1
sudo ovs-vsctl set-controller s1 tcp:192.168.8.128:6653
```

Set routes inside Mininet:
```
h1 ip route add default via 192.168.20.129
...
```

---

# 📥 **Traffic Capture for Dataset**

- `tcpdump` on OVS bridges  
- `tcpdump` on IoT nodes  
- `tcpdump` on attack/victim devices

This produces the multi-perspective data used to build the ASEADOS‑SDN‑IoT dataset.

