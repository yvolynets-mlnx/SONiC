# Draft
# Asymmetric PFC Test Plan

* [Overview](#Overview)
   * [Scope](#Scope)
   * [Testbed](#Testbed)
* [Setup configuration](#Setup%20configuration)
* [Python  modules to setup and run test](#Python%20modules%20to%20setup%20and%20run%20test)
* [Test](#Test)
* [Test cases](#Test%20cases)
* [TODO](#TODO)
* [Open questions](#Open%20questions)

## Overview
The purpose is to test functionality of Asymmetric PFC on the SONIC based DUT, closely resembling production environment.

### Scope
The test is targeting a running SONIC system with fully functioning configuration. The purpose of the test is to test functional testing of Asymmetric PFC on SONIC system.

### Testbed
The test will run on the following testbeds:
* T0

## Setup configuration
No setup pre-configuration is required, test will configure and clean-up all the configuration.


## Python  modules to setup and run test (TODO)

## Test

## Test cases

### Test case # 1 – Asymmetric PFC Off Generate PFC frames
#### Test objective
Verify that DUT generates PFC frames only on lossless priorities when asymmetric PFC is disabled
#### Test steps
- Setup:
  - Start ARP responder
  - Limit maximum bandwith rate on the destination port by setting "1" into SAI port attribute SAI_PORT_ATTR_QOS_SCHEDULER_PROFILE_ID
  
- Clear all counters for all ports
- Get lossless priorities
- Get lossy priorities
- Get server ports info
- Get non server port info (Portchannel peers)
- Send packets for lossless priorities from all server ports (src) to non-server port (dst)
- Verify that some packets are dropped on src ports, which means that Rx queue is full
- Verify that PFC frames are generated for lossless priorities from src ports
- Send packets for lossy priorities from all server ports (src) to non-server port (dst)
- Verify that PFC frames are not generated for lossy priorities

- Teardown:
  - Restore maximum bandwith rate on the destination port by setting "0" into SAI port attribute SAI_PORT_ATTR_QOS_SCHEDULER_PROFILE_ID
   - Stop ARP responder

### Test case # 2 – Asymmetric PFC Off RX pause frames
#### Test objective
Verify that while receiving PFC frames DUT drops packets only for lossless priorities (RX and Tx queue buffers are full)
#### Test steps
- Setup:
  - Start ARP responder

- Clear all counters for all ports
- Get lossless priorities
- Get lossy priorities
- Get server ports info
- Get non server port info (Portchannel peers)
- Start PFC generator on fanout switch
- Send packets for lossy priorities from non-server port (src) to all server ports (dst)
- Verify that packets are not dropped on src port
- Verify that packets are not dropped on dst ports
- Verify that packets are transmitted from from dst ports
- Send packets for lossless priorities from non-server port (src) to all server ports (dst)
- Stop PFC generator on fanout switch
- Verify that some packets are dropped on src port, which means that Rx queue is full
- Verify that some packets are dropped on dst ports, which means that Tx buffer is full

- Teardown:
  - Stop ARP responder

### Test case # 3 – Asymmetric PFC On Generate PFC frames
#### Test objective
Verify that DUT generates PFC frames only on lossless priorities when asymmetric PFC is enabled
#### Test steps
- Setup:
  - Start ARP responder
  - Limit maximum bandwith rate on the destination port by setting "1" into SAI port attribute SAI_PORT_ATTR_QOS_SCHEDULER_PROFILE_ID

- Enable asymmetric PFC on all server interfaces
- As SAI attributes stores PFC values like a bit vector, calculate bitmask for each PFC mode according to configured "lossless" priorities:
  - Calculate bitmask for the PFC value
  - Calculate bitmask for the asymmetric PFC Tx value
  - Calculate bitmask for the asymmetric PFC Rx value

- Verify asymetric PFC mode is enabled on each server port
- Get asymmetric PFC Rx value for all server ports
- Verify asymmetric PFC Rx value for each server port
- Get asymmetric PFC Tx value for all server ports
- Verify asymmetric PFC Tx value for all server ports

- Clear all counters for all ports
- Get lossless priorities
- Get lossy priorities
- Get server ports info
- Get non server port info (Portchannel peers)

- Send packets for lossless priorities from all server ports (src) to non-server port (dst)
- Verify that some packets are dropped on src ports, which means that Rx queue is full
- Verify that PFC frames are generated for lossless priorities
- Send packets for lossy priorities from all server ports (src) to non-server port (dst)
- Verify that PFC frames are not generated for lossy priorities

- Disable asymmetric PFC on all server interfaces
- Verify PFC value is restored to default

- Teardown:
  - Restore maximum bandwith rate on the destination port by setting "0" into SAI port attribute SAI_PORT_ATTR_QOS_SCHEDULER_PROFILE_ID
   - Stop ARP responder

### Test case # 4 – Asymmetric PFC "On" handle PFC frames on all priorities
#### Test objective
Verify that while receiving PFC frames DUT handle PFC frames on all priorities when asymetric mode is enabled
#### Test steps
- Setup:
  - Start ARP responder

- Enable asymmetric PFC on all server interfaces

- Clear all counters for all ports
- Get lossless priorities
- Get lossy priorities
- Get server ports info
- Get non server port info (Portchannel peers)
- Start PFC generator on fanout switch
- Send packets for lossy priorities from non-server port (src) to all server ports (dst)
- Verify that packets are not dropped on src port
- Verify that some packets are dropped on dst ports, which means that Tx buffer is full
- Send packets for lossless priorities from non-server port (src) to all server ports (dst)
- Verify that some packets are dropped on src port, which means that Rx queue is full
- Verify that some packets are dropped on dst ports, which means that Tx buffer is full

- Stop PFC generator on fanout switch
- Disable asymmetric PFC on all server interfaces
- Verify PFC value is restored to default

- Teardown:
  - Stop ARP responder


## TODO

## Open questions
