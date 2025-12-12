# ASEADOS-SDN-IoT: A Novel SDN–IoT Intrusion Detection Dataset and Testbed Framework

## Dataset will release soon!!!

## Overview
ASEADOS–SDN–IoT is a publicly available and fully documented intrusion detection dataset built from a hybrid SDN–IoT testbed integrating physical IoT devices, virtual nodes, multi-segment routing, and ONOS controller telemetry. It contains **457,044 labeled flows** with **83 statistical features**, capturing both benign traffic and four major attack classes: **DoS, DDoS, Botnet, and Probe**.

## Testbed Architecture
- **Physical IoT devices**: Raspberry Pi boards, Amazon Echo Show, Echo Dot.
- **Virtual IoT nodes**: Flask/Python-based sensors (temperature, humidity, pressure, light, motion).
- **SDN Infrastructure**: ONOS controller + OVS with 5 logical bridges.
- **Attack VMs**: Kali Linux.
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

------------------------------------------------------------------------

# 🖥️ VM2 --- Ubuntu 18.04 (OVS + Mininet + Flask)

### 1. Update Packages

``` bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
```

### 2. Install OVS + Mininet

``` bash
sudo apt-get install -y openvswitch-switch mininet
sudo ufw disable
```

------------------------------------------------------------------------

# 🖥️ VM4 --- ONOS Controller Setup (Ubuntu 18.04)

### 1. Create ONOS User

``` bash
sudo adduser sdn --system --group
```

### 2. Install Dependencies

``` bash
sudo apt install -y git zip curl unzip python2-minimal:i386 openjdk-11-jdk
```

### 3. Install ONOS 2.6.0

``` bash
cd /opt
sudo wget <ONOS-TAR-GZ-LINK>
sudo tar xzf onos-2.6.0.tar.gz
sudo mv onos-2.6.0 onos
sudo chown -R sdn:sdn onos
```

### 4. Configure ONOS Options

Edit `/opt/onos/options`:

``` bash
export ONOS_USER=sdn
export ONOS_APPS=drivers,openflow,gui2
```

### 5. Enable Service

``` bash
sudo systemctl enable onos
sudo systemctl start onos
```

### 6. Access

-   UI: http://127.0.0.1:8181/onos/ui\
-   CLI:

``` bash
ssh -p 8101 onos@localhost
# password: rocks
```

------------------------------------------------------------------------

# 🖥️ VM1 --- Kali Linux (Attack Host)

### Assign IP

``` bash
sudo ifconfig eth1 200.175.2.130/24 up
sudo ip route add default via 200.175.2.129
```

Tools used: - Nmap - Hydra - Slowloris - Hping3 - Metasploit

------------------------------------------------------------------------

# 🖥️ VM3 --- Metasploitable2 (Victim)

### Assign IP

``` bash
sudo ifconfig eth0 192.168.3.130/24 up
sudo ip route add default via 192.168.3.129
```

------------------------------------------------------------------------

# 🔀 OVS & Network Configuration (VM2)

### 1. Network Interface IPs --- `/etc/network/interfaces`

``` bash
auto ens37
iface ens37 inet static
    address 192.168.3.129
    netmask 255.255.255.0

auto ens39
iface ens39 inet static
    address 200.175.2.129
    netmask 255.255.255.0

auto ens40
iface ens40 inet static
    address 192.168.20.129
    netmask 255.255.255.0

auto ens42
iface ens42 inet static
    address 192.168.50.128
    netmask 255.255.255.0

auto ens41
iface ens41 inet static
    address 192.168.40.129
    netmask 255.255.255.0
```

### 2. Configure OVS Bridges

``` bash
sudo ovs-vsctl del-br br1
sudo ovs-vsctl del-br br2

sudo ovs-vsctl add-br br1
sudo ovs-vsctl add-br br2

sudo ovs-vsctl add-port br1 ens39
sudo ovs-vsctl add-port br2 ens37

sudo ifconfig ens37 0
sudo ifconfig ens39 0

sudo ip addr add 200.175.2.129/24 dev br1
sudo ip addr add 192.168.3.129/24 dev br2

sudo ovs-vsctl set-controller br1 tcp:192.168.8.128:6653
sudo ovs-vsctl set-controller br2 tcp:192.168.8.128:6653

sudo ifconfig br1 up
sudo ifconfig br2 up
```

### 3. Switch S1

