# firefly-configurations

## Introduction

This repository contains a collection of [Junos](http://www.juniper.net/us/en/products-services/nos/junos/) configuration files with accompanying notes and diagrams designed to help you learn about Junos.

Junos is Juniper's network operating system which used across the entire Juniper product line of [routers](http://www.juniper.net/us/en/products-services/routing/), [switches](http://www.juniper.net/us/en/products-services/switching/), [security gateways](http://www.juniper.net/us/en/products-services/security/), and [more](http://www.juniper.net/us/en/products-services/) -- from the smallest branch office router to the biggest core router. 

The configurations in this repository should work on any Juniper product, but they have only been tested on [JunosV Firefly](http://forums.juniper.net/t5/Security-Mobility-Now/Edge-Say-Hello-to-Security-At-Scale/ba-p/161314) which is a virtualized version of the Juniper [SRX Series Service Gateway](http://www.juniper.net/us/en/products-services/security/srx-series/) running in a virtual machine. JunosV Firefly is currently in beta; controlled availability is scheduled for Q4'2012 and general availability (GA) is scheduled for 1H'2012.

If you don't have access to Juniper equipment you can use [Junosphere](http://www.juniper.net/us/en/products-services/software/junos-platform/junosphere/) to learn Junos. Junosphere allows you to build virtual networks in Juniper's cloud and experiment with them. These virtual networks can contain virtual Juniper routers, switches, security gateways, managers, etc. all running the real Junos operating system (not a simulation of it). You can also include virtualized hosts and testing equipment from partners in the topology. 

## Background

This is a personal project which is not sanctioned by or associated with Juniper Networks in any way. I use these configuration files myself to experiment with Junos features and learn. I am open sourcing them as a service to the community and to provide an easy path to learning Junos to anyone who is interested. See the [LICENSE](license.md) file for details on the open source license and *the complete absence of any warranty of any kind*.

Bruno Rijsman 