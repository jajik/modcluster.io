---
layout: page
title: Examples
modified: 2015-12-26T16:44:17+01:00
excerpt: "Examples"
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

**This site is work in progress** {{ site.time | date: '%Y-%m-%d' }}: Please, refer to [http://mod-cluster.jboss.org/](http://mod-cluster.jboss.org/) until the content here is complete.
{: .notice}

# Apache HTTP Server examples
Downlaod mod_cluster sources, compile and run with your own Apache HTTP Server on Linux platform. The following demo deals with
compile dependencies and a basic setup including SELinux.

[![asciicast](https://asciinema.org/a/7563u1eu6o5jlg3a0gk4wv69f.png)](https://asciinema.org/a/7563u1eu6o5jlg3a0gk4wv69f)

You can skip directly to compilation on [52s mark](https://asciinema.org/a/7563u1eu6o5jlg3a0gk4wv69f?t=52).

# Wildfly examples
Under construction...

# Tomcat examples
Under construction...

# JBoss AS7 examples
Under construction...

## How do I force mod_cluster to use HTTPS instead of AJP?

With JBoss AS7: [mod_cluster.conf & standalone-ha.xml](https://gist.github.com/Karm/6ac503924a1909564051)

## JBoss AS instances in Domain mode with mod_cluster load balancer

Goal: To have _Domain Controller_ and one _Host Controller_ with one server running
on a one box and a second _Host Controller_ with second server running on
the second box. Both servers should automatically register to the
mod_cluster load balancer running on the third box.

This example is valid for JBoss AS 7.x, JBoss EAP 6.x and Apache HTTP Server 2.2.x. It will work for Wildfly application server as well.
If you would like to use Apache HTTP Server 2.4.x, mind the slight configuration changes regarding
`Allow from...` vs. `Requires granted`.

Requirements: 3 virtual machines or physical servers, JBoss AS, Apache HTTP Server and mod_cluster distributions.

First, we are about to install the Apache HTTP Server to act as mod_cluster load balancer. Please, note the paths,
IP addresses and ports are arbitrary unless stated otherwise.

#### Apache HTTP Server

    Installed in : /opt/load-balancer/jboss-ews-2.x/httpd
    box IP:        192.168.122.74

 1. run ```.postinstall```
 1. make sure you have these modules in the appropriate ```modules/``` directory:
    * mod_slotmem.so
    * mod_proxy_cluster.so
    * mod_manager.so
    * mod_advertise.so
 1. comment out mod_proxy_balancer.so (it's incompatible with our mod_proxy_cluster.so)
 1. for the sake of this example, comment out any Listen directives in httpd.conf
 1. LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
 1. `/opt/load-balancer/jboss-ews-2.x/httpd/conf.d/mod_cluster.conf`
{% highlight apacheconf linenos %}
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule slotmem_module modules/mod_slotmem.so

LoadModule manager_module modules/mod_manager.so
LoadModule advertise_module modules/mod_advertise.so

Listen 192.168.122.74:2181

MemManagerFile "/opt/load-balancer/jboss-ews-2.x/httpd/cache/mod_cluster"

ServerName 192.168.122.74:2181

<IfModule manager_module>
    Listen 192.168.122.74:8847
    LogLevel debug
    <VirtualHost 192.168.122.74:8847>
        ServerName 192.168.122.74:8847
        <Directory />
            Order deny,allow
            Deny from all
            Allow from all
        </Directory>
        AdvertiseGroup 224.0.1.111:23365
        EnableMCPMReceive
        <Location /mcm>
            SetHandler mod_cluster-manager
            Order deny,allow
            Deny from all
            Allow from all
        </Location>
    </VirtualHost>
</IfModule>
{% endhighlight %}

#### JBoss AS

    Unzipped into: /opt/jboss-eap-6.x

 1. use ```/opt/jboss-eap-6.x/bin/add-user.sh``` to add these 3 users to ManagementRealm
    * admin
    * machine1
    * machine2
    Please, note that machine1 and machine2 user names are not arbitrary. Answer _yes_ to all the questions and use the same passwords (this is just for testing): ```R3dHatRulezz.``` or update the ```<secret value="``` attributes accordingly.
 2. copy ```domain``` directory so as to create the following structure:
        /opt/jboss-eap-6.x/
        ├── domain
        ├── machine1
        │   └── domain
        └── machine2
            └── domain

##### Domain Controller

Edit ```machine1/domain/configuration/domain.xml```, so as:
{% highlight xml %}
+++
<cluster-password>${jboss.messaging.cluster.password:jboss}</cluster-password>
+++
<socket-binding-group name="full-ha-sockets" default-interface="public">
    +++
    <socket-binding name="modcluster" port="0" multicast-address="224.0.1.111" multicast-port="23365"/>
    +++
</socket-binding-group>
+++
<server-groups>
    <server-group name="main-server-group" profile="full-ha">
        <jvm name="default">
            <heap size="713m" max-size="713m"/>
            <permgen max-size="128m"/>
        </jvm>
        <socket-binding-group ref="full-ha-sockets"/>
    </server-group>
</server-groups>
+++
{% endhighlight %}

Configure the master in ```machine1/domain/configuration/host-master.xml```:

{% highlight xml %}
+++
<host name="machine1-master" xmlns="urn:jboss:domain:...
+++
        <security-realm name="ManagementRealm">
            <server-identities>
                 <secret value="UjNkSGF0UnVsZXp6Lg=="/>
            </server-identities>
+++
{% endhighlight %}

##### Host Controller 1

Configure your Host Controller 1 in ```machine1/domain/configuration/host-slave.xml```. Please, note the name is not arbitrary, it must match the user names created above.

{% highlight xml %}
<host name="machine1" xmlns="urn:jboss:domain:...
+++
        <security-realm name="ManagementRealm">
            <server-identities>
                 <secret value="UjNkSGF0UnVsZXp6Lg=="/>
            </server-identities>
+++
<servers>
    <server name="server-one" group="main-server-group"/>
</servers>
+++
{% endhighlight %}

##### Host Controller 2

Configure your Host Controller 2  ```machine2/domain/configuration/host-slave.xml```:

{% highlight xml %}
    +++
    <host name="machine2" xmlns="urn:jboss:domain:...
    +++
            <security-realm name="ManagementRealm">
                <server-identities>
                     <secret value="UjNkSGF0UnVsZXp6Lg=="/>
                </server-identities>
    +++
    <servers>
        <server name="server-two" group="main-server-group">
            <socket-bindings port-offset="150"/>
        </server>
    </servers>
    +++
{% endhighlight %}

#### Overview of the current layout

<figure>
    <a href="/images/mod_cluster-example-visual-elements-by-red-hat.png"><img src="/images/mod_cluster-example-visual-elements-by-red-hat.png"></a>
    <figcaption>Clients, balancer and worker nodes.</figcaption>
</figure>

#### Run!

For the sake of this example, let's suppose you have the Apache HTTP Server residing on the box 192.168.122.74.
Copy your ```/opt/jboss-eap-6.x``` folder to the two boxes that are supposed to act as JBoss AS worker nodes,
e.g. 192.168.122.78 and 192.168.122.204.

 1. Make sure your firewalls allow UDP multicast between the nodes and that all the used ports are open
 1. Please, check firewalls and network interfaces again, i.e. Apache HTTP Server must be able to access AJP connector on your JBoss AS workers etc.
 1. Start Apache HTTP Server on 192.168.122.74
   * *./apachectl start* in */opt/load-balancer/jboss-ews-2.x/httpd/sbin*
   * one might appreciate seeing the mod_cluster messages: *tail -f /opt/load-balancer/jboss-ews-2.x/httpd/logs/error_log*
 1. Start Domain Controller on 192.168.122.78, /opt/jboss-eap-6.x/bin
   * *./domain.sh --host-config=host-master.xml -Djboss.domain.base.dir=/opt/jboss-eap-6.x/machine1/domain -Djboss.bind.address.management=192.168.122.78*
 1. Start Host Controller 1 on 192.168.122.78
   * *./domain.sh -Djboss.domain.base.dir=/opt/jboss-eap-6.x/machine1/domain --host-config=host-slave.xml -Djboss.domain.master.address=192.168.122.78 -Djboss.management.native.port=29999 -Djboss.bind.address.management=192.168.122.78 -Djboss.bind.address=192.168.122.78 -Djboss.bind.address.unsecure=192.168.122.78*
 1. Start Host Controller 2 on 192.168.122.204
   * *./domain.sh -Djboss.domain.base.dir=/opt/jboss-eap-6.x/machine2/domain --host-config=host-slave.xml -Djboss.domain.master.address=192.168.122.78  -Djboss.bind.address.management=192.168.122.204 -Djboss.bind.address=192.168.122.204 -Djboss.bind.address.unsecure=192.168.122.204*
 1. Access http://192.168.122.74:8847/mcm and wait dozen of seconds for two worker nodes to appear
 1. Deploy/manage/play with Domain Controller http://192.168.122.78:9990/
