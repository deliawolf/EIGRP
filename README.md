# EIGRP

EIGRP (Enhanced Interior Gateway Routing Protocol) is an enhanced distance vector protocol that uses Diffusing Update Algorithm (DUAL) for shortest path calculation. 

## Neighbor adjacency

The following parameters should match in order for EIGRP to create a neighbor adjacency:

1. AS number: An EIGRP router sends hello packets out all EIGRP-enabled interfaces. The EIGRP multicast address is 224.0.0.10. An EIGRP router only establishes neighbor relationships (adjacencies) with other routers within the same autonomous system. An EIGRP autonomous system number is a unique number established by an enterprise. It is used to identify a group of devices and enables that system to exchange interior routing information with other neighboring routers with the same autonomous systems.

2. K values (metric): EIGRP K values are the metrics that EIGRP uses to calculate routes. Mismatched K values can prevent neighbor relationships from being established and can negatively impact network convergence.

3. Common subnet: EIGRP cannot form neighbor relationships using secondary addresses, as only primary addresses are used as the source IP addresses of all EIGRP packets

4. Authentication method and password: Regarding authentication, EIGRP will neighbor with any router that sends a valid hello packet. Due to security considerations, this "completely" open aspect requires policy capabilities to limit peering to valid routers. The security implications of a threat actor manipulating the routing tables are vast. Even though routing protocol authentication is not covered in this course, it is highly recommended that this feature is enabled.


## EIGRP vector metrics:

1. Bandwidth (K1): The smallest bandwidth of all outgoing interfaces between the source and destination, in kilobits.

2. Load (K2): This value represents the worst load on a link between the source and destination, which is computed based on the packet rate and the configured bandwidth of the interface.

3. Delay (K3): The cumulative (sum) of all interface delay along the path, in tens of microseconds.

4. Reliability (K4): The likelihood of successful packet transmission, expressed as a number between 0 and 255, where 255 means 100 percent reliability and 0 means no reliability.

5. MTU (K5): The minimum-maximum MTU size of the route in bytes.

## Path Selection

EIGRP uses two parameters to determine the best route (successor) and any backup routes (feasible successors) to a destination:

1. Advertised Distance (AD): The EIGRP metric for an EIGRP neighbor to reach a destination network, also referred to as reported distance.

2. Feasible Distance (FD): The EIGRP metric for a local router to reach a destination network. In other words, it is the sum of the advertised distance of an EIGRP neighbor and the metric to reach the neighbor. This sum provides an end-to-end metric from the router to the remote network.

## Load Balancing

## Equal-Cost Load Balancing
```
HQ(config)# router eigrp 100
HQ(config-router)# maximum-paths ?
  <1-16>  Number of paths
```
## Unequal-Cost Load Balancing
```
HQ(config-router)# router eigrp 100
HQ(config-router)# variance ?
  <1-128>  Metric variance multiplier 

HQ(config-router)# variance 4
HQ(config-router)#
```

## EIGRP Config
There only 1 mandatory step for it
1. Configure routes 
2. Configure authentication ( additional )

i am using below topology for EIGRP. the topology have 2 servers and 5 routers.
![Creating a VLAN](https://raw.githubusercontent.com/deliawolf/EIGRP/main/Untitled%20Diagram.drawio.png)

### Configure ROUTES

when configuring routes make sure using same ASN and only put network neighboar.
note* : here i already configured the ip for each interface.

ROUTER 1
```
router eigrp 100
 network 10.10.11.0 0.0.0.255
 network 10.10.13.0 0.0.0.255
 network 192.168.2.0
```
ROUTER 2
```
router eigrp 100
 network 10.10.13.0 0.0.0.255
 network 10.10.14.0 0.0.0.255
```
ROUTER 3
```
router eigrp 100
 network 10.10.12.0 0.0.0.255
 network 10.10.14.0 0.0.0.255
 network 192.168.3.0
 auto-summary
```
ROUTER 4
```
router eigrp 100
 network 10.10.11.0 0.0.0.255
 network 10.10.12.0 0.0.0.255
 network 10.10.15.0 0.0.0.255
```
ROUTER 5
```
router eigrp 100
 network 10.10.15.0 0.0.0.255
```
Til now actually the EIGRP itself already working and traffic can pass between server and routers.
but next we going to configure the authentication so no rouge router can join out network.

### Configure EIGRP Authenticaion

it is very simple, the command below is configured to each interface that form the EIGRP network.

first we create the KEY
```
key chain EIGRP_AUTH
 key 1
  key-string MY_AUTH_KEY
```
then assign the key to the interface
```
interface GigabitEthernet0/0
 ip address 10.10.15.15 255.255.255.0
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP_AUTH
```
modify the hello interval and hold-time

the number "10" refers to the EIGRP autonomous system number, "5" refers to the hello interval in seconds.
the number "10" again refers to the EIGRP autonomous system number, and "15" refers to the hold time in seconds.
```
interface ethernet0/1
 ip hello-interval eigrp 10 5
 ip hold-time eigrp 10 15
```
Hardening EIGRP using passive interface. passive interface make the interface to not adventise the EIGRP. normally it is implemented in access link.
```
router eigrp 100
 passive-interface default
 no passive-interface gi0/1
```

Just that the EIGRP is configured.

## EIGRP for ipv6

For EIGRP for IPv6, you first have to assign IPv6 addresses to the interfaces. Here is the corrected example:

Enable IPv6 routing:
```
Router(config)# ipv6 unicast-routing
```
Go into the interface configuration mode and assign an IPv6 address to the interface:
```
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ipv6 address 2001:DB8:0:1::1/64
```
Enable EIGRP on the interface:
```
Router(config-if)# ipv6 eigrp 100
```
Repeat steps 2 and 3 for each interface that you want to participate in EIGRP.

Finally, enable the EIGRP routing process:
```
Router(config)# ipv6 router eigrp 100
Router(config-rtr)# no shutdown
```
This "no shutdown" command is required to start the EIGRP process for IPv6, unlike in IPv4.

Here's a full example with EIGRP AS 100, and enabling EIGRP on two interfaces:
```
Router(config)# ipv6 unicast-routing
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ipv6 address 2001:DB8:0:1::1/64
Router(config-if)# ipv6 eigrp 100
Router(config)# interface GigabitEthernet0/2
Router(config-if)# ipv6 address 2001:DB8:0:2::1/64
Router(config-if)# ipv6 eigrp 100
Router(config)# ipv6 router eigrp 100
Router(config-rtr)# no shutdown
```
Remember to replace the Autonomous System (AS) number and interface names with your actual values. Also, replace the IPv6 addresses with those appropriate for your network.



### Additional Note

EIGRP Loop-Free Alternate (LFA) Fast Reroute (FRR)

Split Horizon is a method where route information is prevented from being sent back in the direction it came from. By default, Split Horizon is enabled in EIGRP.

Poison Reverse is a technique used to prevent routing loops by advertising 'unreachable' routes back to the source they were learned from. In EIGRP, this behavior is automatic and there isn't a specific command to enable or disable it.

https://networklessons.com/cisco/ccie-routing-switching-written/eigrp-otp-over-the-top
