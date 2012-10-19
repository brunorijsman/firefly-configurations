# RIP

In this chapter shows how to configure the Routing Information Protocol (RIP) in Junos using a virtual topology using routing instances.

## Topology

We use the [five-nodes-p2p-and-lan](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) topology:

![Foo](https://raw.github.com/brunorijsman/firefly-configurations/master/topologies/five-nodes-p2p-and-lan/five-nodes-p2p-and-lan.jpg	)

## Basic RIP Configuration

Apply the following configuration patch[^1] to the topology configuration to enable RIP on all interfaces: 

[^1]: To apply a configuration patch enter the `load patch terminal` command in configuration mode, copy & paste the configuration patch, and press control-D. 

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

## Basic RIP Monitoring

Use `show route protocol rip` to see the RIP routes; use the `table` option to limit the output to the IPv4 table of one routing instance:

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

Use `show route … detail` to see more details of a RIP route:

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


## References

[Junos 12.2 RIP Configuration Guide](http://www.juniper.net/techpubs/en_US/junos12.2/information-products/pathway-pages/config-guide-routing/config-guide-routing-rip.pdf)






 