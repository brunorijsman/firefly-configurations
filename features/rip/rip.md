# RIP

In this chapter shows how to configure the Routing Information Protocol (RIP) in Junos using a virtual topology using routing instances.

## Topology

We use the [five-nodes-p2p-and-lan](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) topology:

![Foo](https://raw.github.com/brunorijsman/firefly-configurations/master/topologies/five-nodes-p2p-and-lan/five-nodes-p2p-and-lan.jpg	)

## Basic RIP Configuration

Apply the configuration patch [rip-1.patch](https://github.com/brunorijsman/firefly-configurations/blob/master/features/rip/rip-1.patch) shown below to the topology configuration to enable RIP on all interfaces. The resulting configuration is [rip-1.conf](https://github.com/brunorijsman/firefly-configurations/blob/master/features/rip/rip-1.conf).

To apply a configuration patch enter the `load patch terminal` command in configuration mode, copy & paste the configuration patch from below, and press control-D.


```
[edit]
+  policy-options {
+      policy-statement rip-export-policy {
+          term term-1 {
+              from protocol [ direct rip ];
+              then accept;
+          }
+      }
+  }
[edit routing-instances router-1]
+    protocols {
+        rip {
+            group rip-group {
+                export rip-export-policy;
+                neighbor lt-0/0/0.121;
+                neighbor lt-0/0/0.141;
+                neighbor ge-0/0/1.71;
+            }
+        }
+    }
[edit routing-instances router-2]
+    protocols {
+        rip {
+            group rip-group {
+                export rip-export-policy;
+                neighbor ge-0/0/2.72;
+                neighbor lt-0/0/0.122;
+                neighbor lt-0/0/0.252;
+            }
+        }
+    }
[edit routing-instances router-3]
+    protocols {
+        rip {
+            group rip-group {
+                export rip-export-policy;
+                neighbor ge-0/0/3.73;
+                neighbor lt-0/0/0.353;
+            }
+        }
+    }
[edit routing-instances router-4]
+    protocols {
+        rip {
+            group rip-group {
+                export rip-export-policy;
+                neighbor lt-0/0/0.454;
+                neighbor lt-0/0/0.144;
+            }
+        }
+    }
[edit routing-instances router-5]
+    protocols {                        
+        rip {
+            group rip-group {
+                export rip-export-policy;
+                neighbor lt-0/0/0.455;
+                neighbor lt-0/0/0.355;
+            }
+        }
+    }
```

## Check Connectivity

If the topology and RIP were properly configured, you should be able to reach any routing instance from any other routing instance:

Ping from router-1 to router-5:

```
root> ping routing-instance router-1 source 1.1.1.1 5.5.5.5 
PING 5.5.5.5 (5.5.5.5): 56 data bytes
64 bytes from 5.5.5.5: icmp_seq=0 ttl=63 time=1.187 ms
64 bytes from 5.5.5.5: icmp_seq=1 ttl=63 time=2.195 ms
64 bytes from 5.5.5.5: icmp_seq=2 ttl=63 time=2.009 ms
```

Trace route from router-2 to router-4:

```
root> traceroute routing-instance router-2 source 2.2.2.2 4.4.4.4    
traceroute to 4.4.4.4 (4.4.4.4) from 2.2.2.2, 30 hops max, 40 byte packets
 1  7.7.7.1 (7.7.7.1)  5.246 ms  5.342 ms  5.924 ms
 2  4.4.4.4 (4.4.4.4)  4.010 ms  3.664 ms  3.885 ms
```

## Basic RIP Monitoring

Use `show route protocol rip` to see the RIP routes; use the `table <table>` option to limit the output to one route table:

```
root> show route protocol rip table router-2.inet.0    

router-2.inet.0: 15 destinations, 17 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1.1.1.1/32         *[RIP/100] 00:09:36, metric 2, tag 0
                      to 12.12.12.1 via lt-0/0/0.122
                    > to 7.7.7.1 via ge-0/0/2.72
3.3.3.3/32         *[RIP/100] 00:09:31, metric 2, tag 0
                    > to 7.7.7.3 via ge-0/0/2.72
4.4.4.4/32         *[RIP/100] 00:09:31, metric 3, tag 0
                      to 12.12.12.1 via lt-0/0/0.122
                    > to 7.7.7.1 via ge-0/0/2.72
5.5.5.5/32         *[RIP/100] 00:09:31, metric 3, tag 0
                    > to 7.7.7.3 via ge-0/0/2.72
7.7.7.0/24          [RIP/100] 00:09:36, metric 2, tag 0
                    > to 12.12.12.1 via lt-0/0/0.122
12.12.12.0/24       [RIP/100] 00:09:36, metric 2, tag 0
                    > to 7.7.7.1 via ge-0/0/2.72
14.14.14.0/24      *[RIP/100] 00:09:36, metric 2, tag 0
                    > to 12.12.12.1 via lt-0/0/0.122
                      to 7.7.7.1 via ge-0/0/2.72
35.35.35.0/24      *[RIP/100] 00:09:31, metric 2, tag 0
                    > to 7.7.7.3 via ge-0/0/2.72
45.45.45.0/24      *[RIP/100] 00:09:31, metric 3, tag 0
                      to 12.12.12.1 via lt-0/0/0.122
                      to 7.7.7.1 via ge-0/0/2.72
                    > to 7.7.7.3 via ge-0/0/2.72
224.0.0.9/32       *[RIP/100] 00:09:36, metric 1
                      MultiRecv
```

Use `show route <destination> detail` to see more details of one RIP route:

```
root> show route 4.4.4.4 table router-2.inet.0 detail       

router-2.inet.0: 15 destinations, 17 routes (15 active, 0 holddown, 0 hidden)
4.4.4.4/32 (1 entry, 1 announced)
        *RIP    Preference: 100
                Next hop type: Router
                Address: 0x940c010
                Next-hop reference count: 3
                Next hop: 12.12.12.1 via lt-0/0/0.122
                Next hop: 7.7.7.1 via ge-0/0/2.72, selected
                State: <Active Int>
                Age: 12:35      Metric: 3       Tag: 0 
                Task: router-2-RIPv2
                Announcement bits (2): 0-KRT 1-router-2-RIPv2 
                AS path: I
                Route learned from 12.12.12.1 expires in 162 seconds
                Route learned from 7.7.7.1 expires in 159 seconds
```

The fields in the output are explained on the [show route detail](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) Juniper technical documentation webpage.

Use `show route advertising-protocol rip <neighbor>` see all the RIP routes advertised to one neighbor. Use the `table` option to limit the output to one route table:

```
root> show route advertising-protocol rip 12.12.12.2 table router-2 

router-2.inet.0: 15 destinations, 17 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2.2.2.2/32         *[Direct/0] 1d 02:27:20
                    > via lo0.2
3.3.3.3/32         *[RIP/100] 07:12:29, metric 2, tag 0
                    > to 7.7.7.3 via ge-0/0/2.72
5.5.5.5/32         *[RIP/100] 07:12:29, metric 3, tag 0
                    > to 7.7.7.3 via ge-0/0/2.72
7.7.7.0/24         *[Direct/0] 17:19:13
                    > via ge-0/0/2.72
                    [RIP/100] 07:12:34, metric 2, tag 0
                    > to 12.12.12.1 via lt-0/0/0.122
25.25.25.0/24      *[Direct/0] 17:19:13
                    > via lt-0/0/0.252
35.35.35.0/24      *[RIP/100] 07:12:29, metric 2, tag 0
                    > to 7.7.7.3 via ge-0/0/2.72
```

Use `show route receive-protocol rip <neighbor>` see all the RIP routes received from one neighbor. Use the `table` option to limit the output to one route table:

```
root> show route receive-protocol rip 12.12.12.1 table router-2                    

router-2.inet.0: 15 destinations, 17 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1.1.1.1/32         *[RIP/100] 07:33:16, metric 2, tag 0
                      to 12.12.12.1 via lt-0/0/0.122
                    > to 7.7.7.1 via ge-0/0/2.72
4.4.4.4/32         *[RIP/100] 07:33:11, metric 3, tag 0
                      to 12.12.12.1 via lt-0/0/0.122
                    > to 7.7.7.1 via ge-0/0/2.72
7.7.7.0/24          [RIP/100] 07:33:16, metric 2, tag 0
                    > to 12.12.12.1 via lt-0/0/0.122
14.14.14.0/24      *[RIP/100] 07:33:16, metric 2, tag 0
                    > to 12.12.12.1 via lt-0/0/0.122
                      to 7.7.7.1 via ge-0/0/2.72
45.45.45.0/24      *[RIP/100] 07:33:11, metric 3, tag 0
                      to 12.12.12.1 via lt-0/0/0.122
                      to 7.7.7.1 via ge-0/0/2.72
                    > to 7.7.7.3 via ge-0/0/2.72
```

Use `show rip neighbor` to see information about RIP neighbors. Use the `instance` option to limit the output to one routing instance:

```
root> show rip neighbor instance router-2 
                  Local  Source          Destination     Send   Receive   In
Neighbor          State  Address         Address         Mode   Mode     Met 
--------          -----  -------         -----------     ----   -------  --- 
ge-0/0/2.72          Up 7.7.7.2         224.0.0.9       mcast  both       1
lt-0/0/0.122         Up 12.12.12.2      224.0.0.9       mcast  both       1
lt-0/0/0.252         Up 25.25.25.2      224.0.0.9       mcast  both       1
```

Use `show rip statistics` to see RIP statistics. Use the `instance` option to limit the output to one routing instance. Optionally specify a neighbor name (i.e. interface name) to limit the output to one neighbor:

```
root> show rip statistics instance router-2 lt-0/0/0.122 
RIPv2 info: port 520; holddown 120s. 
    rts learned  rts held down  rqsts dropped  resps dropped
              9              0              0              0

lt-0/0/0.122:  5 routes learned; 6 routes advertised; timeout 180s; update interval 30s
Counter                         Total   Last 5 min  Last minute
-------                   -----------  -----------  -----------
Updates Sent                       45           11            2
Triggered Updates Sent              3            0            0
Responses Sent                      0            0            0
Bad Messages                        0            0            0
RIPv1 Updates Received              0            0            0
RIPv1 Bad Route Entries             0            0            0
RIPv1 Updates Ignored               0            0            0
RIPv2 Updates Received             47           11            3
RIPv2 Bad Route Entries             0            0            0
RIPv2 Updates Ignored               0            0            0
Authentication Failures             0            0            0
RIP Requests Received               1            0            0
RIP Requests Ignored                0            0            0
none                                0            0            0
```

## Basic RIP tracing

Apply the configuration patch [rip-2.patch](https://github.com/brunorijsman/firefly-configurations/blob/master/features/rip/rip-2.patch) shown below to the configuration to enable all RIP tracing for routing instance router-1:

```
[edit routing-instances router-1 protocols rip]
+      traceoptions {
+          file rip-trace;
+          flag all;
+      }
```

Note, instead of turning on all RIP tracing, you can also enable tracing for one or more specific RIP features:

```
root# set flag ?
Possible completions:
  all                  Trace everything
  auth                 Trace RIP authentication
  error                Trace RIP errors
  expiration           Trace RIP route expiration processing
  general              Trace general events
  holddown             Trace RIP hold-down processing
  normal               Trace normal events
  nsr-synchronization  Trace NSR synchronization events
  packets              Trace all RIP packets
  policy               Trace policy processing
  request              Trace RIP information packets
  route                Trace routing information
  state                Trace state transitions
  task                 Trace routing protocol task processing
  timer                Trace routing protocol timer processing
  trigger              Trace RIP triggered updates
  update               Trace RIP update packets
```

Back in operational mode, start monitoring the rip-trace file:

```
root> monitor start rip-trace 
```

RIP trace message will start appearing:

```
*** rip-trace ***
Oct 19 18:06:59.041403 Preparing to send RIPv2 updates on nbr lt-0/0/0.141, group: rip-group.
Oct 19 18:06:59.041454 task_timer_uset: timer router-1-RIPv2_RIPv2 xmit timer <Touched SpawnJob> set to offset 0 at 18:06:59
Oct 19 18:06:59.041470 task_job_create_background: create prio 6 job RIPv2 xmit timer for task router-1-RIPv2
Oct 19 18:06:59.041518 background dispatch running job RIPv2 xmit timer for task router-1-RIPv2
Oct 19 18:06:59.041524 Update job: sending 20 msgs; nbr: lt-0/0/0.141; group: rip-group; msgp: 0x9339000.
Oct 19 18:06:59.041530  nbr lt-0/0/0.141; msgp 0x9339000.
Oct 19 18:06:59.041536          sending msg 0x9339004, 7 rtes 
Oct 19 18:06:59.041540 Update job done for nbr lt-0/0/0.141 group: rip-group
Oct 19 18:06:59.041546 task_job_delete: delete background job RIPv2 xmit timer for task router-1-RIPv2
Oct 19 18:06:59.041550 background dispatch completed job RIPv2 xmit timer for task router-1-RIPv2
Oct 19 18:06:59.935622 task_job_create_background: create prio 1 job "RIPv2 process rcvd response packet" for task router-1-RIPv2
Oct 19 18:06:59.935763 background dispatch running job "RIPv2 process rcvd response packet" for task router-1-RIPv2
Oct 19 18:06:59.935767 received response: sender 7.7.7.2, command 2, version 2, mbz: 0; 3 routes.
Oct 19 18:06:59.935773             2.2.2.2/0xffffffff: tag 0, nh         0.0.0.0, met 1.
Oct 19 18:06:59.935779  2.2.2.2/32: metric-in: 2, change: 2 -> 2; # gw: 2, pkt_upd_src 7.7.7.2, inx: 0, rte_upd_src 7.7.7.2
Oct 19 18:06:59.935785          12.12.12.0/0xffffff00: tag 0, nh         0.0.0.0, met 1.
Oct 19 18:06:59.935791  12.12.12.0/24: metric-in: 2, change: 2 -> 2; # gw: 1, pkt_upd_src 7.7.7.2, inx: 0, rte_upd_src 7.7.7.2
Oct 19 18:06:59.935797          25.25.25.0/0xffffff00: tag 0, nh         0.0.0.0, met 1.
Oct 19 18:06:59.935803  25.25.25.0/24: metric-in: 2, change: 2 -> 2; # gw: 2, pkt_upd_src 7.7.7.2, inx: 0, rte_upd_src 7.7.7.2
Oct 19 18:06:59.935809 task_job_delete: delete background job "RIPv2 process rcvd response packet" for task router-1-RIPv2
Oct 19 18:06:59.935842 background dispatch completed job "RIPv2 process rcvd response packet" for task router-1-RIPv2
Oct 19 18:07:01.313458 task_job_create_background: create prio 1 job "RIPv2 process rcvd response packet" for task router-1-RIPv2
Oct 19 18:07:01.313543 background dispatch running job "RIPv2 process rcvd response packet" for task router-1-RIPv2
```

To stop monitoring:

```
root> monitor stop rip-trace 
```

## References

[Junos 12.2 RIP Configuration Guide](http://www.juniper.net/techpubs/en_US/junos12.2/information-products/pathway-pages/config-guide-routing/config-guide-routing-rip.pdf)






 