``` bash
sudo ovs-vsctl del-br s1
sudo ovs-vsctl add-br s1
sudo ovs-vsctl add-port s1 ens40
sudo ifconfig ens40 0
sudo ip addr add 192.168.20.129/24 dev s1
sudo ovs-vsctl set-controller s1 tcp:192.168.8.128:6653
sudo ifconfig s1 up
```

### 4. Switch S2

``` bash
sudo ovs-vsctl del-br s2
sudo ovs-vsctl add-br s2
sudo ovs-vsctl add-port s2 ens42
sudo ifconfig ens42 0
sudo ip addr add 192.168.50.128/24 dev s2
sudo ovs-vsctl set-controller s2 tcp:192.168.8.128:6653
sudo ifconfig s2 up
sudo ip route add 192.168.4.0/24 via 192.168.50.130 dev s2
```

### 5. Switch S3

``` bash
sudo ovs-vsctl del-br s3
sudo ovs-vsctl add-br s3
sudo ovs-vsctl add-port s3 ens41
sudo ifconfig ens41 0
sudo ip addr add 192.168.40.129/24 dev s3
sudo ovs-vsctl set-controller s3 tcp:192.168.8.128:6653
sudo ifconfig s3 up
```

------------------------------------------------------------------------

# 🌐 Mininet Simulation

`topology.py`

``` python
from mininet.net import Mininet
from mininet.node import OVSBridge, Host
from mininet.link import TCLink
from mininet.cli import CLI

net = Mininet(link=TCLink)
s1 = net.addSwitch('s1', cls=OVSBridge)

s1.cmd('ifconfig s1 192.168.20.129/24 up')

hosts = [
    net.addHost('h1', ip='192.168.20.131/24', defaultRoute='via 192.168.20.129'),
    net.addHost('h2', ip='192.168.20.132/24', defaultRoute='via 192.168.20.129'),
    net.addHost('h3', ip='192.168.20.133/24', defaultRoute='via 192.168.20.129'),
    net.addHost('h4', ip='192.168.20.134/24', defaultRoute='via 192.168.20.129')
]

for h in hosts:
    net.addLink(h, s1)

net.start()
net.pingAll()
CLI(net)
net.stop()
```

Run:

``` bash
sudo python topology.py
```

------------------------------------------------------------------------

# 🌐 Virtual IoT Simulation

`viot.py`

