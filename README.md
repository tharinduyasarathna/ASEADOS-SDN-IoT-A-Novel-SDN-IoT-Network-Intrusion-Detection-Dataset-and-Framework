
# ASEADOS-SDN-IoT: A Novel SDN–IoT Intrusion Detection Dataset and Testbed Framework

## Overview
ASEADOS–SDN–IoT is a publicly available, fully documented intrusion detection dataset built from a hybrid SDN–IoT testbed that integrates physical IoT devices, virtual nodes, multi-segment routing, and ONOS controller telemetry.  
It contains **457,044 labelled flows** with **83 statistical features**, capturing both benign traffic and four major attack classes: **DoS, DDoS, Botnet, and Probe**.

## Key Features
- Hybrid SDN–IoT testbed with physical Raspberry Pi devices and virtual IoT sensors.
- Integrated ONOS telemetry (packet-in, flow-mod, topology updates).
- Time-synchronised PCAP captures across SDN controller, IoT devices, and VMs.
- Flow-based dataset generated using CICFlowMeter.
- Fully labelled multi-class dataset supporting ML/DL intrusion detection.

## Testbed Architecture
- **Physical IoT devices**: Raspberry Pi boards, Amazon Echo Show, and Echo Dot.
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

## Feature Set
Flows were extracted using CICFlowMeter, producing **83 numerical features**, including:
- Packet/byte statistics  
- Inter-arrival times  
- TCP flag counts  
- Active/idle times  
- Throughput metrics  

After cleaning, **64 core features** remain for modeling.

## Data Preprocessing Pipeline
- Removal of duplicates, incomplete flows, zero-duration sessions.
- Feature refinement to remove low-variance/multicollinearity attributes.
- Outlier clipping on durations, counts, throughput.
- Min–Max normalization.
- Label encoding:  
  - `0 = Benign`, `1 = DoS`, `2 = DDoS`, `3 = Botnet`, `4 = Probe`
- 80/20 training-testing split (stratified).

## Attack Scenarios
### DoS
High-rate floods targeting IoT nodes, SDN switches, or services.

### DDoS
Distributed floods involving multiple nodes with synchronized multi-protocol attacks.

### Botnet
BoNeSI-driven beaconing + surge attacks to simulate real C2 behavior.

### Probe/Exploitation
Nmap scans + Metasploit exploitation lifecycle traffic.

## Research Applications
- Machine Learning & Deep Learning IDS evaluation
- Cross-plane correlation of ONOS telemetry with IoT traffic
- Time-series modeling
- SDN controller stress testing
- Behavioral profiling of IoT ecosystems

## Limitations
- Single-controller (ONOS) environment.
- Four attack categories (future expansion expected).
- No application-layer payload features (privacy-focused).

## Future Work
- Multi-controller SDN architectures.
- Expanded IoT diversity and larger topologies.
- Inclusion of encrypted traffic fingerprints.
- Emerging AI-driven, multi-stage attacks.
- Explainable and graph-based IDS frameworks.

---

## Citation
If you use ASEADOS–SDN–IoT, please cite the corresponding publication.

---

## Dataset & Code Access
Dataset, scripts, and instructions are available in this repository. Additional PCAPs and ONOS logs may be provided upon request.

