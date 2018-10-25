---
layout: page
title: Project mod_cluster
tags: [about, mod_cluster, load, balancer]
comments: true
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

## Introduction

Project mod_cluster is an intelligent load balancer.
Like mod_jk and mod_proxy, mod_cluster uses a communication channel to forward requests from the load balancer to one of a set of application server nodes.
Unlike mod_jk and mod_proxy, mod_cluster leverages an additional connection between the application server nodes and the load balancer.

The application server nodes use this connection to transmit server-side load balance factors and lifecycle events
back to load balancer via a custom set of HTTP methods, affectionately called the 
Mod-Cluster Management Protocol (MCMP). 
This additional feedback channel allows mod_cluster to offer a level of intelligence and granularity not found in other load balancing solutions.

Within Apache httpd, mod_cluster is implemented as a set of modules for httpd with enabled mod_proxy. 
Much of the logic comes from mod_proxy, e.g. mod_proxy_ajp provides all the AJP logic needed by mod_cluster.
A pure-Java load balancer implementation is available as part of [Undertow](http://undertow.io/).
Container integration modules are available for [WildFly](http://wildfly.org/) (formerly known as JBoss AS), JBoss EAP and [Tomcat](http://tomcat.apache.org/).

## Advantages
mod_cluster boasts the following advantages over other load balancers:

### Dynamic configuration of httpd workers
Traditional httpd-based load balancers require explicit configuration of the workers available to a proxy.
In mod_cluster, the bulk of the proxy's configuration resides on the application servers.
The set of proxies to which an application server will communicate is determined either by a static list or using dynamic discovery via the advertise mechanism.
The application server relays lifecycle events (e.g. server startup/shutdown) to the proxies allowing them to effectively auto-configure themselves. 
Notably, the graceful shutdown of a server will not result in a fail-over response by a proxy, as is the case with traditional httpd-based load balancers. 

### Server-side load balance factor calculation
In contrast with traditional load balancers, mod_cluster uses load balance factors calculated and provided by the application servers, rather than computing these in the proxy. 
Consequently, mod_cluster offers a more robust and accurate set of load metrics than is available from the proxy.

### Fine grained web-app lifecycle control
Traditional load balancers do not handle web application undeployments particularly well.
From the proxy's perspective requests to an undeployed web application are indistinguishable from a request for an non-existent resource, 
and will result in 404 errors.
In mod_cluster, each server forwards any web application context lifecycle events (e.g. web-app deploy/undeploy) to the proxy
informing it to start/stop routing requests for a given context to that server. 

### AJP is optional
Unlike mod_jk, mod_cluster does not require AJP.
Connections to application server nodes can use HTTP, HTTPS, or AJP.  
