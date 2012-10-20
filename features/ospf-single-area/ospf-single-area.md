# OSPF Single Area

In this chapter shows how to configure the Open Shortest Path First (OSPF) in Junos using a virtual topology using routing instances. This is a basic OSPF configuration with a single area.

## Topology

We use the [five-nodes-p2p-and-lan](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) topology:

![Foo](https://raw.github.com/brunorijsman/firefly-configurations/master/topologies/five-nodes-p2p-and-lan/five-nodes-p2p-and-lan.jpg	)

## Basic OSPF Configuration

Apply the configuration patch [ospf-single-area-1.patch](ospf-single-area-1.patch) shown below to the topology configuration to enable ospf on all interfaces. The resulting configuration is [ospf-single-area-1.conf](ospf-single-area-1.conf).

To apply a configuration patch enter the `load patch terminal` command in configuration mode, copy & paste the configuration patch from below, and press control-D.


```
[edit routing-instances router-1]
+    protocols {
+        ospf {
+            area 0 {
+                interface all;
+            }
+        }
+    }
[edit routing-instances router-2]
+    protocols {
+        ospf {
+            area 0 {
+                interface all;
+            }
+        }
+    }
[edit routing-instances router-3]
+    protocols {
+        ospf {
+            area 0 {
+                interface all;
+            }
+        }
+    }
[edit routing-instances router-4]
+    protocols {
+        ospf {
+            area 0 {
+                interface all;
+            }
+        }
+    }
[edit routing-instances router-5]
+    protocols {                        
+        ospf {
+            area 0 {
+                interface all;
+            }
+        }
+    }
```

## Check Connectivity

If the topology and OSPF were properly configured, you should be able to reach any routing instance from any other routing instance:

Ping from router-1 to router-5:

```
root> ping instance router-1 source 1.1.1.1 5.5.5.5 
PING 5.5.5.5 (5.5.5.5): 56 data bytes
64 bytes from 5.5.5.5: icmp_seq=0 ttl=63 time=3.156 ms
64 bytes from 5.5.5.5: icmp_seq=1 ttl=63 time=2.176 ms
64 bytes from 5.5.5.5: icmp_seq=2 ttl=63 time=4.219 ms
```

Trace route from router-2 to router-4:

```
root> traceroute instance router-2 source 2.2.2.2 4.4.4.4 
traceroute to 4.4.4.4 (4.4.4.4) from 2.2.2.2, 30 hops max, 40 byte packets
 1  7.7.7.1 (7.7.7.1)  5.508 ms  5.406 ms  5.992 ms
 2  4.4.4.4 (4.4.4.4)  4.015 ms  5.739 ms  5.929 ms
```

## Basic OSPF Monitoring

Use `show route protocol ospf` to see the OSPF routes; use the `table <table>` option to limit the output to one route table:

```
root> show route protocol ospf table router-3.inet.0    

router-3.inet.0: 14 destinations, 15 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1.1.1.1/32         *[OSPF/10] 00:06:31, metric 1
                    > to 7.7.7.1 via ge-0/0/3.73
2.2.2.2/32         *[OSPF/10] 00:06:41, metric 1
                    > to 7.7.7.2 via ge-0/0/3.73
4.4.4.4/32         *[OSPF/10] 00:06:31, metric 2
                      to 7.7.7.1 via ge-0/0/3.73
                    > via lt-0/0/0.353
5.5.5.5/32         *[OSPF/10] 00:07:16, metric 1
                    > via lt-0/0/0.353
12.12.12.0/24      *[OSPF/10] 00:06:31, metric 2
                    > to 7.7.7.1 via ge-0/0/3.73
                      to 7.7.7.2 via ge-0/0/3.73
14.14.14.0/24      *[OSPF/10] 00:06:31, metric 2
                    > to 7.7.7.1 via ge-0/0/3.73
25.25.25.0/24      *[OSPF/10] 00:06:41, metric 2
                    > to 7.7.7.2 via ge-0/0/3.73
35.35.35.0/24       [OSPF/10] 00:07:21, metric 1
                    > via lt-0/0/0.353
45.45.45.0/24      *[OSPF/10] 00:07:16, metric 2
                    > via lt-0/0/0.353
224.0.0.5/32       *[OSPF/10] 00:07:26, metric 1
                      MultiRecv
```

Use `show route <destination> detail` to see more details of one OSPF route:

```
root> show route table router-3.inet.0 5.5.5.5 detail   

router-3.inet.0: 14 destinations, 15 routes (14 active, 0 holddown, 0 hidden)
5.5.5.5/32 (1 entry, 1 announced)
        *OSPF   Preference: 10
                Next hop type: Router, Next hop index: 689
                Address: 0x9348d00
                Next-hop reference count: 6
                Next hop: via lt-0/0/0.353, selected
                State: <Active Int>
                Age: 8:37       Metric: 1 
                Area: 0.0.0.0
                Task: router-3-OSPF
                Announcement bits (1): 0-KRT 
                AS path: I
```

The fields in the output are explained on the [show route detail](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) Juniper technical documentation webpage.

The `show route` command is a general purpose command for looking at routes of any protocol after they have been placed in a route table. OSPF also maintains additional internal information about each route which is not stored in the route table. Use the 'show ospf route' command to see this additional information. Use the `instance <instance-name>` option to limit the output to one routing instance. 

```
root> show ospf route instance router-2                      
Topology default Route Table:

Prefix             Path  Route      NH       Metric NextHop       Nexthop      
                   Type  Type       Type            Interface     Address/LSP
1.1.1.1            Intra Router     IP            1 ge-0/0/2.72   7.7.7.1
                                                    lt-0/0/0.122
3.3.3.3            Intra Router     IP            1 ge-0/0/2.72   7.7.7.3
4.4.4.4            Intra Router     IP            2 ge-0/0/2.72   7.7.7.1
                                                    lt-0/0/0.122
5.5.5.5            Intra Router     IP            2 ge-0/0/2.72   7.7.7.3
1.1.1.1/32         Intra Network    IP            1 ge-0/0/2.72   7.7.7.1
                                                    lt-0/0/0.122
2.2.2.2/32         Intra Network    IP            0 lo0.2
3.3.3.3/32         Intra Network    IP            1 ge-0/0/2.72   7.7.7.3
4.4.4.4/32         Intra Network    IP            2 ge-0/0/2.72   7.7.7.1
                                                    lt-0/0/0.122
5.5.5.5/32         Intra Network    IP            2 ge-0/0/2.72   7.7.7.3
7.7.7.0/24         Intra Network    IP            1 ge-0/0/2.72
12.12.12.0/24      Intra Network    IP            1 lt-0/0/0.122
14.14.14.0/24      Intra Network    IP            2 ge-0/0/2.72   7.7.7.1
                                                    lt-0/0/0.122
25.25.25.0/24      Intra Network    IP            1 lt-0/0/0.252
35.35.35.0/24      Intra Network    IP            2 ge-0/0/2.72   7.7.7.3
45.45.45.0/24      Intra Network    IP            3 ge-0/0/2.72   7.7.7.1
                                                    ge-0/0/2.72   7.7.7.3
                                                    lt-0/0/0.122
```

Specify a `<destination>` prefix and/or use the `detail` or `extensive` option to see more details.

```
root> show ospf route instance router-2 5.5.5.5/32 extensive 
Topology default Route Table:

Prefix             Path  Route      NH       Metric NextHop       Nexthop      
                   Type  Type       Type            Interface     Address/LSP
5.5.5.5            Intra Router     IP            2 ge-0/0/2.72   7.7.7.3
  area 0.0.0.0, origin 5.5.5.5, optional-capability 0x0
5.5.5.5/32         Intra Network    IP            2 ge-0/0/2.72   7.7.7.3
  area 0.0.0.0, origin 5.5.5.5, priority medium
```

Use `show ospf neighbor` to see information about OSPF neighbors. Use the `instance <instance-name>` option to limit the output to one routing instance. 

```
root> show ospf neighbor instance router-2             
Address          Interface              State     ID               Pri  Dead
7.7.7.3          ge-0/0/2.72            Full      3.3.3.3          128    38
7.7.7.1          ge-0/0/2.72            Full      1.1.1.1          128    30
12.12.12.1       lt-0/0/0.122           Full      1.1.1.1          128    38
```

Specify a `<neighbor>` address and/or use the `detail` or `extensive` option to see more details.

```
root> show ospf neighbor instance router-2 12.12.12.1 extensive 
Address          Interface              State     ID               Pri  Dead
12.12.12.1       lt-0/0/0.122           Full      1.1.1.1          128    37
  Area 0.0.0.0, opt 0x52, DR 0.0.0.0, BDR 0.0.0.0
  Up 00:12:01, adjacent 00:12:01
   Topology default (ID 0) -> Bidirectional
```

Use `show ospf interface` to see information about OSPF interfaces. Use the `instance <instance-name>` option to limit the output to one routing instance. 

```
root> show ospf interface instance router-2                 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/2.72         BDR     0.0.0.0         3.3.3.3         2.2.2.2            2
lo0.2               DR      0.0.0.0         2.2.2.2         0.0.0.0            0
lt-0/0/0.122        PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
lt-0/0/0.252        PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            0
```

Specify an `<interface-name>` and/or use the `detail` or `extensive` option to see more details.

```
root> show ospf interface instance router-2 lt-0/0/0.252 extensive 
Interface           State   Area            DR ID           BDR ID          Nbrs
lt-0/0/0.252        PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            0
  Type: P2P, Address: 0.0.0.0, Mask: 0.0.0.0, MTU: 4470, Cost: 1
  Adj count: 0
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 1
lt-0/0/0.252        PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            0
  Type: P2P, Address: 25.25.25.2, Mask: 255.255.255.0, MTU: 4470, Cost: 1
  Adj count: 0, Passive
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Passive, Cost: 1
```

Use `show ospf overview` to see information about an OSPF instance. Use the `instance <instance-name>` option to specify the routing instance.

```
root> show ospf overview instance router-2    
Instance: router-2
  Router ID: 2.2.2.2
  Route table index: 5
  LSA refresh time: 50 minutes
  Area: 0.0.0.0
    Stub type: Not Stub
    Authentication Type: None
    Area border routers: 0, AS boundary routers: 0
    Neighbors
      Up (in full state): 3
  Topology: default (ID 0)
    Prefix export count: 0
    Full SPF runs: 9
    SPF delay: 0.200000 sec, SPF holddown: 5 sec, SPF rapid runs: 3
    Backup SPF: Not Needed
``` 

Use `show ospf database` to see information about the OSPF database. Use the `instance <instance-name>` option to specify the routing instance.

```
root> show ospf database instance router-2                     

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   1.1.1.1          1.1.1.1          0x80000005  2018  0x22 0xcb22  96
Router  *2.2.2.2          2.2.2.2          0x80000004  2017  0x22 0x4fbe  84
Router   3.3.3.3          3.3.3.3          0x80000005  2017  0x22 0xa020  72
Router   4.4.4.4          4.4.4.4          0x80000004  2028  0x22 0x3efd  84
Router   5.5.5.5          5.5.5.5          0x80000003  2028  0x22 0x2a82  84
Network  7.7.7.3          3.3.3.3          0x80000002  2018  0x22 0x4dab  36
```

Use the `detail` or `extensive` option to provide more details:

```
root> show ospf database instance router-2 extensive              

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   1.1.1.1          1.1.1.1          0x80000005  2102  0x22 0xcb22  96
  bits 0x0, link count 6
  id 7.7.7.3, data 7.7.7.1, Type Transit (2)
    Topology count: 0, Default metric: 1
  id 1.1.1.1, data 255.255.255.255, Type Stub (3)
    Topology count: 0, Default metric: 0
  id 2.2.2.2, data 12.12.12.1, Type PointToPoint (1)
    Topology count: 0, Default metric: 1
  id 12.12.12.0, data 255.255.255.0, Type Stub (3)
    Topology count: 0, Default metric: 1
  id 4.4.4.4, data 14.14.14.1, Type PointToPoint (1)
    Topology count: 0, Default metric: 1
  id 14.14.14.0, data 255.255.255.0, Type Stub (3)
    Topology count: 0, Default metric: 1
  Topology default (ID 0)
    Type: PointToPoint, Node ID: 4.4.4.4
      Metric: 1, Bidirectional
    Type: PointToPoint, Node ID: 2.2.2.2
      Metric: 1, Bidirectional
    Type: Transit, Node ID: 7.7.7.3
      Metric: 1, Bidirectional
  Aging timer 00:24:57
  Installed 00:34:59 ago, expires in 00:24:58, sent 00:34:59 ago
  Last changed 00:34:59 ago, Change count: 4
[…more…]
```



## Basic OSPF tracing

Apply the configuration patch [ospf-single-area-2.patch](ospf-single-area-2.patch) shown below to the configuration to enable all OSPF tracing for routing instance router-1:

```
[edit routing-instances router-1 protocols ospf]
+  traceoptions {
+      file ospf-trace;
+      flag all;
+  }
```

Instead of turning on all ospf tracing, you can also enable tracing for one or more specific ospf features:

```
root# set flag ?
Possible completions:
  all                  Trace everything
  database-description  Trace database description packets
  error                Trace errored packets
  event                Trace OSPF state machine events
  flooding             Trace LSA flooding
  general              Trace general events
  graceful-restart     Trace graceful restart
  hello                Trace hello packets
  ldp-synchronization  Trace synchronization between OSPF and LDP
  lsa-ack              Trace LSA acknowledgment packets
  lsa-analysis         Trace LSA analysis
  lsa-request          Trace LSA request packets
  lsa-update           Trace LSA update packets
  normal               Trace normal events
  nsr-synchronization  Trace NSR synchronization events
  on-demand            Trace demand circuit extensions
  packet-dump          Dump the contents of selected packet types
  packets              Trace all OSPF packets
  policy               Trace policy processing
  restart-signaling    Trace restart signaling
  route                Trace routing information
  spf                  Trace SPF calculations
  state                Trace state transitions
  task                 Trace routing protocol task processing
  timer                Trace routing protocol timer processing
```

Back in operational mode, start monitoring the ospf-trace file:

```
root> monitor start ospf-trace
```

ospf trace message will start appearing:

```
*** ospf-trace ***
Oct 20 09:53:18.180524 task_process_events: recv ready for OSPF I/O./var/run/ppmd_control
Oct 20 09:53:18.180621 task_process_events: recv ready for OSPF I/O./var/run/ppmd_control
Oct 20 09:53:18.180625 task_timer_uset: timer OSPF I/O./var/run/ppmd_control_PPM Hold <Touched> set to offset 2:00 at 9:55:18
Oct 20 09:53:18.180631 OSPF hello from 7.7.7.2 (IFL 82, area 0.0.0.0) absorbed
Oct 20 09:53:18.716247 task_process_events: recv ready for OSPF I/O./var/run/ppmd_control
Oct 20 09:53:18.716305 task_process_events: recv ready for OSPF I/O./var/run/ppmd_control
Oct 20 09:53:18.716309 task_timer_uset: timer OSPF I/O./var/run/ppmd_control_PPM Hold <Touched> set to offset 2:00 at 9:55:18
Oct 20 09:53:18.716315 OSPF periodic xmit from (null) to 224.0.0.5 (IFL 81 area 0.0.0.0)
Oct 20 09:53:18.872581 task_process_events: recv ready for OSPF I/O./var/run/ppmd_control
Oct 20 09:53:18.872633 task_process_events: recv ready for OSPF I/O./var/run/ppmd_control
Oct 20 09:53:18.872637 task_timer_uset: timer OSPF I/O./var/run/ppmd_control_PPM Hold <Touched> set to offset 2:00 at 9:55:18
```

To stop monitoring:

```
root> monitor stop ospf-trace
```

## References

[Junos 12.2 OSPF Configuration Guide](http://www.juniper.net/techpubs/en_US/junos12.2/information-products/pathway-pages/config-guide-routing/config-guide-ospf.pdf)
