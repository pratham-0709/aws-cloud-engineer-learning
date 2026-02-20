# NAT Instance Setup Guide — Ubuntu 22.04 on AWS

---

## Step 1: Launch an Ubuntu 22.04 EC2 Instance (Public Subnet)

| Setting | Value |
|---|---|
| **AMI** | Ubuntu 22.04 LTS (official Canonical AMI) |
| **Instance Type** | `t3.micro` / `t3a.micro` (sufficient for light NAT traffic) |
| **Subnet** | Place in a **public subnet** |
| **Auto-assign Public IP** | Enable (or attach an Elastic IP) |

---

## Step 2: Disable Source/Destination Check ⚠️ Critical!

> By default, EC2 drops traffic not destined for its own IP. This step disables that behavior.

### Via AWS Console
```
EC2 → Instances → Select NAT Instance → Actions → Networking → Change source/destination check → ✅ Stop
```

### Via AWS CLI
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-0xxxxxxxxxxxx \
  --no-source-dest-check
```

---

## Step 3: Security Group for NAT Instance

### Inbound Rules

| Type | Port | Source |
|---|---|---|
| All Traffic | All | Private Subnet CIDR (e.g., `10.0.2.0/24`) |
| SSH | 22 | Your IP (for management) |

### Outbound Rules

| Type | Port | Destination |
|---|---|---|
| All Traffic | All | `0.0.0.0/0` |

---

## Step 4: Configure Ubuntu 22.04 as a NAT Instance

### 4a. Enable IP Forwarding

SSH into the instance and run:

```bash
sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-nat.conf
sysctl -p /etc/sysctl.d/99-nat.conf
cat /proc/sys/net/ipv4/ip_forward
```

> ✅ Output should show: `1`

---

### 4b. Configure iptables NAT (Masquerade)

First, find your primary network interface:

```bash
ip route show default
```

Then set up the MASQUERADE rule:

```bash
iptables -t nat -F POSTROUTING 2>/dev/null || true
iptables -F FORWARD 2>/dev/null || true
iptables -t nat -A POSTROUTING -o ens5 -s 10.0.0.0/16 -j MASQUERADE
iptables -A FORWARD -i ens5 -o ens5 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens5 -s 10.0.0.0/16 -j ACCEPT
```

---

### 4c. Make iptables Rules Persistent

```bash
DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y iptables-persistent
netfilter-persistent save
```

**Expected output:**
```
run-parts: executing /usr/share/netfilter-persistent/plugins.d/15-ip4tables save
run-parts: executing /usr/share/netfilter-persistent/plugins.d/25-ip6tables save
```

---

## Verification Checks

### Check 1 — NAT Table

```bash
sudo iptables -t nat -L -v -n
```

**Expected output:**
```
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   44  3458 MASQUERADE  all  --  *      ens5    10.0.0.0/16          0.0.0.0/0
```

---

### Check 2 — FORWARD Chain

```bash
sudo iptables -L FORWARD -v -n
```

**Expected output:**
```
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT      all  --  ens5   ens5    0.0.0.0/0            0.0.0.0/0   state RELATED,ESTABLISHED
    0     0 ACCEPT      all  --  ens5   *       10.0.0.0/16          0.0.0.0/0
```

---

### Check 3 — POSTROUTING with Line Numbers

```bash
iptables -t nat -L POSTROUTING -v -n --line-numbers
```

**Expected output:**
```
Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1       55  4294 MASQUERADE  all  --  *      ens5    10.0.0.0/16          0.0.0.0/0
```