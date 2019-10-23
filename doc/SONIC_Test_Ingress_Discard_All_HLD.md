### [DRAFT, UNDER DEVELOPMENT]

#### Overview
The purpose is to test "ingress_discard_all" counter triggers on receiving specific packets.
The "ingress_discard_all" counter counts all discard events. This counter counts concurrently with other discard counters.
The test assumes all necessary configuration are already pre-configured on the SONIC switch before test runs.
Destination IP address of the injected packet must be routable to ensure packet was dropped.

#### Scope
The purpose of the test is testing of "ingress_discard_all" counter triggering on SONIC system, making sure that specific traffic drops correctly, according to sent packet and configured packet discards.
Supported topologies:
t0
t1
t1-lag
ptf32

#### Related DUT CLI commands
Command to check "ingress_discard_all" counter value:
```show interfaces counters```

Check field:
```RX_DRP```

#### General test flow
Inject packet into tor port. Set specifc MAC or IP addresses to BGP route learned on spine ports. Check "ingress_discard_all" counter incremented. Check the packet was dropped on spine port.

#### Run test
py.test --inventory ../ansible/inventory --host-pattern [DEVICE] --module-path ../ansible/library/ --testbed [DEVICE-TOPO] --testbed_file ../ansible/testbed.csv --junit-xml=./target/test-reports/ --show-capture=no --log-cli-level debug -ra -vvvvv ingress_discard/test_ingress_discard.py

#### Test cases
Each test case will be additionally validated by the loganalyzer utility.
Each test case will run specific traffic to trigger specific discard reasone.
Pytest fixture - "ptfadapter" is used to construct and send packets.
After packet is sent using source port index, test framework waits for specific packet did not appear, due to ingress packet drop, on one of the destination port indices.

#### Test case #1
Test objective
Verify packet drops when SMAC and DMAC are equal.

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SMAC_EQUALS_DMAC"
- Verify "ingress_discard_all" counter increment

#### Test case #2
Test objective
Verify packet drops when packet VLAN ID does not match ingress port VLAN ID

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_VLAN_TAG_NOT_ALLOWED"
- Verify "ingress_discard_all" counter increment

#### Test case #3
Test objective
Verify packet with SMAC Multicast drops

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SMAC_MULTICAST"
- Verify "ingress_discard_all" counter increment

#### Test case #4
Test objective
Verify packet with reserved DMAC drops

Packet to trigger drop
```
###[ Ethernet ]###
  dst = 01:80:c2:00:00:09
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
- PTF host will send IP packet specifying reserved DMAC
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_DMAC_RESERVED"
- Verify "ingress_discard_all" counter increment

#### Test case #5
Test objective
Verify packet drops by loop-back filter

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_L2_LOOPBACK_FILTER"
- Verify "ingress_discard_all" counter increment

#### Test case #6
Test objective
Verify packet which exceed router interface MTU (for IP and/or MPLS packets) drops

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_EXCEEDS_L3_MTU"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_TTL"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_NON_ROUTABLE"
- Verify "ingress_discard_all" counter increment

- PTF host will send IGMP v1 membership report
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_NON_ROUTABLE"
- Verify "ingress_discard_all" counter increment

- PTF host will send IGMP v2 membership report
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_NON_ROUTABLE"
- Verify "ingress_discard_all" counter increment

- PTF host will send IGMP v2 leave group
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_NON_ROUTABLE"
- Verify "ingress_discard_all" counter increment

- PTF host will send IGMP v3 membership report
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_NON_ROUTABLE"
- Verify "ingress_discard_all" counter increment

#### Test case #9
Test objective
Verify packet which is not ip/mpls (no ip header) drops

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_NO_L3_HEADER"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_IP_HEADER_ERROR"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_IP_HEADER_ERROR"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet3
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_IP_HEADER_ERROR"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_UC_DIP_MC_DMAC"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_DIP_LOOPBACK"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_DIP_LOOPBACK"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_LOOPBACK"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_LOOPBACK"
- Verify "ingress_discard_all" counter increment
 
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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_MC"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_MC"
- Verify "ingress_discard_all" counter increment

#### Test case #15
Test objective
Verify DUT drops packet where SRC IP address is in class e
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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_CLASS_E"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_CLASS_E"
- Verify "ingress_discard_all" counter increment

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
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_UNSPECIFIED"
- Verify "ingress_discard_all" counter increment

- PTF host will send packet2
- When packet reaches SONIC DUT, it should be dropped by "SAI_IN_DROP_REASON_SIP_UNSPECIFIED"
- Verify "ingress_discard_all" counter increment