``` python
# full script

import random
import time
import requests
import threading
from flask import Flask, request, jsonify
from mininet.net import Mininet
from mininet.node import OVSBridge, Host
from mininet.link import TCLink
from mininet.cli import CLI
from mininet.term import makeTerm

# Initialize Flask server
app = Flask(__name__)

# Store sensor data in memory (in a list or dictionary)
sensor_data_store = []

# Route to handle sensor data submission (POST request)
@app.route('/sensor/data', methods=['POST'])
def sensor_data():
    data = request.get_json()  # Get the JSON data from the request
    if data is None:
        return "Bad Request: expected JSON", 400
    # Optional: add a timestamp
    data['_received_ts'] = time.time()
    print("Received data from sensor {} ({}) : {}".format(data.get('sensor_id', 'unknown'), data.get('sensor_type', 'unknown'), data))
    
    # Store the received data
    sensor_data_store.append(data)

    return jsonify({"status": "ok"}), 200

# Route to fetch all sensor data (GET request)
@app.route('/sensor/data', methods=['GET'])
def get_all_sensor_data():
    """Return all sensor data stored in memory."""
    return jsonify(sensor_data_store), 200

def run_server():
    """Run the Flask server in a separate thread."""
    # Use host 0.0.0.0 to make it reachable from Mininet hosts if required
    app.run(host="0.0.0.0", port=5000, threaded=True)

# IoT Sensor Class to simulate IoT behavior
class IoTSensor:
    def __init__(self, sensor_id, sensor_type, ip, gateway, server_url):
        self.sensor_id = sensor_id
        self.sensor_type = sensor_type
        self.ip = ip
        self.gateway = gateway
        self.server_url = server_url
        self.configure_network()

    def configure_network(self):
        """Configure the sensor's IP address and default route.
        NOTE: In this script we are simulating network configuration; in a
        real deployment you'd configure the network interface on the host."""
        print("Configuring sensor {} with IP: {} and Gateway: {}".format(self.sensor_id, self.ip, self.gateway))
        # Real network configuration commands are purposely omitted here
        pass  # Simulating network setup

    def generate_data(self):
        """Generate random data based on the sensor type."""
        if self.sensor_type == "Temperature":
            return {"sensor_id": self.sensor_id, "sensor_type": self.sensor_type, "temperature": round(random.uniform(20.0, 30.0), 2)}
        elif self.sensor_type == "Humidity":
            return {"sensor_id": self.sensor_id, "sensor_type": self.sensor_type, "humidity": round(random.uniform(40.0, 60.0), 2)}
        elif self.sensor_type == "Motion":
            return {"sensor_id": self.sensor_id, "sensor_type": self.sensor_type, "motion_detected": random.choice([True, False])}
        elif self.sensor_type == "Light":
            return {"sensor_id": self.sensor_id, "sensor_type": self.sensor_type, "light_intensity": round(random.uniform(100.0, 1000.0), 2)}
        elif self.sensor_type == "Pressure":
            return {"sensor_id": self.sensor_id, "sensor_type": self.sensor_type, "pressure": round(random.uniform(950.0, 1050.0), 2)}
        else:
            return {"sensor_id": self.sensor_id, "sensor_type": self.sensor_type, "value": None}

    def send_data(self):
        """Send data to the HTTP server."""
        data = self.generate_data()
        try:
            response = requests.post(self.server_url, json=data, timeout=5)
            if response.status_code == 200:
                print("Sensor {} ({}) successfully sent data: {}".format(self.sensor_id, self.sensor_type, data))
            else:
                print("Sensor {} ({}) failed to send data. Status: {}".format(self.sensor_id, self.sensor_type, response.status_code))
        except Exception as e:
            print("Sensor {} ({}) exception when sending data: {}".format(self.sensor_id, self.sensor_type, e))

    def start_sending_data(self, interval=5):
        """Start periodically sending data every `interval` seconds."""
        while True:
            self.send_data()
            time.sleep(interval)

# Mininet network setup
def create_network():
    net = Mininet(link=TCLink)

    # Create the switch (for simplicity, we use one OVS switch)
    s3 = net.addSwitch('s3', cls=OVSBridge)

    # Assign IPs and gateways for each sensor
    sensor_ips = ['192.168.40.150', '192.168.40.151', '192.168.40.152', '192.168.40.153', '192.168.40.154']
    default_gateway = '192.168.40.129'

    sensors = []
    for i, ip in enumerate(sensor_ips):
        # Note: Mininet host names must start with a letter; use sensor150 -> 's150'
        host_name = 's{}'.format(150 + i)
        sensor = net.addHost(host_name, cls=Host, ip='{}/24'.format(ip), defaultRoute='via {}'.format(default_gateway))
        net.addLink(sensor, s3)
        sensors.append(sensor)

    # Start the network
    net.start()

    # Optionally launch separate xterm windows for each sensor (requires X11)
    for sensor in sensors:
        makeTerm(sensor)  # This will open a new terminal for each sensor

    # Test connectivity between sensors (optional)
    print("Testing connectivity between all sensors...")
    net.pingAll()

    return net, sensors

# Start Mininet and sensors
def start_sensors(sensors, server_url):
    """Start IoT sensors and send data."""
    threads = []
    for i, sensor in enumerate(sensors):
        sensor_ip = sensor.IP()  # Get the IP of the sensor
        sensor_name = sensor.name
        sensor_type = ["Temperature", "Humidity", "Motion", "Light", "Pressure"][i]  # Assign sensor types
        iot_sensor = IoTSensor(sensor_id=150+i, sensor_type=sensor_type, ip=sensor_ip, gateway="192.168.40.129", server_url=server_url)
        
        # Start sending data in a separate thread for each sensor
        tthread = threading.Thread(target=iot_sensor.start_sending_data, args=(5,))
        tthread.setDaemon(True)  # Set the thread as a daemon thread
        tthread.start()
        threads.append(tthread)
    return threads

if __name__ == "__main__":
    # The URL of the server to send data to
    SERVER_URL = "http://127.0.0.1:5000/sensor/data"

    # Start Flask server in a thread
    server_thread = threading.Thread(target=run_server, daemon=True)
    server_thread.start()

    # Create and start the Mininet network
    net, sensors = create_network()

    # Start sensors sending data
    threads = start_sensors(sensors, SERVER_URL)

    # Start CLI for manual interaction
    CLI(net)

    # Stop the network after CLI exit
    net.stop()

```

Run:

``` bash
sudo python viot.py
```

------------------------------------------------------------------------

# 📥 Traffic Capture

Use `tcpdump` on: - ONOS Controller\
- IoT/VIoT nodes\
- Attack/Victim devices

This produces the multiperspective capture used to generate
ASEADOS‑SDN‑IoT dataset.
