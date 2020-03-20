# CRM test plan

* [Overview](#Overview)
   * [Scope](#Scope)
   * [Testbed](#Testbed)
* [Setup configuration](#Setup-configuration)
   * [Python modules to setup and run test](#Python-modules-to-setup-and-run-test)
      * [Pytest fixtures](#Pytest-fixtures)
* [Common test case steps](#Common-test-case-steps)
* [Test](#Test)
* [Test cases](#Test-cases)
  - IPv4 route
  - IPv6 route
  - IPv4 nexthop
  - IPv6 nexthop
  - IPv4 neighbor
  - IPv6 neighbor
  - Nexthop group object
  - Nexthop group member
  - FDB entry
  - ACL group
  - ACL table
  - ACL entry
  - ACL counter
  - CRM counters with applied VNET config

## Overview
The purpose is to test functionality of CRM on the SONIC switch DUT, closely resembling production environment.

### Scope
The test is targeting a running SONIC system with fully functioning configuration. The purpose of the test is not to test specific API, but functional testing of CRM on SONIC system.

### Testbed
The test will run on the all testbeds.

## Setup configuration
No setup pre-configuration is required, test will configure and clean-up all the configuration.

### Python modules to setup and run test

```tests/crm/test_crm.py``` - test suite

```test/dut_configs/acl.json``` - ACL configuration for ACL test cases

```tests/dut_configs/fdb.json``` - FDB configuration for FDB test case


### Pytest fixtures
- crm_interface
- adjust_polling_interval

#### crm_interface(scope="module", autouse=True)
Get DUT interfaces used for CRM testing

#### adjust_polling_interval(scope="module", autouse=True)
On setup:
- get current polling interval
- configure CRM polling interval to 1 second

On teardown: 
- configure CRM polling interval to previous value

## Common test case steps
The following steps will be executed for each of test case:
1. Apply required configuration.
2. Verify "used" and "free" counters.
3. Verify "EXCEEDED" and "CLEAR" messages appeared in syslog for all types of thresholds: used, free, percentage.
4. Restore configuration.

## Test

## Test cases

### Test case # 1 – IPv4 route
#### Test objective
Verify "IPv4 route" CRM resource.
#### Test steps
* Configure 1 route and observe that counters were updated as expected.
* Remove 1 route and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 2 – IPv6 route
#### Test objective
Verify "IPv6 route" CRM resource.
#### Test steps
* Configure 1 route and observe that counters were updated as expected.
* Remove 1 route and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 3 – IPv4 nexthop
#### Test objective
Verify "IPv4 nexthop" CRM resource.
#### Test steps
* Add 1 nexthop and observe that counters were updated as expected.
* Remove 1 nexthop and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 4 – IPv6 nexthop
#### Test objective
Verify "IPv6 nexthop" CRM resource.
#### Test steps
* Add 1 nexthop and observe that counters were updated as expected.
* Remove 1 nexthop and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 5 – IPv4 neighbor
#### Test objective
Verify "IPv4 neighbor" CRM resource.
#### Test steps
* Configure 1 neighbor and observe that counters were updated as expected.
* Remove 1 neighbor and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 6 – IPv6 neighbor
#### Test objective
Verify "IPv6 neighbor" CRM resource.
#### Test steps
* Configure 1 neighbor and observe that counters were updated as expected.
* Remove 1 neighbor and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 7 – Nexthop group object
#### Test objective
Verify "nexthop group object" CRM resource.
#### Test steps
* Configure 1 ECMP route and observe that counters were updated as expected.
* Remove 1 ECMP route and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 8 – Nexthop group member
#### Test objective
Verify "nexthop group member" CRM resource.
#### Test steps
* Configure 1 ECMP route and observe that counters were updated as expected.
* Remove 1 ECMP route and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 9 – FDB entry
#### Test objective
Verify "FDB entry" CRM resource.
#### Test steps
* Configure 1 FDB entry and observe that counters were updated as expected.
* Remove 1 FDB entry and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 10 – ACL group
#### Test objective
Verify "ACL group" CRM resource.
#### Test steps
* Configure 1 ACL and observe that counters were updated as expected.
* Remove 1 ACL and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 11 – ACL table
#### Test objective
Verify "ACL table" CRM resource.
#### Test steps
* Configure 1 ACL and observe that counters were updated as expected.
* Remove 1 ACL and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 12 – ACL entry
#### Test objective
Verify "ACL entry" CRM resource.
#### Test steps
* Configure 1 ACL rule and observe that counters were updated as expected.
* Remove 1 ACL rule and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default CRM thresholds configuration.

### Test case # 13 – ACL counter
#### Test objective
Verify "ACL entry" CRM resource.
#### Test steps
* Configure 1 ACL rule and observe that counters were updated as expected.
* Remove 1 ACL rule and observe that counters were updated as expected.
* Perform the following steps for all threshold types ("percentage", "used", "free"):
	* Set low and high thresholds according to current usage and type.
	* Verify that "EXCEEDED" message is logged (using log analyzer).
	* Set low and high thresholds to default values.
	* Verify that "CLEAR" message is logged (using log analyzer).
* Restore default configuration.

### Test case # 14 – CRM counters with applied VNET config
#### Test objective
Verify "IPv4 route, IPv4 nexthop, IPv4 neighbor, FDB entry" CRM resources after applying VNET configuration.

To apply VNET configuration run ```roles/test/tasks/vnet_vxlan.yml``` test case.

#### Test steps
* Apply VNET configuration and observe that counters were updated as expected.
* Clean VNET config and observe that counters were updated as expected.
