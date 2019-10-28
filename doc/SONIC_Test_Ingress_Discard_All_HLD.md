### [DRAFT, UNDER DEVELOPMENT]

- [Overview](#overview)
    - [Scope](#scope)
    - [Related DUT CLI commands](#related-dut-cli-commands)
- [General test flow](#general-test-flow)
- [Run test](#run-test)
- [Test cases](#test-cases)
    - [Test case #1](#test-case-1)
    - [Test case #2](#test-case-2)
    - [Test case #3](#test-case-3)
    - [Test case #4](#test-case-4)
    - [Test case #5](#test-case-5)
    - [Test case #6](#test-case-6)
    - [Test case #7](#test-case-7)
    - [Test case #8](#test-case-8)
    - [Test case #9](#test-case-9)
    - [Test case #10](#test-case-10)
    - [Test case #11](#test-case-11)
    - [Test case #12](#test-case-12)
    - [Test case #13](#test-case-13)
    - [Test case #14](#test-case-14)
    - [Test case #15](#test-case-15)
    - [Test case #16](#test-case-16)

#### Overview
The purpose is to test "RX_DRP" counter got from show command "show interfaces counters" triggers on receiving specific packets.
The "RX_DRP" counter counts all discard events. This counter counts concurrently with other discard counters.
The test assumes all necessary configuration are already pre-configured on the SONIC switch before test runs.
Destination IP address of the injected packet must be routable to ensure packet should not be routed but dropped.

#### Scope
The purpose of the test is testing of "RX_DRP" counter triggering on SONIC system, making sure that specific traffic drops correctly, according to sent packet and configured packet discards.
Supported topologies:
```
t0
t1
t1-lag
ptf32
```

#### Related DUT CLI commands
| **Command**                                                      | **Comment** |
|------------------------------------------------------------------|-------------|
| show interfaces counters              | Check ```RX_DRP```                     |
| counterpoll rif enable                | Enable RIF counters                    |
| show interface counters rif           | Show RIF counters                      |
| aclshow -a                            | Check ```PACKETS COUNT```              |

#### SAI APIs
```SAI_PORT_STAT_IF_IN_DISCARDS``` - to query number of all discards

#### General test flow
Each test case will use the following port types:
- VLAN (T0)
- LAG (T0, T1-LAG)
- Router (T1, T1-LAG, PTF32)

##### Sent packet number:
N - 5

##### Repeat test scenario for all available port types depends on run topology (VLAN, LAG, Router)

- Select two pairs of PTF and DUT ports considering topology: PTF_PORT[1] <--->DUT_PORT[1], PTF_PORT[2] <--->DUT_PORT[2]
- Clear counters. Use CLI command "sonic-clear counters"
- Inject N packet into PTF_PORT[1] (N - depends on test case)
- Check "RX_DRP" counter incremented on N
	- If counter was not incremented on N, test fails with expected message
- Check other counters were incremented on N based on sent packet type, sent port and expected drop reason (depends on test case)
	- If counter was not incremented on N, test fails with expected message
- Check the packet was dropped by sniffing packet absence on PTF_PORT[2]


#### Run test
```
py.test --inventory ../ansible/inventory --host-pattern [DEVICE] --module-path ../ansible/library/ --testbed [DEVICE-TOPO] --testbed_file ../ansible/testbed.csv --junit-xml=./target/test-reports/ --show-capture=no --log-cli-level debug -ra -vvvvv ingress_discard/test_ingress_discard.py
```

#### Test cases
Each test case will be additionally validated by the loganalyzer utility.

Each test case will run specific traffic to trigger specific discard reasone.

Pytest fixture - "ptfadapter" is used to construct and send packets.

After packet is sent using source port index, test framework waits during 5 seconds for specific packet did not appear, due to ingress packet drop, on one of the destination port indices.

#### Test case #1
Test objective

Verify packet drops when SMAC and DMAC are equal

Packet to trigger drop
```
###[ Ethernet ]###
  dst = 00:00:00:00:01:02
  src = 00:00:00:00:01:02
  type = 0x800
...
```

Test steps
- PTF host will send IP packet specifying identical src and dst MAC.
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #2
Test objective

Verify VLAN tagged packet drops when packet VLAN ID does not match ingress port VLAN ID

Packet to trigger drop
```
###[ Ethernet ]###
  dst = [auto]
  src = [auto]
  type = 0x8100
  vid = 2
###[ IP ]###
    version = 4  
    ttl = [auto]
    proto = tcp  
    src = 10.0.0.2
    dst = [get_from_route_info]
...
```

Test steps
- PTF host will send IP packet specifying VID different then port VLAN
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #3
Test objective

Verify packet with multicast SMAC drops

Packet to trigger drop
```
###[ Ethernet ]###
  dst = [auto]
  src = 01:00:5e:00:01:02
  type = 0x800
###[ IP ]###
    version = 4  
    ttl = [auto]
    proto = tcp  
    src = 10.0.0.2
    dst = [get_from_route_info]
...
```

Test steps
- PTF host will send IP packet specifying multicast SMAC.
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #4
Test objective

Verify packet with reserved DMAC drops

Packet1 to trigger drop (use reserved for future standardization MAC address)
```
###[ Ethernet ]###
  dst = 01:80:C2:00:00:05
  src = [auto]
  type = 0x800
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = 10.0.0.2
    dst = [get_from_route_info]
...
```
Packet2 to trigger drop (use provider Bridge group address)
```
###[ Ethernet ]###
  dst = 01:80:C2:00:00:08
  src = [auto]
  type = 0x800
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = 10.0.0.2
    dst = [get_from_route_info]
...
```

Test steps
- PTF host will send IP packet1 specifying reserved DMAC
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send IP packet2 specifying reserved DMAC
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #5
Test objective

Verify packet drops by loop-back filter. Loop-back filter means that route to the host with DST IP of received packet exists on received interface

Packet to trigger drop
```
###[ Ethernet ]###
  dst = [auto]
  src = [auto]
  type = 0x800
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [auto]
    dst = [known_bgp_neighboar_ip]
...
```

Test steps
- PTF host will send IP packet specifying DST IP of VM host. Port to send is port which IP interface is in VM subnet.
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #6
Test objective

Verify packet which exceed router interface MTU (for IP packets) drops
Note: make sure that configured MTU on testbed server and fanout are greater then DUT port MTU

Packet to trigger drop
```
###[ Ethernet ]###
  dst = [auto]
  src = [auto]
  type = 0x800
...
###[ TCP ]###  
    sport = [auto]
    dport = [auto]
    data = [max_mtu + 1]
...
```

Test steps
- PTF host will send IP packet which exceed router interface MTU
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #7
Test objective

Verify packet with TTL expired (ttl <= 0) drops

Packet to trigger drop
```
###[ Ethernet ]###
  dst = [auto]
  src = [auto]
  type = 0x800
...
###[ IP ]###
    version = 4
    ttl = 0
    proto = tcp
    src = [auto]
    dst = [auto]
...
```

Test steps
- PTF host will send IP packet with TTL = 0
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #8
Test objective

Verify non-routable packets discarded at router interface
Packet list:
- IGMP v1 v2 v3 membership query
- IGMP v1 membership report
- IGMP v2 membership report
- IGMP v2 leave group
- IGMP v3 membership report

Test steps
- PTF host will send IGMP v1 v2 v3 membership query
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send IGMP v1 membership report
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send IGMP v2 membership report
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send IGMP v2 leave group
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send IGMP v3 membership report
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #9
Test objective

Verify packet which is not ip (no ip header available) drops

Packet to trigger drop
```
###[ Ethernet ]###
  dst = [auto]
  src = [auto]
  type = 0x800
###[ TCP ]###  
    sport = [auto]
    dport = [auto]
```

Test steps
- PTF host will send packet without IP header
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #10
Test objective

Verify DUT drop packet with broken ip header due to header checksum or IPver or IPv4 IHL too short

Packet1 to trigger drop (Incorrect checksum)
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [auto]
    dst = [auto]
    checksum = [generated_value]
...
```

Packet2 to trigger drop (Incorrect IP version)
```
...
###[ IP ]###
    version = 1
    ttl = [auto]
    proto = tcp
    src = [auto]
    dst = [auto]
...
```

Packet3 to trigger drop (Incorrect IPv4 IHL)
```
...
###[ IP ]###
    version = 4
    ihl = 1
    ttl = [auto]
    proto = tcp
    src = [auto]
    dst = [auto]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet3
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #11
Test objective

Verify DUT drops unicast IP packet sent via:
- multicast DST MAC
- broadcast DST MAC

Packet1 to trigger drop
```
###[ Ethernet ]###
  dst = 01:00:5e:00:01:02
  src = [auto]
  type = 0x800
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = 10.0.0.2
    dst = [get_from_route_info]
...
```

Packet2 to trigger drop
```
###[ Ethernet ]###
  dst = ff:ff:ff:ff:ff:ff
  src = [auto]
  type = 0x800
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = 10.0.0.2
    dst = [get_from_route_info]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #12
Test objective

Verify DUT drops packet where DST IP is loopback address
For ipv4: dip==127.0.0.0/8
For ipv6: dip==::1/128 OR dip==0:0:0:0:0:ffff:7f00:0/104

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [auto]]
    dst = [127.0.0.1]
...
```

Packet2 to trigger drop
```
...
###[ IP ]###
    version = 6
    ttl = [auto]
    proto = tcp
    src = [auto]]
    dst = [::1/128]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #13
Test objective

Verify DUT drops packet where SRC IP is loopback address
For ipv4: dip==127.0.0.0/8
For ipv6: dip==::1/128 OR dip==0:0:0:0:0:ffff:7f00:0/104

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [127.0.0.1]
    dst = [auto]
...
```

Packet2 to trigger drop
```
...
###[ IP ]###
    version = 6
    ttl = [auto]
    proto = tcp
    src = [::1/128]
    dst = [auto]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
 
#### Test case #14
Test objective

Verify DUT drops packet where SRC IP is multicast address
For ipv4: sip = 224.0.0.0/4
For ipv6: sip == FF00::/8

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [224.0.0.5]
    dst = [auto]
...
```

Packet2 to trigger drop
```
...
###[ IP ]###
    version = 6
    ttl = [auto]
    proto = tcp
    src = [ff02::5]
    dst = [auto]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #15
Test objective

Verify DUT drops packet where SRC IP address is in class E
IPv4
AND sip == 240.0.0.0/4
AND sip != 255.255.255.255

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [240.0.0.1]
    dst = [auto]
...
```

Packet2 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [255.255.255.254]
    dst = [auto]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #16
Test objective

Verify DUT drops packet where SRC IP address is not specified

IPv4 sip == 0.0.0.0/32
Note: for IPv6 (sip == ::0)

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [0.0.0.0]
    dst = [auto]
...
```

Packet2 to trigger drop
```
...
###[ IP ]###
    version = 6
    ttl = [auto]
    proto = tcp
    src = [::0]
    dst = [auto]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #17
Test objective

Verify DUT drops packet where DST IP address is not specified

IPv4 sip == 0.0.0.0/32
Note: for IPv6 (sip == ::0)

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [auto]
    dst = [0.0.0.0]
...
```

Packet2 to trigger drop
```
...
###[ IP ]###
    version = 6
    ttl = [auto]
    proto = tcp
    src = [auto]
    dst = [::0]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
---
- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment

#### Test case #18
Test objective

Verify DUT drops packet when configured ACL DROP for SRC IP 20.0.0.0/24

Packet1 to trigger drop
```
...
###[ IP ]###
    version = 4
    ttl = [auto]
    proto = tcp
    src = [20.0.0.10]
    dst = [auto]
...
```

Test steps
- PTF host will send packet1
- When packet reaches SONIC DUT, it should be dropped according to the test objective
- Verify "RX_DRP" counter increment
