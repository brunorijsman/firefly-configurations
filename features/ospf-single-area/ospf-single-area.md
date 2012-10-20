# OSPF Single Area

**WORK IN PROGRESS**

In this chapter shows how to configure the Open Shortest Path First (OSPF) in Junos using a virtual topology using routing instances. This is a basic OSPF configuration with a single area.

## Topology

We use the [five-nodes-p2p-and-lan](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) topology:

![Foo](https://raw.github.com/brunorijsman/firefly-configurations/master/topologies/five-nodes-p2p-and-lan/five-nodes-p2p-and-lan.jpg	)

## Basic OSPF Configuration

Apply the configuration patch [@.patch](@) shown below to the topology configuration to enable ospf on all interfaces. The resulting configuration is [@.conf](@).

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

If the topology and ospf were properly configured, you should be able to reach any routing instance from any other routing instance:

Ping from router-1 to router-5:

```
```

Trace route from router-2 to router-4:

```
```

## Basic OSPF Monitoring

Use `show route protocol ospf` to see the ospf routes; use the `table <table>` option to limit the output to one route table:

```
```

Use `show route <destination> detail` to see more details of one ospf route:

```
```

The fields in the output are explained on the [show route detail](https://github.com/brunorijsman/firefly-configurations/tree/master/topologies/five-nodes-p2p-and-lan) Juniper technical documentation webpage.

Use `show route advertising-protocol ospf <neighbor>` see all the ospf routes advertised to one neighbor. Use the `table` option to limit the output to one route table:

```
```

Use `show route receive-protocol ospf <neighbor>` see all the ospf routes received from one neighbor. Use the `table` option to limit the output to one route table:

```
```

Use `show ospf neighbor` to see information about ospf neighbors. Use the `instance` option to limit the output to one routing instance:

```
```

Use `show ospf statistics` to see ospf statistics. Use the `instance` option to limit the output to one routing instance. Optionally specify a neighbor name (i.e. interface name) to limit the output to one neighbor:

```
```

## Basic ospf tracing

Apply the configuration patch [ospf-2.patch](https://github.com/brunorijsman/firefly-configurations/blob/master/features/ospf/ospf-2.patch) shown below to the configuration to enable all ospf tracing for routing instance router-1:

```
```

Note, instead of turning on all ospf tracing, you can also enable tracing for one or more specific ospf features:

```
```

Back in operational mode, start monitoring the ospf-trace file:

```
```

ospf trace message will start appearing:

```
```

To stop monitoring:

```
```

## References

[Junos 12.2 ospf Configuration Guide](http://www.juniper.net/techpubs/en_US/junos12.2/information-products/pathway-pages/config-guide-routing/config-guide-routing-ospf.pdf)
