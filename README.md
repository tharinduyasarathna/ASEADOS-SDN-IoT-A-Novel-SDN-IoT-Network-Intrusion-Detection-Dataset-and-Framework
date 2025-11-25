# ASEADOS-SDN-IoT-A-Novel-SDN-IoT-Network-Intrusion-Detection-Dataset-and-Framework
ASEADOS–SDN–IoT, a novel and publicly available intrusion detection dataset supported by a fully documented hybrid testbed framework. The resulting dataset contains 457,044 labeled flow instances with 83 statistical features and was evaluated using state-of-the-art machine learning (ML) and deep learning (DL) techniques.



# ASEADOS-SDN-IoT Testbed & Dataset Reproduction Guide

This README consolidates the entire SDN–IoT testbed setup, architecture, and dataset-generation methodology based on the ASEADOS-SDN-IoT paper.

## 1. Background

The ASEADOS-SDN-IoT dataset was created to address limitations in previous SDN intrusion datasets by integrating realistic IoT traffic, multi-segment routing, modern cyberattacks, and SDN controller telemetry.

## 2. High-Level Architecture

The system includes:
- VM2: Ubuntu 16.04 with OVS, Mininet, DVWA (Docker)
- VM4: ONOS SDN Controller
- VM1: Kali Linux attack host
- VM3: Metasploitable2 vulnerable server
- Virtual IoT devices simulated via Mininet

## 3. Components

- ONOS for flow control
- OVS Layer-2/Layer-3 switching
- DVWA and Metasploitable2 as attack targets
- Kali for generating multiple attack types
- Mininet to emulate IoT devices and sensor traffic

## 4. VM Setup Instructions

### VM2 – OVS, Mininet, Docker, DVWA
System updates:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
Install OVS/Mininet:
```
sudo apt-get install openvswitch-switch mininet
sudo ufw disable
```
Docker installation and DVWA:
```
docker run --rm -it -p 8080:80 vulnerables/web-dvwa
```

### VM4 – ONOS Controller
Install Java, dependencies, and ONOS package.  
Enable ONOS service and access GUI at:
```
http://127.0.0.1:8181/onos/ui
```

### VM1 – Kali Linux
Configure IP, generate attacks (Nmap, Hydra, Metasploit, Slowloris, etc.)

### VM3 – Metasploitable2
Default login: `msfadmin / msfadmin`

## 5. OVS Layer-3 Routing

Multi-segment routing between:
- 200.175.2.0/24
- 192.168.3.0/24
- 192.168.8.0/24
- 192.168.20.0/24

Bridges:
```
br1 – external / attack network
br2 – vulnerable server network
s1  – IoT subnet
```

## 6. Mininet IoT Simulation

IoT devices:
- h1–h4
- Each configured with default gateway of 192.168.20.129

Startup:
```
sudo python topology.py
```

## 7. Dataset Generation Flow

ASEADOS-SDN-IoT follows:
1. Benign IoT traffic generation
2. Attack injection (DoS, scans, brute force, exploitation)
3. ONOS OpenFlow telemetry capture
4. `tcpdump` packet capture
5. Labeling by attack class

## 8. Attack Scenarios

ASEADOS covers:
- Reconnaissance
- DoS/DDoS
- Web exploitation (DVWA)
- Brute force
- OS exploitation
- Spoofing/MITM
- IoT-targeted attacks
- Malware-like sessions

## 9. Traffic Capture

Captured via:
- tcpdump on OVS bridges
- host-level packet captures
- ONOS controller logs

## 10. Citation
If using this environment or dataset, cite the ASEADOS-SDN-IoT paper.

