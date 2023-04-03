
# Egress SFlow Test Plan

## Overview





The purpose is to test the functionality of the sFlow monitoring system in both Ingress and Egress direction on the SONIC switch DUT . The test assumes that the Sonic device has been preconfigured according to the t0 topology.

### Scope

The test is targeting a running SONIC system with fully functioning configuration. Rather, it is testing the functionality of sFlow on the SONIC system by verifying that sFlow is monitoring traffic flow data correctly.

## Test Structure

### Setup Configuration

The test will run on the t0 testbed:

![testbed-t0.png](https://github.com/sonic-net/sonic-mgmt/blob/master/docs/testbed/img/testbed-t0.png?raw=true)

- 1 port from each port channel is used for traffic. Sflow is enabled on these 4 ports in both the directions.
- 2 ports from Vlan1000 are removed and used to reach sflow collector in ptf docker .
- Traffic is sent from ptf docker with different packet sizes to the port channel interfaces  which are enabled with sflow .
- Collection is implemented using the sflowtool. Counter sampling output and flow sampling output are directed to a text file. The test script parses the text file and validates the data according to the polling/sampling rate configured and the interfaces enabled.
- Ingress and Egress samples will be identified using integer attribute PSAMPLE_ATTR_SAMPLE_GROUP - expects ingress samples to appear with group=1, and egress samples to appear with group=2 
- Sflowtool to be installed in ptf docker to run these tests.

### Test Cases

#### Test Case #1

##### Test objective

Verify that the egress-sampling on the port connected to leaf.

| #    | Steps                                                        | Expected Result                                              |
| ---- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1.   | 1. Enable egress sflow on leaf port <br />2. Add a single collector with default port (6343), <br />3. Enable sFlow globally,<br />4. Send traffic | The configurations should be reflected in “show sflow” and "show sflow interface". <br />Traffic is sent to portchannel interfaces and Samples should be received by the collector . |
| 2.   | 1. Enable ingress sflow on server port <br />2. Send traffic | The configurations should be reflected in “show sflow” and "show sflow interface".<br />Verify collector receives both ingress and egress samples. |
| 3.   | 1. Disable egress sflow globally<br />2. Send traffic | The configurations should be reflected in “show sflow” and "show sflow interface". only Ingress samples should be received by both collector.|
| 4.   | 1. Enable back the egress-sflow <br />2. Send traffic | Verify collector receives both ingress and egress samples |

#### Test Case #2

##### Test objective

Verify that the ingress and egress sflow on the same port.

| #    | Steps                                                        | Expected Result                                              |
| ---- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1.   | 1. Enable ingress and egress sflow on server port1 <br />2.Enable sFlow globally,<br />3. Send bi-directional traffic from server port1 and port2| Verify collector receives both ingress and egress samples.|
| 2.   | 1. Disable ingress sflow on server port1 <br />2. Send traffic | The configurations should be reflected in “show sflow” and "show sflow interface".<br />Verify collector receives only egress samples. |
| 3.   | 1. Enable ingress and Disable egress sflow globally<br />2. Send traffic | The configurations should be reflected in “show sflow” and "show sflow interface". only Ingress samples should be received by collector.|
| 4.   | 1. Enable back the egress-sflow <br />2. Send traffic | Verify collector receives both ingress and egress samples |

#### Test Case #3

##### Test objective

Verify that the ingress and egress sflow with different sample-rates on the different port.

| #    | Steps                                                        | Expected Result                                              |
| ---- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1.   | 1. Enable egress sflow on leaf port <br />2. sample-rate = 1024 <br />2.Enable ingress sflow on server port with sample-rate = 256 <br />3. Enable sFlow globally,<br />4. Send traffic | Verify collector receives both ingress and egress samples .|
| 2.   | 1. Update the sample-rate to 256 for egress-sflow and 1024 for ingress sflow <br />2. Send traffic | Verify collector receives both ingress and egress samples. |
| 3.   | 1. Update sample-rate to 8388608 for ingress sflow <br />2. Send traffic | Verify collector receives both ingress and egress samples as per the configured sample-rate.|
| 4.   | 1. Update sample-rate to 8388608 for egress sflow <br />2. Send traffic | Verify collector receives both ingress and egress samples as per the configured sample-rate.|


#### Test Case #4

##### Test objective

Verify that sflow configs at interface-level has higher precedence then global configuration.

| #    | Steps                                                        | Expected Result                                              |
| ---- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1.   | 1. Enable sflow globally with default direction i.e rx <br />| The configurations should be reflected in “show sflow” and "show sflow interface.|
| 2.   | 1. Update the sample-direction to 'tx' at interface level for leaf ports and sample-rate to 512 and Disable sflow on ingress server ports<br />2. Send traffic| The configurations should be reflected in “show sflow” and "show sflow interface. | Verify collector receives only egress samples.|
| 3.   | 1. Disable sflow on all leaf ports at interface level <br />2. Send traffic | Verify no samples are received at collector.|
| 4.   | 1. Enable sflow on server port1 and port2 at interface level <br />2.Update the direction to both at the interface level for server port1 and port2 <br />3. Send bi-directional traffic | Verify both ingress and egress samples are received at the collector.|


#### Test Case #5

##### Test objective

Verify that sflow sampling works when enabled at interface level and disable at global level.

| #    | Steps                                                        | Expected Result                                              |
| ---- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1.   | 1. Disable sflow globally.| Verify sflow admin-state is Down on all ports.The configurations should be reflected in “show sflow” and "show sflow interface.|
| 2.   | 1. Enable egress sflow on leaf ports at interface level.| Verify that sflow admin-state is UP on the leaf ports.The configurations should be reflected in “show sflow” and "show sflow interface.|
| 3.   | 1. Send traffic from server port to leaf ports.| Verify the collector is receives egress samples.|
| 4.   | 1. Enable ingress sflow on server ports at interface level.| Verify that sflow admin-state is UP on the server ports.The configurations should be reflected in “show sflow” and "show sflow interface.|
| 5.   | 1. Send traffic from server port to leaf ports.| Verify the collector is receives both ingress and egress samples.|


#### Test Case #6

##### Test objective

Verify that sflow admin-state.

| #    | Steps                                                        | Expected Result                                              |
| ---- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1.   | 1. Enable sflow globally. "config sflow enable" and direction to both.| The configurations should be reflected in “show sflow” and "show sflow interface.|
| 2.   | 1. Send traffic from server port to leaf ports.| Verify the collector is receives ingress and egress samples.|
| 3.   | 1. Now update the admin-state in sflow to "Down", use sonic-cfgjen utility to apply the config| The configurations should be reflected in “show sflow” and "show sflow interface.|
| 4.   | 1. Send traffic from server port to leaf ports.| Verify that samples are not received at the collector.|
| 5.   | 1. Now update the admin-state in sflow to "Up", use sonic-cfgjen utility to apply the config| The configurations should be reflected in “show sflow” and "show sflow interface.|
| 6.   | 1. Send traffic from server port to leaf ports.| Verify that samples are received at the collector.|
