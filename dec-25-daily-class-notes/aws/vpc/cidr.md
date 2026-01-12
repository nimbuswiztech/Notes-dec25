# cidr

## CIDR Calculation: A Step-by-Step Classroom Demonstration

CIDR (Classless Inter-Domain Routing) turns IP subnetting into simple arithmetic on bits. Follow the walk-through below to show your students exactly how to slice a `/24` network into four `/26` subnets and verify every key value along the way.

### Overview

CIDR expresses a network as `<base-IP>/<prefix-length>`.

* `prefix-length` = number of **fixed** (network) bits.
* Remaining `32-prefix` bits vary for **hosts**.

Example: `192.168.1.0/24`

* 24 network bits → 8 host bits → 2⁸ = 256 total addresses → 251 usable (5 Ip's will be reserved reserved).

### Demonstration Scenario

* Start with VPC‑style private block `192.168.1.0/24` (common classroom lab).
* Goal: create four equal-size subnets, each large enough for ≈ 60 hosts.
* Solution: borrow **2 more bits** from the host portion.
  * New prefix = 24 + 2 = /26 → host bits = 6 → 2⁶ = 64 total, 62 usable per subnet.

### Binary Logic Cheat-Sheet

| Concept         | Binary Operation            | Result          | Decimal Meaning        |
| --------------- | --------------------------- | --------------- | ---------------------- |
| Network address | IP AND Netmask              | All host bits 0 | Identifies the subnet  |
| Broadcast       | IP OR (\~Netmask)           | All host bits 1 | Last address in subnet |
| Usable range    | Network + 1 → Broadcast – 1 | —               | Assignable to devices  |

### Hands-On Walk-Through

{% stepper %}
{% step %}
### Determine Subnet Mask

* `/26` = first 26 bits are 1s → binary `11111111.11111111.11111111.11000000`
* Convert to dotted-decimal → `255.255.255.192`.
{% endstep %}

{% step %}
### List the New Subnet Blocks

The original `/24` ranges from `192.168.1.0` to `192.168.1.255`.\
Increment size = 64 (2⁶) addresses.

| Subnet # | CIDR             | Network       | First Usable  | Last Usable   | Broadcast     | Total | Usable |
| -------- | ---------------- | ------------- | ------------- | ------------- | ------------- | ----- | ------ |
| 1        | 192.168.1.0/26   | 192.168.1.0   | 192.168.1.1   | 192.168.1.62  | 192.168.1.63  | 64    | 62     |
| 2        | 192.168.1.64/26  | 192.168.1.64  | 192.168.1.65  | 192.168.1.126 | 192.168.1.127 | 64    | 62     |
| 3        | 192.168.1.128/26 | 192.168.1.128 | 192.168.1.129 | 192.168.1.190 | 192.168.1.191 | 64    | 62     |
| 4        | 192.168.1.192/26 | 192.168.1.192 | 192.168.1.193 | 192.168.1.254 | 192.168.1.255 | 64    | 62     |

(The table was generated live with Python using the `ipaddress` module.)
{% endstep %}

{% step %}
### Visual Aid: Bit-Level View of Subnet #1

```
textNetwork bits (26)                 Host bits (6)
11111111.11111111.11111111.11 000000   = 192.168.1.0  (network)
                                 000001   = 192.168.1.1  (first host)
                                 ...
                                 111110   = 192.168.1.62 (last host)
                                 111111   = 192.168.1.63 (broadcast)
```
{% endstep %}
{% endstepper %}

{% hint style="info" %}
#### Teaching Tips

* **Whiteboard the AND operation** using `192.168.1.70` & `255.255.255.192` to show how the result collapses to `192.168.1.64` (proves which subnet the host belongs to).
* **Mnemonic for host count:** `hosts = 2^(32-prefix) – 2` (subtract network & broadcast).
* Use online calculators (e.g., `cidr.xyz`) in class so students can verify their manual work instantly.
* Relate to AWS VPC limits: AWS allows subnet sizes `/28`–`/16`; five IPs per subnet are always reserved.
{% endhint %}

### Practice Exercise for Students

1. Split `10.0.0.0/22` into eight equal subnets.
   * Ask: what is the new prefix? (`/25`)
   * How many usable IPs per subnet? (`126`)
2. Identify the broadcast address of `172.16.8.200/20`.
3. Given `192.168.200.9/30`, prove why it is **not** in the same subnet as `192.168.200.5/30`.

### Key Takeaways

* CIDR offers flexible, non-classful subnet sizing, conserving address space.
* Subnetting is pure binary math: decide hosts required → calculate prefix → enumerate ranges.
* Mastering CIDR is essential for cloud design (AWS, Azure, GCP), on-prem routing, firewalls, and ACLs.

Guide your class through these steps live, and they’ll grasp CIDR calculation with confidence.
