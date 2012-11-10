# OSPF LAN Adjacency

This example explores the state machine for establishing an OSPF adjacency over a LAN connection.

## Topology

### Physical Topology

![ospf-lan-adjacency-topology](https://raw.github.com/brunorijsman/firefly-configurations/master/features/ospf-lan-adjacency/ospf-lan-adjacency-topology.jpg)

### Protocols

![ospf-lan-adjacency-protocols](https://raw.github.com/brunorijsman/firefly-configurations/master/features/ospf-lan-adjacency/ospf-lan-adjacency-protocols.jpg)

## Configuration

### Physical Topology

```
## Last changed: 2012-11-07 09:33:58 UTC
version "12.2I0 [slt-builder]";
system {
    root-authentication {
        encrypted-password "$1$3RRU14Td$9NyynPTnDBN82m79i1uG10"; ## SECRET-DATA
    }
    services {
        ssh;
        web-management {
            http {
                interface ge-0/0/0.0;
            }
        }
    }
    syslog {
        file messages {
            any any;
        }
    }
    license {
        autoupdate {
            url https://ae1.juniper.net/junos/key_retrieval;
        }
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 192.168.229.155/24;
            }
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 12.12.12.1/24;
            }
        }
    }
    ge-0/0/2 {
        unit 0 {
            family inet {
                address 12.12.12.2/24;
            }
        }
    }
    lo0 {
        unit 1 {
            family inet {
                address 1.1.1.1/32;
            }
        }
        unit 2 {
            family inet {
                address 2.2.2.2/32;
            }
        }
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
    zones {
        security-zone trust {
            tcp-rst;
            host-inbound-traffic {
                system-services {
                    all;
                }
                protocols {
                    all;
                }
            }
            interfaces {
                ge-0/0/0.0;
            }
        }
    }
}
routing-instances {
    router-1 {
        instance-type virtual-router;
        interface ge-0/0/1.0;
        interface lo0.1;
    }
    router-2 {
        instance-type virtual-router;
        interface ge-0/0/2.0;
        interface lo0.2;
    }
}
```

### OSPF Configuration


```
[edit routing-instances router-1]
+    protocols {
+        ospf {
+            area 0.0.0.0 {
+                interface ge-0/0/1.0;
+                interface lo0.1;
+            }
+        }
+    }
[edit routing-instances router-2]
+    protocols {
+        ospf {
+            area 0.0.0.0 {
+                interface ge-0/0/2.0;
+                interface lo0.2;
+            }
+        }
+    }
```

## Validation

### OSPF Interfaces

OSPF interfaces on router-1. Note that router-1 is the Backup Designated Router (BDR) on the LAN:

<pre>
root> show ospf interface instance router-1 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/1.0          <font color="red">BDR</font>     0.0.0.0         <font color="red">2.2.2.2</font>         <font color="red">1.1.1.1</font>            1
lo0.1               DR      0.0.0.0         1.1.1.1         0.0.0.0            0
</pre>

OSPF interfaces on router-2. Note that router-2 is the Designated Router (DR) on the LAN:

<pre>
root> show ospf interface instance router-2    
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/2.0          <font color="red">DR</font>      0.0.0.0         <font color="red">2.2.2.2</font>         <font color="red">1.1.1.1</font>            1
lo0.2               DR      0.0.0.0         2.2.2.2         0.0.0.0            0
</pre>

### OSPF Neighbors

OSPF neighbors on router-1:

```
root> show ospf neighbor instance router-1 detail       
Address          Interface              State     ID               Pri  Dead
12.12.12.2       ge-0/0/1.0             Full      2.2.2.2          128    37
  Area 0.0.0.0, opt 0x52, DR 12.12.12.2, BDR 12.12.12.1
  Up 00:09:34, adjacent 00:09:02
```

OSPF neighbors on router-2:

```
root> show ospf neighbor instance router-2 detail    
Address          Interface              State     ID               Pri  Dead
12.12.12.1       ge-0/0/2.0             Full      1.1.1.1          128    34
  Area 0.0.0.0, opt 0x52, DR 12.12.12.2, BDR 12.12.12.1
  Up 00:10:49, adjacent 00:10:17
```

### OSPF Routes

OSPF routes on router-1:

```
root> show ospf route instance router-1    
Topology default Route Table:

Prefix             Path  Route      NH       Metric NextHop       Nexthop      
                   Type  Type       Type            Interface     Address/LSP
2.2.2.2            Intra Router     IP            1 ge-0/0/1.0    12.12.12.2
1.1.1.1/32         Intra Network    IP            0 lo0.1
2.2.2.2/32         Intra Network    IP            1 ge-0/0/1.0    12.12.12.2
12.12.12.0/24      Intra Network    IP            1 ge-0/0/1.0
```

OSPF routes on router-2:

```file:///Users/brunorijsman/firefly-configurations/features/ospf-lan-adjacency/adjancency-packet-flow.tiff
root> show ospf route instance router-2    
Topology default Route Table:

Prefix             Path  Route      NH       Metric NextHop       Nexthop      
                   Type  Type       Type            Interface     Address/LSP
1.1.1.1            Intra Router     IP            1 ge-0/0/2.0    12.12.12.1
1.1.1.1/32         Intra Network    IP            1 ge-0/0/2.0    12.12.12.1
2.2.2.2/32         Intra Network    IP            0 lo0.2
12.12.12.0/24      Intra Network    IP            1 ge-0/0/2.0
```

### Traceroute

Traceroute from router-1 to router-2:

```
root> traceroute 2.2.2.2 source 1.1.1.1 routing-instance router-1 
traceroute to 2.2.2.2 (2.2.2.2) from 1.1.1.1, 30 hops max, 40 byte packets
 1  2.2.2.2 (2.2.2.2)  6.491 ms  7.281 ms  5.985 ms
```

## Analysis

### The Experiment

The goal of this experiment to use Wireshark to capture the OSPF packets during the initial adjacency establishment.

Note that this experiment does not show the full complexity of adjacency establishment on a LAN interface. Since we only have two routers on the LAN, we only have a DR router and a BDR router, but we don't have any DRother routers.

Initially, I used `clear ospf neighbor instance router-1` to clear the adjacency. But then I realized that this did not accurately reproduce the sequence of events during the initial adjacency establishment because `clear ospf neighbor …` does not clear the link state database.

To more accurately reproduce the packet exchange during the _initial_ adjacency establishment I did something more drastic: 

* Delete the entire OSPF configuration
* Commit
* Start the Wireshark trace
* Reload the OSPF configuration
* Commit

```
[edit]
root# delete routing-instances 

[edit]
root# delete interfaces lo0  

[edit]
root# commit 
commit complete
```

Start the Wireshark trace here.

```
root# load override start-topology.conf 
load complete

[edit]
root# load patch ospf-1.patch 
load complete

[edit]
root# commit 
commit complete

[edit]
```

### Ladder Diagram

The Wireshark flow graph shows the packet ladder diagram for reestabilish the adjacency. Note that 224.0.0.5 is the "all OSPF routers" multicast address.

![adjancency-packet-flow](adjancency-packet-flow.tiff)

The Wireshark capture file is saved as `adjancency-packet-flow.pcapng` and contains a total of 41 packets.

We now look at some packets in more detail from the perspective of router-2.

### Init State

Router-2 starts sending Hello packets to Router-1; see packets #1, #2 and #3 in the capture file.

Interestingly router-2 starts out with router ID 12.12.12.1 (packet #1) and then changes to router ID 2.2.2.2 (packets #2 and #3). Evidently interface ge-0/0/2.0 came up before interface lo0.2.

Initially, router-2 has not yet received any Hello packets from Router-1. At this point, router-2 is in state Init. Router-2 does not report router-1 as an active neighbor in the Hello packets.

The following packet is an example of such a Hello (this is packet #3):

![packet-3](packet-3.tiff)

Once router-2 receives the first Hello from router-1, router-2 will start reporting router-1 as an active neighbor; see packets #5, #6, #9, #10, #13, and #15.

At this point, router-1 is not yet reporting router-2 as an active neighbor. Thus the adjacency is not yet in TwoWay state.

One consequence of not yet being in the TwoWay state is that both the DR and the BDR are still set to 0.0.0.0.

The following packet is an example of such a Hello (this is packet #15):

![packet-15](packet-15.tiff)

### TwoWay State

Eventually, router-1 and router-2 both report each other as active neighbors in their Hello packets.

At that point the adjacency goes to state TwoWay.

Once the adjacency reaches TwoWay router-2 will also pick the DR and the BDR. Since the priorities are the same and router-2 has the higher router ID, router-2 will select itself as the DR and router-1 as the BDR. 

All subsequent Hello packets (packets #17, #21, and #40) sent by router-2 report router-2 as the DR and router-1 as the BDR.

### ExStart State

Since router-2 is the DR it will move into the ExStart state. Router-1, being the BDR, also moves into the ExStart state. Router-1 and router-2 will start negotiating who will be the master and who will be the slave for the LSDB synchronization process.

Note: if we had more than two routers on the LAN then some of the routers would be DRother. The adjancies between DRother routers would remain in state TwoWay.

Both router-1 and router-2 will start by assuming they are the master. They will both send an empty DB Description packet with:

* The Initialize (I) bit set to 1.
* The More (M) bit set to 1.
* The Master/Slave (MS) bit set to 1 i.e. Master.
* A DD sequence number picket by themselves.

Packet #18 is the initial DB Description packet sent by router-2:

![packet-18](packet-18.tiff)

When router-1 receives the initialize DB Description from router-2 it will discover that router-1 has the lower router ID and will assume the slave role. Router-1 will indicate that it is the slave by sending a DB Description packet with:

Similarly, when router-2 receives the intialize DB Description from router-1 will discover that router-2 has the higher router ID and will assume the master role.

Each router will move to state Exchange and inform the other router of its decision by setting the Initial bit to 0 and by setting the Master/Slave bit according to its decission in all subsequent DB Description packet.

From this point on, the slave (router-1 in this example) will only send DB Description packets to acknowledge DB Description packets sent by the master (router-2 in this example).

## Exchange State

(To be continued from here)

 

