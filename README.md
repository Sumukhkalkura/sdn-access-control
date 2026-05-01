# SDN-Based Access Control System (Mininet + Ryu)

---

## Problem Statement

Traditional networks lack the flexibility to enforce fine-grained access control at the data plane level. This project implements an SDN-based access control system using the Ryu controller, where only authorized hosts are permitted to communicate over the network. Unauthorized hosts are identified and blocked dynamically by installing drop rules via OpenFlow, preventing any unauthorized traffic from traversing the switch.

---

## Objective

- Implement access control logic using the Ryu SDN controller
- Allow only whitelisted MAC addresses to communicate through the network
- Block unauthorized hosts dynamically by installing high-priority drop rules
- Demonstrate allowed versus blocked communication behavior in a Mininet environment

---

## Technologies Used

- Mininet
- Ryu Controller
- OpenFlow 1.3
- Python 3

---

## Network Topology

A single switch topology is used with three connected hosts:

- **s1** — OpenFlow-enabled switch (OVS)
- **h1** — Authorized host (whitelisted)
- **h2** — Authorized host (whitelisted)
- **h3** — Unauthorized host (blocked)

Traffic between h1 and h2 is permitted. Any traffic involving h3 is intercepted by the controller and dropped via a flow rule installed on the switch.

---

## How to Run

### Terminal 1 — Start the Ryu Controller

```bash
cd ~/ryu
source ~/ryu-env/bin/activate
export PYTHONPATH=$PWD
python3 -m ryu.cmd.manager ~/Desktop/access-controller-project/access_controller.py
```

Wait until the controller is running and listening before proceeding to Terminal 2.

### Terminal 2 — Start the Mininet Topology

```bash
sudo mn -c
sudo mn --topo single,3 --controller=remote --switch ovsk,protocols=OpenFlow13
```

The `-c` flag cleans up any previous Mininet state. Once the topology is up, use the Mininet CLI to run test cases.

---

## Test Cases

### Test Case 1 — Allowed Communication

Run the following command from the Mininet CLI:

```bash
h1 ping -c 3 h2
```

**Expected Result:**

- 0% packet loss
- All ICMP packets are received successfully
- Communication between whitelisted hosts works as intended

---

### Test Case 2 — Blocked Communication

Run the following command from the Mininet CLI:

```bash
h3 ping -c 3 h2
```

**Expected Result:**

- 100% packet loss
- Destination unreachable or no response
- Controller logs show a BLOCKED event for h3's MAC address

---

## Flow Table Verification

After running the test cases, inspect the flow table on switch s1:

```bash
sudo ovs-ofctl -O OpenFlow13 dump-flows s1
```

**Expected Flow Entries:**

| Priority | Description |
|----------|-------------|
| `priority=10` | Drop rule — matches traffic from unauthorized MAC (h3), action: drop |
| `priority=1` | Forwarding rules — matches whitelisted MACs, action: output to port |
| `priority=0` | Table-miss entry — sends unmatched packets to the controller |

Higher priority rules are evaluated first. The drop rule for h3 is installed at priority 10, ensuring it is matched before any forwarding rule.

---

## Observations

- The Ryu controller inspects each new flow and dynamically installs rules on the switch
- Authorized hosts (h1 and h2) communicate normally with zero packet loss
- Unauthorized hosts (h3) are silently dropped at the switch level after the first packet-in event
- The flow table on s1 accurately reflects the access control policy enforced by the controller
- Controller terminal output displays explicit ALLOWED or BLOCKED log messages per MAC address

---

## Proof of Execution

The following artifacts serve as evidence of successful execution:

- **Ping screenshots** — Terminal output showing 0% loss for h1-h2 and 100% loss for h3
- **Flow table output** — Result of `ovs-ofctl dump-flows s1` showing drop and forwarding entries
- **Controller logs** — Ryu terminal displaying `ALLOWED` for whitelisted MACs and `BLOCKED` for h3's MAC address

---
