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
The purpose is to test functionality of Asymmetric PFC on the SONIC switch DUT, closely resembling production environment.

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
- Clear all counters for all ports
- Get lossless priorities
- Get lossy priorities
- Get server ports info
- Get non server port info (Portchannel peers)
- Send packets for lossless priorities from all server ports (src) to non-server port (dst)
- Verify that some packets are dropped on dst port, which means that Tx buffer is full
    - 1. Verify that some packets are dropped on src ports, which means that Rx queue is full
    - 2. Verify that PFC frames are generated for lossless priorities
- Send packets for lossy priorities from all server ports (src) to non-server port (dst)
- Verify that PFC frames are not generated for lossy priorities

### Test case # 2 – Asymmetric PFC Off RX pause frames
#### Test objective
Verify that while receiving PFC frames DUT drops packets only for lossless priorities (RX and Tx queue buffers are full)
#### Test steps
- Clear all counters for all ports
- Get lossless priorities
- Get lossy priorities
- Get server ports info
- Get non server port info (Portchannel peers)
- Start PFC generator on fanout switch
- Send packets for lossy priorities from non-server port (src) to all server ports (dst)
- Verify that packets are not dropped on src port
    - 1. Verify that packets are not dropped on dst ports
    - 2. Verify that packets are transmitted from from dst ports
- Send packets for lossless priorities from non-server port (src) to all server ports (dst)
- Stop PFC generator on fanout switch
- Verify that some packets are dropped on src port, which means that Rx queue is full
- Verify that some packets are dropped on dst ports, which means that Tx buffer is full

### Test case # 3 – Asymmetric PFC On Generate PFC frames
#### Test objective
Verify that DUT generates PFC frames only on lossless priorities when asymmetric PFC is enabled
#### Test steps
- Enable asymmetric PFC on all server interfaces

- Set bitmask for the PFC value
- Set bitmask for the asymmetric PFC Tx value
- Set bitmask for the asymmetric PFC Rx value

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
- Verify that some packets are dropped on dst port, which means that Tx buffer is full
    - 1. Verify that some packets are dropped on src ports, which means that Rx queue is full
    - 2. Verify that PFC frames are generated for lossless priorities
- Send packets for lossy priorities from all server ports (src) to non-server port (dst)
- Verify that PFC frames are not generated for lossy priorities

- Disable asymmetric PFC on all server interfaces
- Verify PFC value is restored to default

### Test case # 4 – Asymmetric PFC "On" handle PFC frames on all priorities
#### Test objective
Verify that while receiving PFC frames DUT handle PFC frames on all priorities when asymetric mode is enabled
#### Test steps
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

## TODO

## Open questions
