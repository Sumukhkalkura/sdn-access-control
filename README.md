# SDN-Based Access Control System (Project 11)

##  Problem Statement
Implement an SDN controller that allows only authorized hosts to communicate within the network. Unauthorized hosts must be blocked using OpenFlow rules.

---

##  Objective
- Implement access control using Ryu controller
- Use MAC-based filtering (whitelist)
- Dynamically install flow rules
- Demonstrate allowed vs blocked communication

---

##  Technologies Used
- Mininet (Network Simulation)
- Ryu Controller (SDN Controller)
- OpenFlow 1.3
- Python

---

##  Network Topology
- Single switch topology (s1)
- Hosts:
  - h1 → Allowed
  - h2 → Allowed
  - h3 → Blocked

---

##  Controller Logic

### 1. Table-Miss Flow
- Sends unknown packets to controller

### 2. MAC Learning
- Controller learns MAC → port mapping

### 3. Access Control
- Whitelist:
  - 00:00:00:00:00:01 (h1)
  - 00:00:00:00:00:02 (h2)

- Any other host (h3) is blocked

### 4. Flow Rules
- Priority 10 → DROP rules
- Priority 1 → Forwarding rules
- Priority 0 → Table-miss

---

How to Run


Terminal 1 (Controller)
cd ~/ryu
source ~/ryu-env/bin/activate
export PYTHONPATH=$PWD

python3 -m ryu.cmd.manager ~/Desktop/access-controller-project/access_controller.py
Terminal 2 (Mininet)
sudo mn -c
sudo mn --topo single,3 --controller=remote --switch ovsk,protocols=OpenFlow13
Test Cases (Mininet CLI)
Test Case 1: Allowed Communication
h1 ping -c 3 h2

Expected:

0% packet loss
Communication works
Test Case 2: Blocked Communication
h3 ping -c 3 h2

Expected:

100% packet loss
Destination unreachable
Flow Table Verification
sudo ovs-ofctl -O OpenFlow13 dump-flows s1

Expected:

priority=10 → drop rule for blocked MAC
priority=1 → forwarding rules
priority=0 → table-miss (controller)
