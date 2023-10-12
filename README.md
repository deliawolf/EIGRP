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
```
R4(config) # interface ethernet0/1
R4(config-if)#ip hello-interval eigrp 10 5
R4(config-if)#ip hold-time eigrp 10 15
```
Just that the EIGRP is configured.
