---
layout: page
title: About mod_cluster
tags: [about, mod_cluster, load, balancer]
modified: 2015-12-26T14:16:08+01:00
comments: true
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

**This site is work in progress** {{ site.time | date: '%Y-%m-%d' }}: Please, refer to [http://mod-cluster.jboss.org/](http://mod-cluster.jboss.org/) until the content here is complete.
{: .notice}

---

mod_cluster is an httpd-based load balancer. Like mod_jk and mod_proxy, mod_cluster uses a communication channel to forward requests 
from httpd to one of a set of application server nodes. Unlike mod_jk and mod_proxy, mod_cluster leverages an additional connection 
between the application server nodes and httpd.


The application server nodes use this connection to transmit server-side load balance 
factors and lifecycle events back to httpd via a custom set of HTTP methods, affectionately called the 
Mod-Cluster Management Protocol (MCMP). This additional feedback channel allows mod_cluster to offer a level of 
intelligence and granularity not found in other load balancing solutions.


Within httpd, mod_cluster is implemented as a set of modules for httpd with mod_proxy enabled. Much of the logic 
comes from mod_proxy, e.g. mod_proxy_ajp provides all the AJP logic needed by mod_cluster. 

---

JBoss already prepares binary packages with httpd and mod_cluster so you can quickly try mod_cluster on the following platforms:

* Linux x86, x64, ia64
* Solaris x64, SPARC
* Windows x86, x64
* HP-UX PA-RISC, ia64


<a markdown="0" href="{{ site.url }}/documentation" class="btn">Documentation</a> <a markdown="0" href="{{ site.owner.downloads }}" class="btn">Downloads</a>

See sections [Building httpd modules](/documentation/#building-httpd-modules) and [Building worker-side components](/documentation/#building-worker-side-components) if you would like to compile from sources.


## Advantages
mod_cluster boasts the following advantages over other httpd-based load balancers:

### Dynamic configuration of httpd workers
Traditional httpd-based load balancers require explicit configuration of the workers available to a proxy. In mod_cluster, the bulk of the proxy's configuration resides on the application servers. The set of proxies to which an application server will communicate is determined either by a static list or using dynamic discovery via the advertise mechanism. The application server relays lifecycle events (e.g. server startup/shutdown) to the proxies allowing them to effectively auto-configure themselves. Notably, the graceful shutdown of a server will not result in a failover response by a proxy, as is the case with traditional httpd-based load balancers. 

### Server-side load balance factor calculation
In contrast with traditional httpd-based load balancers, mod_cluster uses load balance factors calculated and provided by the application servers, rather than computing these in the proxy. Consequently, mod_cluster offers a more robust and accurate set of load metrics than is available from the proxy. (see Load Metrics for more) 

### Fine grained web-app lifecycle control
Traditional httpd-based load balancers do not handle web application undeployments particularly well. From the proxy's perspective requests to an undeployed web application are indistinguishable from a request for an non-existent resource, and will result in 404 errors. In mod_cluster, each server forwards any web application context lifecycle events (e.g. web-app deploy/undeploy) to the proxy informing it to start/stop routing requests for a given context to that server. 

### AJP is optional
Unlike mod_jk, mod_cluster does not require AJP. httpd connections to application server nodes can use HTTP, HTTPS, or AJP.  
The original concepts are described in a [wiki](http://www.jboss.org/community/docs/DOC-11431).

## Requirements

### Balancer side
* Apache HTTP Server 2.2.15+ (legacy 2.2.8+)

### Worker side
mod_cluster java module is provided for all the undermentioned containers:

* Tomcat 6
* Tomcat 7
* Tomcat 8
* JBoss AS7+
* Wildfly

## Auxiliary

### Strong cryptography
mod_cluster contains mod_ssl, therefore the warning (copied from OpenSSL [web page](https://www.openssl.org/)).


**Warning:** Please remember that export/import and/or use of strong cryptography software, providing cryptography hooks, or even just communicating technical details about cryptography software is illegal in some parts of the world. So when you import this package to your country, re-distribute it from there or even just email technical suggestions or even source patches to the authors or other people you are strongly advised to pay close attention to any laws or regulations which apply to you. The authors of openssl are not liable for any violations you make here. So be careful, it is your responsibility.
{: .notice}

### Contributors
<table>
{% for contributor in site.github.contributors %}
<tr><td><img src="{{ contributor.avatar_url }}" class="bio-photo-small" alt="{{ contributor.login }} bio photo"></td><td><a href="{{ contributor.html_url }}">{{ contributor.login }}</a></td></tr>
{% endfor %}
</table>
