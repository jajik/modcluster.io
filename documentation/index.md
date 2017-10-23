---
layout: page
title: Documentation
modified: 2015-12-26T17:18:50+01:00
excerpt: "mod_cluster documentation"
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

**This site is work in progress** {{ site.time | date: '%Y-%m-%d' }}: Please, refer to [http://mod-cluster.jboss.org/](http://mod-cluster.jboss.org/) until the content here is complete.
{: .notice}

## Overview

mod_cluster is an httpd-based load balancer. Like mod_jk and
mod_proxy, mod_cluster uses a communication channel to forward
requests from httpd to one of a set of application server nodes. Unlike
mod_jk and mod_proxy, mod_cluster leverages an additional connection
between the application server nodes and httpd. The application server
nodes use this connection to transmit server-side load balance factors
and lifecycle events back to httpd via a custom set of HTTP methods,
affectionately called the Mod-Cluster Management Protocol (MCMP). This
additional feedback channel allows mod_cluster to offer a level of
intelligence and granularity not found in other load balancing
solutions.


Within httpd, mod_cluster is implemented as a set of modules for httpd
with mod_proxy enabled. Much of the logic comes from mod_proxy, e.g.
mod_proxy_ajp provides all the AJP logic needed by mod_cluster.

###  Platforms

JBoss already prepares [binary
packages](http://www.jboss.org/mod_cluster/downloads.html) with httpd
and mod_cluster so you can quickly try mod_cluster on the following
platforms:

-   Linux x86, x64, ia64

-   Solaris x86, SPARC

-   Windows x86, x64, ia64

-   HP-UX PA-RISC, ia64

### Advantages
mod_cluster boasts the following advantages over other httpd-based load balancers:

* Dynamic configuration of httpd workers
Traditional httpd-based load balancers require explicit configuration of the workers available to a proxy. In mod_cluster, the bulk of the proxy's configuration resides on the application servers. The set of proxies to which an application server will communicate is determined either by a static list or using dynamic discovery via the advertise mechanism. The application server relays lifecycle events (e.g. server startup/shutdown) to the proxies allowing them to effectively auto-configure themselves. Notably, the graceful shutdown of a server will not result in a failover response by a proxy, as is the case with traditional httpd-based load balancers. 

* Server-side load balance factor calculation
In contrast with traditional httpd-based load balancers, mod_cluster uses load balance factors calculated and provided by the application servers, rather than computing these in the proxy. Consequently, mod_cluster offers a more robust and accurate set of load metrics than is available from the proxy. (see Load Metrics for more) 

* Fine grained web-app lifecycle control
Traditional httpd-based load balancers do not handle web application undeployments particularly well. From the proxy's perspective requests to an undeployed web application are indistinguishable from a request for an non-existent resource, and will result in 404 errors. In mod_cluster, each server forwards any web application context lifecycle events (e.g. web-app deploy/undeploy) to the proxy informing it to start/stop routing requests for a given context to that server. 

* AJP is optional
Unlike mod_jk, mod_cluster does not require AJP. httpd connections to application server nodes can use HTTP, HTTPS, or AJP.  
The original concepts are described in a [wiki](http://www.jboss.org/community/docs/DOC-11431).

### Requirements

#### Balancer side
* Apache HTTP Server 2.2.15+ (legacy 2.2.8+)

#### Worker side
mod_cluster java module is provided for all the undermentioned containers:

* Tomcat 6
* Tomcat 7
* Tomcat 8
* JBoss AS7+
* Wildfly

### Limitations

mod_cluster uses shared memory to keep the nodes description, the shared memory is created at the start of httpd and the structure of each item is fixed. The following cannot be changed by configuration directives.

* Max Alias length 40 characters (Host: hostname header, Alias in\<Host/\>).
* Max context length 40 (for example myapp.war deploys in /myapp/myapp is the context).
* Max balancer name length 40 (balancer property in mbean).
* Max JVMRoute string length 80 (JVMRoute in \<Engine/\>).
* Max load balancing group name length 20 (domain property in mbean).
* Max hostname length for a node 64 (address in the \<Connector/\>).
* Max port length for a node 7 (8009 is 4 characters, port in the \<Connector/\>).
* Max scheme length for a node 6 (possible values are http, https, ajp, liked with the protocol of \<Connector/\>).
* Max cookie name 30 (the header cookie name for sessionid default value: JSESSIONID from org.apache.catalina.Globals.SESSION_COOKIE_NAME).
* Max path name 30 (the parameter name for the sessionid default value: jsessionid from org.apache.catalina.Globals.SESSION_PARAMETER_NAME).
* Max length for a sessionid 120 (something like BE81FAA969BF64C8EC2B6600457EAAAA.node01).

### Downloads

Download the latest [mod_cluster release]({{ site.owner.downloads }}).


The release is comprised of the following artifacts:

* httpd binaries for common platforms
* Wildfly/JBoss AS/JBossWeb/Tomcat Java distribution

Alternatively, you can build from source using the [mod_cluster git repository](https://github.com/modcluster/mod_cluster) and [mod_proxy_cluster git repository](https://github.com/modcluster/mod_proxy_cluster).

### Configuration

If you want to skip the details and just set up a minimal working
installation of mod_cluster, see the [Quick Start Guide](#Quick_Start_Guide).

* Configuring [balancer](#balancer_config)
* Configuring [workers](#worker_config)

### Migration

Migrating from mod_jk or mod_proxy is fairly straightforward. In general, much of the configuration previously
found in httpd.conf is now defined in the application server worker nodes.

* Migrating from [mod_jk](#Migratingfrommod_jk)
* Migrating from [mod_proxy](#Migratingfrommod_proxy)

### SSL support

Both the request connections between httpd and the application server nodes, and the feedback channel
between the nodes and httpd can be secured. The former is achieved via the [mod_proxy_https module](#mod_proxy_https)
and a corresponding ssl-enabled HTTP connector in JBoss Web or Undertow. The latter requires the
[mod_ssl module](#UsingSSL) and [explicit configuration in JBoss AS/Web/Undertow](#worker_config).


mod_cluster contains mod_ssl, therefore the warning (copied from OpenSSL [web page](https://www.openssl.org/)).


**Strong cryptography warning:** Please remember that export/import and/or use of strong cryptography software, providing cryptography hooks, or even just communicating technical details about cryptography software is illegal in some parts of the world. So when you import this package to your country, re-distribute it from there or even just email technical suggestions or even source patches to the authors or other people you are strongly advised to pay close attention to any laws or regulations which apply to you. The authors of openssl are not liable for any violations you make here. So be careful, it is your responsibility.
{: .notice}


## Quick Start Guide

The following are the steps to set up a minimal working installation of
mod_cluster on a single httpd server and a single back end server,
either JBoss AS, JBossWeb, Undertow or Tomcat. The steps can be repeated to add as
many httpd servers or back end servers to your cluster as is desired.

---

The steps shown here are not intended to demonstrate how to set up a production install of mod_cluster;
for example [using SSL to secure access](#UsingSSL) to the httpd-side mod_manager component is not covered. See the
[balancer-side](#balancer_config) and
[worker-side](#worker_config) configuration documentation for the full set of configuration options.

### Download mod_cluster components

Download the latest [httpd and java release bundles]({{ site.owner.downloads }}). If there is no pre-built httpd bundle
appropriate for your OS or system architecture, you can [build the binary from source](#nativebuilding).

### Install the httpd binary

#### Install the whole httpd

The httpd-side bundles are zipped and include a full httpd installation. As they contain already an Apache httpd installation
you do not need to download Apache httpd. Just extract them in root, e.g.

```bash
cd /
unzip mod_cluster-1.3.1.Final-linux2-x64-ssl.zip
```

That will give you a full httpd install in your ```/opt/jboss``` directory with all mod_cluster modules.

#### Install only the mod_cluster modules

If you already have a working httpd install that you would prefer to
use, you'll need to download the bundle named mod_cluster httpd dynamic
libraries corresponding to you plaform, extract the modules and copy
them directory to your httpd install's module directory. 

```
cd /tmp
tar xvf mod_cluster-1.x.y.Final-linux2-x86-so.tar.gz
```
And then you have to copy the files below to your module directory:

* mod_slotmem.so
* mod_manager.so
* mod_proxy_cluster.so
* mod_advertise.so

**httpd version mismatch:** ```[warn] httpd version mismatch detected``` Please, beware that one cannot simply load the aforementioned modules into an arbitrary httpd installation. 
These modules were built with a particular minor httpd version and they cannot be used with an older one. 
For instance, mod_cluster modules built with httpd 2.4.x cannot be loaded into httpd 2.2.x. 
Always check the release notes as to which version was used during the build: [mod_cluster release]({{ site.owner.downloads }}).
{: .notice}

#### Install in an arbitrary directory

There is a script ```opt/jboss/httpd/sbin/installhome.sh``` in the httpd bundle distribution that allows reconfiguration of the
bundle installation so that it can run in user's home directory. To do that, simply extract the bundle in your home directory 
and run the script. Once that done, httpd will run on port 8000 and will accept MCMP messages on localhost:6666 and offer
```/mod_cluster_manager``` on the same host and port.

#### Install in Windows

Unzip the bundle corresponding to your architecture.
Change to the bin directory of the subfolder ```httpd-*``` where you unzipped the bundle and run the installconf.bat shell script.

{% highlight bash %}
cd httpd-2.4\bin
installconf.bat
{% endhighlight %}
You may run httpd directly by using:


```
httpd.exe
```
or install Apache HTTP Server as a service:


```
httpd.exe -k install -n myApache
```
and start the service via net start or using httpd.exe:



```
net start myApache
```
or


```
httpd.exe -k start
```


Note that on windows bundles have a flat directory structure, so you have *httpd-2.4/modules/* instead of ```opt/jboss/httpd/lib/httpd/modules```.

### Configure httpd

```httpd.conf``` is preconfigured with the Quick Start values. You should adapt the default values to your configuration.
There follows an example configuration. If you extracted the download bundle to root as shown above and you are using that
extract as your httpd install, httpd.conf is located in ```/opt/jboss/httpd/httpd/conf```.

{% highlight apacheconf linenos %}
LoadModule proxy_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy.so
LoadModule proxy_ajp_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy_ajp.so
LoadModule cluster_slotmem_module /opt/jboss/httpd/lib/httpd/modules/mod_cluster_slotmem.so
LoadModule manager_module /opt/jboss/httpd/lib/httpd/modules/mod_manager.so
LoadModule proxy_cluster_module /opt/jboss/httpd/lib/httpd/modules/mod_proxy_cluster.so
LoadModule advertise_module /opt/jboss/httpd/lib/httpd/modules/mod_advertise.so

<IfModule manager_module>
  Listen 127.0.0.1:6666
  ManagerBalancerName mycluster
  <VirtualHost 127.0.0.1:6666>
    <Location />
     Require ip 127.0.0
    </Location>

    KeepAliveTimeout 300
    MaxKeepAliveRequests 0
    #ServerAdvertise on http://@IP@:6666
    AdvertiseFrequency 5
    #AdvertiseSecurityKey secret
    #AdvertiseGroup @ADVIP@:23364
    EnableMCPMReceive

    <Location /mod_cluster_manager>
       SetHandler mod_cluster-manager
       Require ip 127.0.0
    </Location>

  </VirtualHost>
</IfModule>
{% endhighlight %}

**mod_cluster 1.2.x vs mod_cluster 1.3.x:** Note that from mod_cluster 1.3.x on, the slotmem module is called 
```LoadModule cluster_slotmem_module modules/mod_cluster_slotmem.so``` instead of former ```LoadModule slotmem_module modules/mod_slotmem.so```.
{: .notice}

### Install the worker-side binaries

First, extract the java libraries distribution to a temporary directory. One can grab the ```mod_cluster-java-libs-*-bin.zip``` from 
[mod_cluster release]({{ site.owner.downloads }}). The following text assumes it is extracted to ```/tmp/mod-cluster```.

Your next step depends on whether your target server is JBoss AS 5.x, JBoss AS 7.x, JBoss Web/Tomcat 6, 7, 8 or Wildfly (Undertow).

#### Installing in JBoss AS 5.x

Assuming \$JBOSS_HOME indicates the root of your JBoss AS install and
that you want to use mod_cluster in the AS's all config:

{% highlight bash %}
cp -r /tmp/mod-cluster/mod-cluster.sar $JBOSS_HOME/server/all/deploy
{% endhighlight %}

#### Installing in Tomcat

Assuming \$CATALINA_HOME indicates the root of your Tomcat install:
{% highlight bash %}
cp -r /tmp/mod-cluster/JBossWeb-Tomcat/lib/* $CATALINA_HOME/lib/
{% endhighlight %}

Note that you should remove in the ```$CATALINA_HOME/lib/``` directory the
```mod_cluster-container-tomcat6*``` file in Tomcat7 and the
```mod_cluster-container-tomcat7*``` in Tomcat 6.

#### Installing in Wildfly

TODO

### Configuring the server-side

#### Configuring mod_cluster with JBoss AS 5.x+

No post-installation configuration necessary!

#### Configuring mod_cluster with standalone JBoss Web or Tomcat

Edit the ```$CATALINA_HOME/conf/server.xml```file, adding the
following next to the other ```<Listener/>``` elements:

{% highlight xml %}
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener" advertise="true"/>
{% endhighlight %}


#### Start httpd

To start httpd do the following:

{% highlight bash %}
/opt/jboss/httpd/sbin/apachectl start
{% endhighlight %}


#### Start the back-end server

##### Starting JBoss AS

{% highlight bash %}
cd $JBOSS_HOME/bin
./run.sh -c all
{% endhighlight %}


##### Starting JBossWeb or Tomcat

{% highlight bash %}
cd $CATALINA_HOME
./startup.sh
{% endhighlight %}

##### Set up more back-end servers

Repeat the back-end server install and configuration steps for each
server in your cluster.

### Experiment with the Load Balancing Demo Application

TODO

## httpd configuration

### Apache httpd configuration

You need to load the modules that are needed for mod_cluster for example:

{% highlight apacheconf %}
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule cluster_slotmem_module modules/mod_cluster_slotmem.so
LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so
{% endhighlight %}

**mod_cluster 1.2.x vs mod_cluster 1.3.x:** Note that from mod_cluster 1.3.x on, the slotmem module is called 
```LoadModule cluster_slotmem_module modules/mod_cluster_slotmem.so``` instead of former ```LoadModule slotmem_module modules/mod_slotmem.so```.
{: .notice}

mod_proxy and mod_proxy_ajp are standard httpd modules. mod_slotmem is a shared slotmem memory provider.
mod_manager is the module that reads information from JBoss AS/JBossWeb/Tomcat and updates the shared memory
information. mod_proxy_cluster is the module that contains the balancer for mod_proxy. mod_advertise is an
additional module that allows httpd to advertise via multicast packets the IP and port where the mod_cluster
is listening. This multi-module architecture allows the modules to easily be changed depending on what the
customer wants to do.

For example when using http instead of AJP, only

{% highlight apacheconf %}
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
{% endhighlight %}

needs to be changed to:

{% highlight apacheconf %}
LoadModule proxy_http_module modules/mod_proxy_http.so
{% endhighlight %}

### mod_proxy configuration

mod_proxy directives like ProxyIOBufferSize could be used to configure mod_cluster. There is no need to use ProxyPass
directives because mod_cluster automatically configures which URLs have to be forwarded to JBossWEB.

### mod_slotmem/cluster_slotmem_module configuration

The actual version does not require any configuration directives.

### mod_proxy_cluster

#### CreateBalancers

CreateBalancers define how balancers are created in the httpd VirtualHosts, this is to allow directives like:

{% highlight apacheconf %}
ProxyPass / balancer://mycluster1/
{% endhighlight %}


* 0 &mdash; Create in all VirtualHosts defined in httpd.

* 1 &mdash; Don't create balancers (requires at least one ProxyPass/ProxyPassMatch to define the balancer names).

* 2 &mdash; Create only the main server.

**Default:** 0

**CreateBalancers 1:** When using 1 don't forget to configure the balancer in the ProxyPass directive, because the default is
empty stickysession and ```nofailover=Off``` and the values received via the MCMP CONFIG message are ignored.
{: .notice}

**Default: 0** Note [MODCLUSTER-430 &mdash; CreateBalancers behave the same with option 0 or 2](https://issues.jboss.org/browse/MODCLUSTER-430)
{: .notice}

#### UseAlias

Check that the Alias corresponds to the ServerName (See [Host Name Aliases](http://labs.jboss.com/file-access/default/members/jbossweb/freezone/docs/latest/config/host.html)).

* Off &mdash; Don't check (ignore Aliases)
* On &mdash; Check aliases

**Default:** Off Ignore the Alias information from the nodes.

**Older mod_cluster versions:** Versions older than 1.3.1.Final only accept values 0 and 1 respectively.
{: .notice}

#### LBstatusRecalTime
Time interval in seconds for loadbalancing logic to recalculate the status of a node.

**Default:** 5 seconds

The actual formula to recalculate the status of a node is:

{% highlight c %}
status = lbstatus + (elected - oldelected) * 1000)/lbfactor;
{% endhighlight %}

lbfactor is received for the node via STATUS messages.lbstatus is recalculated every LBstatusRecalTime seconds using the formula:

{% highlight c %}
lbstatus = (elected - oldelected) * 1000)/lbfactor;
{% endhighlight %}

```elected``` is the amount of time the worker has been elected. ```oldelected``` is ```elected``` last time the ```lbstatus```
was recalculated. The node with the lowest ```status``` is selected. Nodes with ```lbfactor ≤ 0``` are skipped by the both
calculation logics.

#### WaitForRemove

Time in seconds before a removed node is forgotten by httpd.

**Default:** 10 seconds

#### ProxyPassMatch/ProxyPass

ProxyPassMatch and ProxyPass are mod_proxy directives that when using ```!``` (instead the back-end url) prevent to
reverse-proxy in the path. This could be used allow httpd to serve static information like images.

{% highlight apacheconf %}
ProxyPassMatch ^(/.*\.gif)$ !
{% endhighlight %}

The above for example will allow httpd to server directly the .gif files.

#### EnableOptions

Use ```OPTIONS``` method to periodically check the active connection. Fulfils the same role as the ```CPING/CPONG``` used by AJP
but for HTTP/HTTPS connections. The endpoint needs to implement at least HTTP/1.1.

 * On (or no value) &mdash; Use OPTIONS
 * Off &mdash; Don't use OPTIONS

**Default:** On

### mod_manager

The Context of a mod_manger directive is VirtualHost except mentioned otherwise. **server config** means that it must be outside a
VirtualHost configuration. If not an error message will be displayed and httpd will not start.

#### EnableMCPMReceive

EnableMCPMReceive &mdash; allow the VirtualHost to receive Mod Cluster Protocol Messages (MCPM). You need one
EnableMCPMReceive in your httpd configuration to allow mod_cluster to
work, put it in the VirtualHost where you configure advertise.

---

This directive was added so as to address the issue of receiving MCPM on arbitrary VirtualHosts which was problematic due to accepting messages on insecure, unintended VirtualHosts.

#### MemManagerFile

That is the base name for the names mod_manager will use to store configuration, generate keys for shared memory or lock
files. That must be an absolute path name; the directories will created if needed. It is highly recommended that those
files are placed on a local drive and not an NFS share. (Context: **server config**)

**Default:** ```$server_root/logs/```
<script src="http://gist-it.appspot.com/github/modcluster/mod_cluster/blob/master/native/mod_manager/mod_manager.c?slice=521:538&footer=minimal"></script>

#### Maxcontext

The maximum number of application contexts supported by mod_cluster. (Context: **server config**)

**Default:**
<script src="http://gist-it.appspot.com/github/modcluster/mod_cluster/blob/master/native/mod_manager/mod_manager.c?slice=55:56&footer=minimal"></script>

#### Maxnode

That is the maximum number of nodes supported by mod_cluster. (Context: **server config**)

**Default:**
<script src="http://gist-it.appspot.com/github/modcluster/mod_cluster/blob/master/native/mod_manager/mod_manager.c?slice=56:57&footer=minimal"></script>

#### Maxhost

That is the maximum number of hosts (Aliases) supported by mod_cluster. That is also the max number of balancers. (Context: **server config**)

**Default:**
<script src="http://gist-it.appspot.com/github/modcluster/mod_cluster/blob/master/native/mod_manager/mod_manager.c?slice=57:58&footer=minimal"></script>

#### Maxsessionid

TODO

Maxsessionid: That is the number of active sessionid we store to give
number of active sessions in the mod_cluster-manager handler. A session
is unactive when mod_cluster doesn't receive any information from the
session in 5 minutes. (Context: server config)

Default: 0 (the logic is not activated).

#### MaxMCMPMaxMessSize

MaxMCMPMaxMessSize: Maximum size of MCMP messages. from other Max
directives.

Default: calculated from other Max directives. Min: 1024

#### ManagerBalancerName 

ManagerBalancerName: That is the name of balancer to use when the JBoss
AS/JBossWeb/Tomcat doesn't provide a balancer name.

Default: mycluster

#### PersistSlots

PersistSlots: Tell mod_slotmem to persist the nodes, Alias and Context
in files. (Context: server config)

Default: Off

#### CheckNonce

CheckNonce: Switch check of nonce when using mod_cluster-manager
handler on | off Since 1.1.0.CR1

Default: on Nonce checked

#### AllowDisplay

AllowDisplay: Switch additional display on mod_cluster-manager main
page on | off Since 1.1.0.GA

Default: off Only version displayed

#### AllowCmd 

AllowCmd: Allow commands using mod_cluster-manager URL on | off Since
mod_cluster 1.1.0.GA

Default: on Commmands allowed

#### ReduceDisplay 

ReduceDisplay - Reduce the information the main mod_cluster-manager
page to allow more nodes in the page. on | off

Default: off Full information displayed

#### SetHandler mod_cluster-manager 

SetHandler mod_cluster-manager: That is the handler to display the node
mod_cluster sees from the cluster. It displays the information about
the nodes like INFO and additionally counts the number of active
sessions.

{% highlight apacheconf linenos %}
# httpd 2.2.x and older
<Location /mod_cluster_manager>
   SetHandler mod_cluster-manager
   Order deny,allow
   Deny from all
   Allow from 127.0.0.1
</Location>
{% endhighlight %}
{% highlight apacheconf linenos %}
# httpd 2.4.x and on
<Location /mod_cluster_manager>
   SetHandler mod_cluster-manager
   Require ip 127.0.0
</Location>
{% endhighlight %}

When accessing the location you define in httpd.conf you get something
like:

TODO: Add pic

Note that:

Transferred: Corresponds to the POST data send to the back-end server.

Connected: Corresponds to the number of requests been processed when the
mod_cluster status page was requested.

sessions: Corresponds to the number of sessions mod_cluster report as
active (on which there was a request during the past 5 minutes). That
field is not present when Maxsessionid is zero.

### mod_advertise

mod_advertise uses multicast packets to advertise the VirtualHost where it is configured that must be the same VirtualHost
where mod_manager is defined. Of course at least one mod_advertise must be in the VirtualHost to allow mod_cluster to find
the right IP and port to give to the ClusterListener.

#### ServerAdvertise

ServerAdvertise On: Use the advertise mechanism to tell the JBoss
AS/JBossWeb/Tomcat to whom it should send the cluster information.

ServerAdvertise On http://hostname:port: Tell the hostname and port to use. Only needed if the VirtualHost is not defined
correctly, if the VirtualHost is a [Name-based Virtual Host](http://httpd.apache.org/docs/2.2/vhosts/name-based.html) or when
VirtualHost is not used.

ServerAdvertise Off: Don't use the advertise mechanism.

Default: Off. (Any Advertise directive in a VirtualHost sets it to On in
the VirtualHost)

#### AdvertiseGroup

AdvertiseGroup IP:port: That is the multicast address to use (something like 232.0.0.2:8888 for example).
IP should correspond to AdvertiseGroupAddress and port to AdvertisePort in the JBoss AS/JBossWeb/Tomcat configuration.
Note that if JBoss AS is used and the -u startup switch is included in the AS startup command, the default AdvertiseGroupAddress
is the value passed via the -u. If port is missing the default port will be used: 23364.

Default: 224.0.1.105:23364.

#### AdvertiseFrequency

AdvertiseFrequency seconds[.miliseconds]: Time between the multicast
messages advertising the IP and port.

Default: 10 Ten seconds.

#### AdvertiseSecurityKey

AdvertiseSecurityKey value: key string used to verify advertisements checksums. If configured on either side the verification
is required. Both sides must use the same security key.

Default: No default value.

#### AdvertiseManagerUrl

AdvertiseManagerUrl value: Not used in this version (It is sent in the X-Manager-Url: value header). That is the URL that
JBoss AS/JBossWeb/Tomcat should use to send information to mod_cluster

Default: No default value. Information not sent.

#### AdvertiseBindAddress

AdvertiseBindAddress IP:port: That is the address and port httpd is bind to send the multicast messages.
This allow to specify an address on multi IP address boxes.

Default: 0.0.0.0:23364

### Minimal Example

Beware of the different names of ```mod_cluster_slotmem.so``` and ```mod_slotmem.so``` between mod_cluster 1.3.x and older versions.
Last but not least, pay attention to httpd 2.2.x and httpd 2.4.x authentication configuration changes.

#### mod_cluster 1.3.x, Apache HTTP Server 2.4.x

{% highlight apacheconf linenos %}
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so

LoadModule cluster_slotmem_module modules/mod_cluster_slotmem.so

LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so

<IfModule manager_module>
  Listen 10.33.144.3:6666
  <VirtualHost 10.33.144.3:6666>

  # Where your worker nodes connect from
  <Location />
     Require ip 127.0.0
  </Location>

  ServerAdvertise On
  EnableMCPMReceive

  # Where administrator reads the console from
  <Location /mod_cluster_manager>
     SetHandler mod_cluster-manager
     Require ip 127.0.0
  </Location>

  </VirtualHost>
</IfModule>
{% endhighlight %}

#### mod_cluster 1.2.x, Apache HTTP Server 2.2.x

{% highlight apacheconf linenos %}
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so

LoadModule slotmem_module modules/mod_slotmem.so

LoadModule manager_module modules/mod_manager.so
LoadModule proxy_cluster_module modules/mod_proxy_cluster.so
LoadModule advertise_module modules/mod_advertise.so

<IfModule manager_module>
  Listen 10.33.144.3:6666
  <VirtualHost 10.33.144.3:6666>

  # Where your worker nodes connect from
  <Location />
     Order deny,allow
     Deny from all
     Allow from 127.0.0.
  </Location>

  ServerAdvertise On
  EnableMCPMReceive

  # Where administrator reads the console from
  <Location /mod_cluster_manager>
     SetHandler mod_cluster-manager
     Order deny,allow
     Deny from all
     Allow from 127.0.0.
  </Location>

  </VirtualHost>
</IfModule>
{% endhighlight %}

## Building httpd modules

mod_cluster 1.3.x and older, both httpd modules and Tomcat/Wildfly java libraries reside in the [mod_cluster](https://github.com/modcluster/mod_cluster) repository, branches 1.3.x and 1.2.x. New development of mod_cluster httpd modules takes place in the new repository: [mod_proxy_cluster](https://github.com/modcluster/mod_proxy_cluster).

See [ASCII recorded tutorial](https://asciinema.org/a/7563u1eu6o5jlg3a0gk4wv69f?t=52) on httpd modules compilation with your own system httpd.

### Build with httpd on Windows

We assume you already have a functional Apache HTTP Server on Windows. This example works with Apache Lounge HTTP Server.
We also assume the system has MS Visual Studio (Community Edition is ample) and CMake installed. The example operates in cmder shell, but it is not mandatory. A simple Windows cmd prompt would work too.

 * Download the [Apache Lounge distribution](http://www.apachelounge.com/download/). Our example uses [httpd-2.4.23-win64-VC14.zip](http://www.apachelounge.com/download/VC14/binaries/httpd-2.4.23-win64-VC14.zip).
 * unzipped:

{% highlight plain %}
C:\Users\karm
λ ls
httpd-2.4.23-win64-VC14/ httpd-2.4.23-win64-VC14.zip
{% endhighlight %}

 * Clone mod_proxy_cluster sources git:

{% highlight plain %}
git clone https://github.com/modcluster/mod_proxy_cluster.git
{% endhighlight %}

or download [zipped master branch directly](https://github.com/modcluster/mod_proxy_cluster/archive/master.zip).

* Proceed with env vars set and CMake build directory preparation:

{% highlight plain %}
C:\Users\karm\mod_proxy_cluster\native (master)
λ mkdir build

C:\Users\karm\mod_proxy_cluster\native (master)
λ cd build\

C:\Users\karm\mod_proxy_cluster\native\build (master)
λ vcvars64.bat
{% endhighlight %}

Here comes the only slightly tricky part: Apache Lounge httpd ships all necessary *.lib files with exported symbols but for mod_proxy. Since mod_proxy is our dependency, we have to generate these exported symbols from mod_proxy dll.

{% highlight plain %}
dumpbin /exports C:\Users\karm\Apache24\modules\mod_proxy.so> C:\Users\karm\Apache24\modules\mod_proxy.exports

echo LIBRARY mod_proxy.so> C:\Users\karm\Apache24\modules\mod_proxy.def

echo EXPORTS>> C:\Users\karm\Apache24\modules\mod_proxy.def

for /f "skip=19 tokens=4" %A in (C:\Users\karm\Apache24\modules\mod_proxy.exports) do echo %A >> C:\Users\karm\Apache24\modules\mod_proxy.def

lib /def:C:\Users\karm\Apache24\modules\mod_proxy.def /OUT:C:\Users\karm\Apache24\modules\mod_proxy.lib /MACHINE:X64 /NAME:mod_proxy.so
{% endhighlight %}

Let's run CMake:

{% highlight plain %}
C:\Users\karm\mod_proxy_cluster\native\build (master)
λ cmake ../ -G "NMake Makefiles" -DCMAKE_BUILD_TYPE=Release -DAPR_LIBRARY=C:\Users\karm\Apache24\lib\libapr-1.lib -DAPR_INCLUDE_DIR=C:\Users\karm\Apache24\include\ -DAPACHE_INCLUDE_DIR=C:\Users\karm\Apache24\include\ -DAPRUTIL_LIBRARY=C:\Users\karm\Apache24\lib\libaprutil-1.lib -DAPRUTIL_INCLUDE_DIR=C:\Users\karm\Apache24\include\ -DAPACHE_LIBRARY=C:\Users\karm\Apache24\lib\libhttpd.lib -DPROXY_LIBRARY=C:\Users\karm\Apache24\modules\mod_proxy.so
-- Found APR: C:/Users/karm/Apache24/lib/libapr-1.lib
-- Found APRUTIL: C:/Users/karm/Apache24/lib/libaprutil-1.lib
-- Found APACHE: C:/Users/karm/Apache24/include
-- Build files have been written to: C:/Users/karm/mod_proxy_cluster/native/build
{% endhighlight %}

Compile

{% highlight plain %}
C:\Users\karm\mod_proxy_cluster\native\build (master)
λ nmake
{% endhighlight %}

Directory modules now contains all necessary modules:

{% highlight plain %}
C:\Users\karm\mod_proxy_cluster\native\build (master)
λ cp modules/*.so C:\Users\karm\Apache24\modules\ -v
'modules/mod_advertise.so' -> 'C:/Users/karm/Apache24/modules/mod_advertise.so'
'modules/mod_cluster_slotmem.so' -> 'C:/Users/karm/Apache24/modules/mod_cluster_slotmem.so'
'modules/mod_manager.so' -> 'C:/Users/karm/Apache24/modules/mod_manager.so'
'modules/mod_proxy_cluster.so' -> 'C:/Users/karm/Apache24/modules/mod_proxy_cluster.so'
{% endhighlight %}

Done.

### Build httpd from its sources

To build httpd-2.2.x from its sources see [ASF httpd 2.2 doc](http://httpd.apache.org/docs/2.2/install.html), 
see [ASF httpd 2.4 doc](http://httpd.apache.org/docs/2.4/install.html) for httpd-2.4.x.

If needed, patch the httpd-2.2.x sources with (The patch prevents long
waiting time when the node IP can't be resolved that should not happen
so you can skip the patch part if you don't want to rebuild httpd).
[mod_proxy_ajp.patch](https://github.com/modcluster/mod_cluster/blob/master/native/mod_proxy_cluster/mod_proxy_ajp.patch)


    (cd modules/proxy
      patch -p0 < $location/mod_proxy_ajp.patch
    )

Configure httpd with something like:

    ./configure  --prefix=apache_installation_directory \
                 --with-mpm=worker \
                 --enable-mods-shared=most \
                 --enable-maintainer-mode \
                 --with-expat=builtin \
                 --enable-ssl \
                 --enable-proxy \
                 --enable-proxy-http \
                 --enable-proxy-ajp \
                 --disable-proxy-balancer

Rebuild (make) and reinstall (make install) after that.

### Build the 4 modules of mod_cluster

You need an httpd installation with mod_proxy (```--enable-proxy```) and ajp
protocol (```--enable-proxy-ajp```) enabled and with dso enabled (```--enable-so```).

Download the mod_cluster sources:

    git clone git://github.com/modcluster/mod_cluster.git

Build the mod_cluster modules components, for each subdirectory
advertise, mod_manager, mod_proxy_cluster and mod_slotmem do
something like:

{% highlight bash %}
sh buildconf
 ./configure --with-apxs=apxs_file_location
 make
 cp *.so apache_installation_directory/modules
{% endhighlight %}

Where apache_installation_directory is the location of an installed
version of httpd-2-2.x.

NOTE: You can ignore the libtool message on most platform:

{% highlight bash %}
libtool: install: warning: remember to run `libtool --finish apache_installation_directory/modules'
{% endhighlight %}

Once that is done use [Apache httpd
configuration](#native.config "Chapter 3. httpd configuration") to
configure mod_cluster.

### Build the mod_proxy module

It is only needed for httpd-2.2.x where x \< 11. Process like the other
mod_cluster modules.

## Installing httpd modules

Several bundles are available at TODO [http://www.jboss.org/mod_cluster/downloads.html](http://www.jboss.org/mod_cluster/downloads.html).

---

TODO: Amend links. The following article is dated.

---

In case you can't find a prepared package of mod_cluser in the download area, it is possible to build mod_cluster for the sources.
You need a distribution of httpd (at least 2.2.8) or (better) a source tarball of httpd and the sources of mod_cluster.

### Configuration

TODO 

A minimal configuration is needed in httpd (See [httpd.conf](#native.config "Chapter 3. httpd configuration")).
A listener must be a added in in JBossWEB conf/server.xml (See [Configuring JBoss AS/Web](#java.config "Chapter 6. Server-side Configuration")).

#### Installing and using the bundles

The bundles are tar.gz on POSIX platforms just extract them in root something like:

{% highlight bash %}
cd /
tar xvf mod-cluster-1.x.y-linux2-x86-ssl.tar.gz
{% endhighlight %}

The httpd.conf is located in */opt/jboss/httpd/httpd/conf* to quick test
just add something like in the [Minimal Example]({{ site.url }}/documentation#minimal-example).

To start httpd do the following:

    httpd/sbin/apachectl start

NOTE: Make sure to use SSL before going in production.

#### Worker-side Configuration 

##### JBoss AS

TODO: JBoss AS 5, 7, Wildfly (Undertow)

mod_cluster is supported in AS7 via the modcluster subsystem See [AS7](#java.AS7config "Chapter 7. AS7 modcluster subsystem Configuration").

In other AS version mod_cluster's configuration resides within the
following file:

    $JBOSS_HOME/server/$PROFILE/deploy/mod_cluster.sar/META-INF/mod_cluster-jboss-beans.xml

The entry point for mod_cluster's server-side configuration is the ```ModClusterListener``` bean, which delegates web container
(e.g. JBoss Web) specific events to a container agnostic event handler.

In general, the ```ModClusterListener``` bean defines:

1. A ```ContainerEventHandler``` in which to handle events from the web container.
1. A reference to the JBoss mbean server.

e.g.

{% highlight xml %}
<bean name="ModClusterListener" class="org.jboss.modcluster.container.jbossweb.JBossWebEventHandlerAdapter"> 
  <constructor> 
    <parameter class="org.jboss.modcluster.container.ContainerEventHandler"> 
      <inject bean="ModClusterService"/>
    </parameter> 
    <parameter class="javax.management.MBeanServer"> 
      <inject bean="JMXKernel" property="mbeanServer"/> 
    </parameter> 
  </constructor> 
</bean>
{% endhighlight %}


#### Configuration Properties 

The ```ModClusterConfig``` bean enumerates the configuration properties used by mod_cluster.
For the complete list of configuration properties and their default values, see the chapter entitled
[Server-side Configuration Properties](#java.properties "Chapter 9. Server-side Configuration Properties").

e.g.

{% highlight xml %}
<bean name="ModClusterConfig" class="org.jboss.modcluster.config.ModClusterConfig" mode="On Demand">
  <!-- Specify configuration properties here -->
</bean>
{% endhighlight %}

##### Connectors

Like mod_jk and mod_proxy_balancer, mod_cluster requires a connector in your server.xml to which to forward web requests.
Unlike mod_jk and mod_proxy_balancer, mod_cluster is not confined to AJP, but can use HTTP as well. While AJP is generally
faster, an HTTP connector can optionally be secured via SSL. If multiple possible connectors are defined in your server.xml,
mod_cluster uses the following algorithm to choose between them:

1. If an native (APR) AJP connector is available, use it.
1. If an AJP connector is available, use it.
1. Otherwise, choose the HTTP connector with the highest max threads.

##### Node Identity

Like mod_jk and mod_proxy_balancer, mod_cluster identifies each node via a unique
[jvm route](http://docs.jboss.org/jbossweb/2.1.x/config/engine.html). By default, mod_cluster uses the following algorithm to
assign the jvm route for a given node:

1.  Use the value from ```server.xml```, ```<Engine jvmRoute="..."/>```, if defined.
1.  Generate a jvm route using the configured TODO. The default implementation does the following:
    1.  Use the value of the ```jboss.mod_cluster.jvmRoute``` system property, if defined.
    1.  Generate a UUID.

While UUIDs are ideal for production systems, in a development or testing environment, it is useful to know which node served
a given request just by looking at the jvm route. In this case, you can utilize the ```org.jboss.modcluster.SimpleJvmRouteFactory```.
The factory generates jvm routes of the form:

*bind-address*:*port*:*engine-name*

#### JBoss Web & Tomcat

mod_cluster's entire configuration for JBoss Web or Tomcat resides entirely within ```$CATALINA_HOME/conf/server.xml```.

This limits the adds the following constraints to mod_cluster's feature set:

* Only non-clustered mode is supported
* [Only one load metric](#java.load "Chapter 10. Server-Side Load Metrics") can be used to calculate a load factor.

##### Lifecycle Listener

The entry point for JBoss Web and Tomcat configuration is the ModClusterListener.
All mod_cluster [configuration properties](#java.properties "Chapter 9. Server-side Configuration Properties") are defined as
attributes of the `<Listener/>`{.code} element. For the complete list of configuration properties and their default values, see
the chapter entitled [Server-side Configuration Properties](#java.properties "Chapter 9. Server-side Configuration Properties").

e.g.

{% highlight xml %}
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener" advertise="true"/>
{% endhighlight %}

##### Additional Tomcat dependencies

mod_cluster uses jboss-logging, which exists in JBoss Web, but not in Tomcat. Consequently, to use mod_cluster with Tomcat,
it is necessary to add
[jboss-logging-spi.jar](http://repository.jboss.org/nexus/content/groups/public-jboss/org/jboss/logging/jboss-logging-spi/)
to ```$CATALINA_HOME/lib```.

TODO: Removed migration guide from mod_cluster 1.0. Is it O.K.?

#### AS7 modcluster subsystem Configuration

JBoss Application Server 7+ mod_cluster subsystem configuration.

TODO: Mention XSD and links to it.

##### ModCluster Subsystem in JBoss AS7

The mod_cluster integration is done via the modcluster subsystem.

##### ModCluster Subsystem minimal configuration

TODO: Explain better for uninitiated readers.

The minimal configuration is having the modcluster ```schemaLocation``` in the ```schemaLocation``` list: 
```urn:jboss:domain:modcluster:1.0``` ```jboss-mod-cluster.xsd```.  and the ```extension module``` in the
```extensions``` list:

{% highlight xml %}
<extension module="org.jboss.as.modcluster"/>
{% endhighlight %}

and ```subsystem``` declaration like:

{% highlight xml %}
<subsystem xmlns="urn:jboss:domain:modcluster:2.0"/>
{% endhighlight %}

TODO: Explain advertisement.

With that configuration modcluster will listen for advertise on ```224.0.1.105:23364```.

#### ModCluster Subsystem configuration

##### mod-cluster-config Attributes

TODO: Link. The attributes correspond to the [properties](#java.properties).

##### Proxy Discovery Configuration

TODO: Explain better the placement for Attributes and properties in their respective contexts.

| Attribute               | Property              | Default                       |
|:------------------------|:--------------------- |:------------------------------|
| proxy-list              | proxyList             | *none*                        |
|----
| proxy-url               | proxyURL              | *none*                        |
|----
| advertise               | advertise             | *true*                        |
|----
| advertise-security-key  | advertiseSecurityKey  | *none*                        |
|----
| excluded-contexts       | excludedContexts      | *none*                        |
|----
| auto-enable-contexts    | autoEnableContexts    | *true*                        |
|----
| stop-context-timeout    | stopContextTimeout    | *10 seconds* (in seconds)     |
|----
| socket-timeout          | nodeTimeout           | *20 seconds* (in milliseconds)|
{: rules="groups"}

##### Proxy Configuration

| Attribute              | Property             | Default                      |
|:-----------------------|:---------------------|:-----------------------------|
| sticky-session         | stickySession        | *true*                       |
|----
| sticky-session-remove  | stickySessionRemove  | *false*                      |
|----
| sticky-session-force   | stickySessionForce   | *true*                       |
|----
| node-timeout           | workerTimeout        | *-1*                         |
|----
| max-attempts           | maxAttempts          | *1*                          |
|----
| flush-packets          | flushPackets         | *false*                      |
|----
| flush-wait             | flushWait            | *-1*                         |
|----
| ping                   | ping                 | *10*                         |
|----
| smax                   | smax                 | *-1* (it uses default value) |
|----
| ttl                    | ttl                  | *-1* (it uses default value) |
|----
| domain                 | loadBalancingGroup   | *none*                       |
|----
| load-balancing-group   | loadBalancingGroup   | *none*                       |
{: rules="groups"}


##### SSL Configuration

TODO: SSL configuration part needs to be added here too

##### simple-load-provider Attributes 

The simple load provider always sends the same load factor. Its purpose is testing, experiments and special scenarios such as [hot stand-by](#hot-stand-by). TODO: Link to Hot Stand-by.

{% highlight xml %}
<subsystem xmlns="urn:jboss:domain:modcluster:1.0">
  <mod-cluster-config>
    <simple-load-provider factor="1"/>
  </mod-cluster-config>
</subsystem>
{% endhighlight %}

|Attribute  | Property            | Default |
|:----------|:--------------------|:--------|
| factor    | LoadBalancerFactor  | *1*     |
{: rules="groups"}

##### dynamic-load-provider Attributes

The dynamic load provide allows to have ```load-metric``` as well as ```custom-load-metric``` defined. For example:

TODO: Check XSD for attributes and their descriptions.

{% highlight xml %}
<subsystem xmlns="urn:jboss:domain:modcluster:1.0">
  <mod-cluster-config advertise-socket="mod_cluster">
    <dynamic-load-provider history="10" decay="2">
       <load-metric type="cpu" weight="2" capacity="1"/>
       <load-metric type="sessions" weight="1" capacity="512"/>
       <custom-load-metric class="mypackage.myclass" weight="1" capacity="512">
          <property name="myproperty" value="myvalue" />
          <property name="otherproperty" value="othervalue" />
       </custom-load-metric>
    </dynamic-load-provider>
  </mod-cluster-config>
</subsystem>
{% endhighlight %}

| Attribute  | Property     | Default  |
|:-----------|:-------------|:---------|
| history    | history      | 512      |
|----
| decay      | decayFactor  | 512      |
{: rules="groups"}

##### load-metric Configuration

The load-metric are the "classes" collecting information to allow computation of the load factor sent to httpd.

| Attribute  | Property                   | Default     |
|:-----------|:---------------------------|:------------|
| type       | A Server-Side Load Metric  | *mandatory* |
|----
| weight     | weight                     | *9*         |
|----
| capacity   | capacity                   | *512*       |
{: rules="groups"}

##### Out-of-box load metric types

| Type             | Corresponding Server-Side Load Metric                     |
|:-----------------|:----------------------------------------------------------|
| cpu              | [AverageSystem](#average-system-load-metric)              |
|----
| mem              | [SystemMemoryUsage](#system-memory-usage-load-metric) See [MODCLUSTER-288](https://issues.jboss.org/browse/MODCLUSTER-288) |
|----
| heap             | [HeapMemoryUsage](#heap-memory-usage-load-metric)         |
|----
| sessions         | [ActiveSessions](#active-sessions-load-metric)            |
|----
| requests         | [RequestCount](#request-count-load-metric)                |
|----
| send-traffic     | [SendTraffic](#send-traffic-load-metric)                  |
|----
| receive-traffic  | [ReceiveTraffic](#receive-traffic-load-metric)            |
|----
| busyness         | [BusyConnectors](#busy-connectors-load-metric)            |
|----
| connection-pool  | [ConnectionPoolUsage](#connection-pool-usage-load-metric) |
{: rules="groups"}

##### custom-load-metric Configuration

The custom-load-metric are for user defined "classes" collecting information.
They are like the load-metric except ```type``` is replaced by ```class```:

|Attribute  | Property            | Default      |
|:----------|:--------------------|:-------------|
|class      | Name of your class  | *Mandatory*  |

See an [Example Custom Load Metric](https://github.com/Karm/mod_cluster-custom-load-metric) that reads load from a local file.

##### load-metric Configuration with the JBoss AS7 CLI

The load-metric have 4 commands to add / remove metrics

* add-metric: Allows to add a ```load-metric``` to the *dynamic-load-provider*, e.g.

    ```./:add-metric(type=cpu, weight=2, capacity=1)```

* remove-metric: Allows to remove a ```load-metric``` from the *dynamic-load-provider*, e.g.

    ./:remove-metric(type=cpu)

* add-custom-metric: Allows to add a ```load-custom-metric``` to the *dynamic-load-provider*, e.g.

    ./:add-custom-metric(class=myclass, weight=2, capacity=1, \
    property=[("pool" => "mypool"), ("var" => "myvariable")])

* remove-custom-metric: Allows to remove a ```load-custom-metric``` from the *dynamic-load-provider*, e.g.

    ./:remove-custom-metric(class=myclass)

## Building worker-side Components

### Requirements
Building mod_cluster's worker-side components from source requires the following tools:

* JDK 5.0+
* Maven 2.0+

### Building

Steps to build:

1.  Download the mod_cluster sources

    git clone git://github.com/modcluster/mod_cluster.git

1.  Use maven "dist" profile to build:

    cd mod_cluster
    mvn -P dist package

**Note:** Some unit tests require UDP port 23365. Make sure your local firewall allows the port.
{: .notice}

### Built Artifacts

The build produces the following output in the target directory:

* mod-cluster.sar
Exploded format sar to copy to the deploy dir in your JBoss AS install.

* JBossWeb-Tomcat/lib directory
Jar files to copy to the lib directory in your JBossWeb or Tomcat install to support use of mod_cluster.

* demo directory
The load balancing demo application. TODO: Explain further, link.

* mod-cluster-XXX.tar.gz
The full distribution tarball; includes the aforementioned elements.

## worker-side Configuration Properties 

The tables below enumerate the configuration properties available to an application server node.
The location for these properties depends on [how mod_cluster is configured](#java.config "Chapter 6. Server-side Configuration").

### Proxy Discovery Configuration

The list of proxies from which an application expects to receive AJP
connections is either defined statically, via the addresses defined in the [proxyList](#proxylist) 
configuration property; or discovered dynamically via the advertise mechanism. Using a special mod_advertise
module, proxies can advertise their existence by periodically broadcasting a multicast message containing their address:port.
This functionality is enabled via the [advertise](#advertise) configuration
property. If configured to listen, a server can learn of the proxy's existence, then notify that proxy of its
own existence, and update its configuration accordingly. This frees both the proxy *and* the server
from having to define static, environment-specific configuration values.

#### Session draining strategy

| Tomcat attribute        | AS7/Wildfly attribute     | Default       | Location | Scope    |
|:------------------------|:--------------------------|:--------------|:---------|:---------|
| sessionDrainingStrategy | session-draining-strategy | ```DEFAULT``` | Worker   | Worker   |
{: rules="groups"}
Indicates the session draining strategy used during undeployment of a web application. There are three possible values:

* ```DEFAULT```: Drain sessions before web application undeploy only if the web application is non-distributable.
* ```ALWAYS```: Always drain sessions before web application undeploy, even for distributable web applications.
* ```NEVER```: Do not drain sessions before web application undeploy, even for non-distributable web application.

#### Proxies

| Tomcat attribute        | AS7 attribute | Wildfly attribute | Default | Location | Scope    |
|:------------------------|:--------------|:------------------|:--------|:---------|----------|
| proxyList               | proxy-list    | proxies           | *None*  | Worker   | Worker   |
{: rules="groups"}

* Tomcat/AS7: Defines a comma delimited list of httpd proxies with which this node will initially communicate. Value should be of the form: *address1*:*port1*,*address2*:*port2*. Using the default configuration, this property can by manipulated via the jboss.mod_cluster.proxyList system property.
* Wildfly: In Wildfly, the ```proxy-list``` attribute of the modcluster subsystem element is deprecated. Instead, one uses an output socket binding. The following example leverages ```jboss-cli.sh```, e.g. :
  * Add a socket binding: ```/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=my-proxies:add(host=10.10.10.11,port=3333)```
  * Add the socket binding to the modcluster subsystem: ```/subsystem=modcluster/mod-cluster-config=configuration:write-attribute(name=proxies, value="my-proxies")```

#### Excluded contexts

| Tomcat attribute | AS7/Wildfly attribute  | Wildfly Default | Tomcat/AS7 Default | Location | Scope    |
|:-----------------|:-----------------------|:----------------|:-------------------|:---------|:---------|
| excludedContexts | excluded-contexts      | *None*          | ROOT, admin-console, invoker, bossws, jmx-console, juddi, web-console | Worker | Worker |
{: rules="groups"}

List of contexts to exclude from httpd registration, of the form: *host1*:*context1*,*host2*:*context2*,*host3*:*context3*
If no host is indicated, it is assumed to be the default host of the server (e.g. localhost). "ROOT" indicates the root context. Using the default configuration, this property can by manipulated via the jboss.mod_cluster.excludedContexts system property.

#### Auto Enable Contexts

| Tomcat attribute   | AS7/Wildfly attribute  | Default | Location | Scope  |
|:-------------------|:-----------------------|:--------|:---------|:-------|
| autoEnableContexts | auto-enable-contexts   | true    | Worker   | Worker |
{: rules="groups"}

If false the contexts are registered disabled in httpd, they need to be enabled via the enable() mbean method, jboss-cli command or via mod_cluster_manager web console on Apache HTTP Server.

#### Stop context timeout

| Tomcat attribute   | AS7/Wildfly attribute  | Default | Location | Scope  |
|:-------------------|:-----------------------|:--------|:---------|:-------|
| stopContextTimeout | stop-context-timeout   | 10 s    | Worker   | Worker |
{: rules="groups"}

The amount of time in seconds for which to wait for a clean shutdown of a context (completion of pending requests for a distributable context; or destruction/expiration of active sessions for a non-distributable context).

#### Stop context timeout unit

| Tomcat attribute       | AS7/Wildfly attribute  | Default | Location | Scope  |
|:-----------------------|:-----------------------|:--------|:---------|:-------|
| stopContextTimeoutUnit | *None*                 | TimeUnit.SECONDS | Worker   | Worker |
{: rules="groups"}

Tomcat allows for configuring an arbitrary TimeUnit for [Stop context timeout](#stop-context-timeout)

#### Proxy URL

| Tomcat attribute  | AS7/Wildfly attribute  | Default | Location | Scope  |
|:------------------|:-----------------------|:--------|:---------|:-------|
| proxyURL          | proxy-url              | /       | Worker   | Balancer |
{: rules="groups"}

If defined, this value will be prepended to the URL of MCMP commands.

#### Socket timeout

| Tomcat attribute  | AS7/Wildfly attribute  | Default | Location | Scope  |
|:------------------|:-----------------------|:--------|:---------|:-------|
| socketTimeout     | socket-timeout         | 20 s    | Worker   | Worker |
{: rules="groups"}

How long to wait for a response from an httpd proxy to MCMP commands before timing out, and flagging the proxy as in error.

#### Advertise

| Tomcat/AS7/Wildfly attribute  | Default                | Location | Scope  |
|:------------------------------|:-----------------------|:--------|:---------|
| advertise                     | true, if [proxyList](#proxies) is undefined, false otherwise | Worker   | Worker |
{: rules="groups"}

If enabled, httpd proxies will be auto-discovered via receiving multicast announcements. This can be used either in concert or in place of a static [proxies](#proxies).

#### Advertise socket group

| Tomcat attribute       | AS7/Wildfly attribute  | Default     | Location | Scope  |
|:-----------------------|:-----------------------|:------------|:---------|:-------|
| advertiseGroupAddress  | advertise-socket       | 224.0.1.105 | Worker   | Worker |
|---
| advertisePort          | in advertise-socket    | 23364       | Worker   | Worker |
{: rules="groups"}

UDP multicast address:port on which to listen for httpd proxy multicast advertisements. Beware of the actual *interface* your
balancer/worker sends to/receives from. See [MODCLUSTER-487](https://issues.jboss.org/browse/MODCLUSTER-487) for Apache HTTP Server behaviour and [MODCLUSTER-495](https://issues.jboss.org/browse/MODCLUSTER-495) for Tomcat's caveat.

#### Advertise security key

| Tomcat attribute     | AS7/Wildfly attribute  | Default | Location | Scope  |
|:---------------------|:-----------------------|:--------|:---------|:-------|
| advertiseSecurityKey | advertise-security-key | *None*  | Worker   | Balancer |
{: rules="groups"}

If specified, httpd proxy advertisements checksums (using this value as a salt) will be required to be verified on the server side. This option *does not* secure your installation, it *does not* replace proper SSL configuration. It merely ensures that only certain workers can talk to certain balancers. Beware of [MODCLUSTER-446](https://issues.jboss.org/browse/MODCLUSTER-446).

#### Advertise thread factory

| Tomcat attribute       | AS7/Wildfly attribute  | Default | Location | Scope  |
|:-----------------------|:-----------------------|:--------|:---------|:-------|
| advertiseThreadFactory | *None*                 | Executors.defaultThreadFactory() | Worker | Worker |
{: rules="groups"}

The thread factory used to create the background advertisement listener.

#### JVMRoute factory

| Tomcat attribute       | AS7/Wildfly attribute  | Default | Location | Scope  |
|:-----------------------|:-----------------------|:--------|:---------|:-------|
| jvmRouteFactory        | *None*                 | new SystemPropertyJvmRouteFactory(new UUIDJvmRouteFactory(), "jboss.mod_cluster.jvmRoute") | Worker | Worker |
{: rules="groups"}

Defines the strategy for determining the jvm route of a node, if none was specified in Tomcat's server.xml.
The default factory first consults the ```jboss.mod_cluster.jvmRoute``` system property. If this system property is undefined, the jvm route is assigned a UUID.
Wildfly with Undertow web subsystem uses Undertow's ```instance-id``` or ```jboss.mod_cluster.jvmRoute``` system property or a UUID.

### Proxy Configuration

The following configuration values are sent to proxies during server
startup, when a proxy is detected via the advertise mechanism, or during
the resetting of a proxy's configuration during error recovery.

| Attribute            | AS7 Attribute                | Default                            | Scope     | Description  |
|:---------------------|:-----------------------------|:-----------------------------------|:----------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| stickySession        | sticky-session               | true                               | Balancer  | Indicates whether subsequent requests for a given session should be routed to the same node, if possible.  |
|---
| stickySessionRemove  | sticky-session-remove        | false                              | Balancer  | Indicates whether the httpd proxy should remove session stickiness in the event that the balancer is unable to route a request to the node to which it is stuck. This property is ignored if [stickySession](#stickySession) is false. |
|---
| stickySessionForce   | sticky-session-force         | false                              | Balancer  | Indicates whether the httpd proxy should return an error in the event that the balancer is unable to route a request to the node to which it is stuck. This property is ignored if [stickySession](#stickySession) is false. |
|---
| workerTimeout        | worker-timeout               | -1                                 | Balancer  | Number of seconds to wait for a worker to become available to handle a request. When no workers of a balancer are usable, mod_cluster will retry after a while (workerTimeout/100). That is timeout in the balancer mod_proxy documentation. A value of -1 indicates that the httpd will not wait for a worker to be available and will return an error if none is available.  |
|---
| maxAttempts          | max-attempts                 | 1                                  | Balancer  | Maximum number of failover attempts before giving up. The minimum value is 0, i.e. no failover. The default value is 1, i.e. do a one failover attempt. |
|---
| flushPackets         | flush-packets                | false                              | Node      | Enables/disables packet flushing |
|---
| flushWait            | flush-wait                   | -1                                 | Node      | Time to wait before flushing packets in milliseconds. A value of -1 means wait forever.  |
|---
| ping                 | ping                         | 10                                 | Node      | Time (in seconds) in which to wait for a pong answer to a ping |
|---
| smax                 | smax                         | Determined by httpd configuration  | Node      | Soft maximum idle connection count (that is the smax in worker mod_proxy documentation). The maximum value depends on the httpd thread configuration (ThreadsPerChild or 1). |
|---
| ttl                  | ttl                          | 60                                 | Node      | Time to live (in seconds) for idle connections above smax  |
|---
| nodeTimeout          | node-timeout                 | -1                                 | Node      | Timeout (in seconds) for proxy connections to a node. That is the time mod_cluster will wait for the back-end response before returning error. That corresponds to timeout in the worker mod_proxy documentation. A value of -1 indicates no timeout. Note that mod_cluster always uses a cping/cpong before forwarding a request and the connectiontimeout value used by mod_cluster is the ping value. |
|---
| balancer             | balancer                     | mycluster                          | Node      | The balancer name  |
|---
| loadBalancingGroup   | domain load-balancing-group  | *None*                             | Node      | If specified, load will be balanced among jvmRoutes withing the same load balancing group. A loadBalancingGroup is conceptually equivalent to a mod_jk domain directive. This is primarily used in conjunction with partitioned session replication (e.g. buddy replication).  |
{: rules="groups"}


**Note:** When nodeTimeout is not defined the ProxyTimeout directive Proxy is
used. If ProxyTimeout is not defined the server timeout (Timeout) is
used (default 300 seconds). nodeTimeout, ProxyTimeout or Timeout is set
at the socket level.
{: .notice}

### SSL Configuration 

The communication channel between application servers and httpd proxies
uses HTTP by default. This channel can be secured using HTTPS by setting
the [ssl](#ssl-property) property to true.

**Note:** This HTTP/HTTPS channel should not be confused with the processing of
HTTP/HTTPS client requests.
{: .notice}

| Attribute                        | AS7 Attribute         | Default                                                          | Description   |
|:---------------------------------|:----------------------|:-----------------------------------------------------------------|:-----------------------------------------------------------------------------------------------   |
| ssl                              | *None*                | false                                                            | Should connection to proxy use a secure socket layer  |
|---
| sslCiphers                       | cipher-suite          | *The default JSSE cipher suites*                                 | Overrides the cipher suites used to initialize an SSL socket ignoring any unsupported ciphers   |
|---
| sslProtocol                      | protocol              | TLS (ALL in AS7)                                                 | Overrides the default SSL socket protocol.  |
|---
| sslCertificateEncodingAlgorithm  | *None*                | *The default JSSE key manager algorithm*                         | The algorithm of the key manager factory  |
|---
| sslKeyStore                      | certificate-key-file  | System.getProperty("user.home") + "/.keystore"          | The location of the key store containing client certificates  |
|---
| sslKeyStorePassword              | password              | changeit                                                         | The password granting access to the key store (and trust store in AS7)  |
|---
| sslKeyStoreType                  | *None*                | JKS                                                              | The type of key store   |
|---
| sslKeyStoreProvider              | *None*                | *The default JSSE security provider*                             | The key store provider  |
|---
| sslTrustAlgorithm                | *None*                | *The default JSSE trust manager algorithm*                       | The algorithm of the trust manager factory  |
|---
| sslKeyAlias                      | key-alias             |                                                                  | The alias of the key holding the client certificates in the key store   |
|---
| sslCrlFile                       | ca-revocation-url     |                                                                  | Certificate revocation list   |
|---
| sslTrustMaxCertLength            | *None*                | 5                                                                | The maximum length of a certificate held in the trust store   |
|---
| sslTrustStore                    | *None*                | javax.net.ssl.trustStorePassword  | The location of the file containing the trust store   |
|---
| sslTrustStorePassword            | *None*                | javax.net.ssl.trustStore          | The password granting access to the trust store.  |
|---
| sslTrustStoreType                | *None*                | javax.net.ssl.trustStoreType      | The trust store type  |
|---
| sslTrustStoreProvider            | *None*                | javax.net.ssl.trustStoreProvider  | The trust store provider  |
{: rules="groups"}

### Load Configuration for JBoss Web and Tomcat 

Additional configuration properties used when mod_cluster is configured
in JBoss Web standalone or Tomcat.

| Attribute           | Default | Description    |
| :--------------------| :---------------------------------------------------------------| :----------------------------------------------------------------------------------------   |
| loadMetricClass     | org.jboss.modcluster.load.metric.impl.BusyConnectorsLoadMetric | Class name of an object implementing org.jboss.load.metric.LoadMetric |
|---
| loadMetricCapacity  | 1                                                              | The capacity of the load metric defined via the loadMetricClass property   |
|---
| loadHistory         | 9                                                              | The number of historic load values to consider in the load balance factor computation.   |
|---
| loadDecayFactor     | 2                                                              | The factor by which a historic load values should degrade in significance.   |
{: rules="groups"}

## Worker-side Load Metrics 

A major feature of mod_cluster is the ability to use server-side load
metrics to determine how best to balance requests.

The `DynamicLoadBalanceFactorProvider`{.code} bean computes the load
balance factor of a node from a defined set of load metrics.

{% highlight xml %}
<bean name="DynamicLoadBalanceFactorProvider" class="org.jboss.modcluster.load.impl.DynamicLoadBalanceFactorProvider" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=LoadBalanceFactorProvider",exposedInterface=org.jboss.modcluster.load.impl.DynamicLoadBalanceFactorProviderMBean.class)</annotation>
  <constructor>
    <parameter>
      <set elementClass="org.jboss.modcluster.load.metric.LoadMetric">
        <inject bean="BusyConnectorsLoadMetric"/>
        <inject bean="HeapMemoryUsageLoadMetric"/>
      </set>
    </parameter>
  </constructor>
  <property name="history">9</property>
  <property name="decayFactor">2</property>
</bean>
{% endhighlight %}

Load metrics can be configured with an associated weight and capacity.

The weight (default is 1) indicates the significance of a metric with
respect to the other metrics. For example, a metric of weight 2 will
have twice the impact on the overall load factor than a metric of weight
1.

The capacity of a metric serves 2 functions:

-   To normalize the load values from each metric. In some load metrics,
    capacity is already reflected in the load values. The capacity of a
    metric should be configured such that 0 \<= (load / capacity) \>= 1.

-   To favor some nodes over others. By setting the metric capacities to
    different values on each node, proxies will effectively favor nodes
    with higher capacities, since they will return smaller load values.
    This adds an interesting level of granularity to node weighting.
    Consider a cluster of two nodes, one with more memory, and a second
    with a faster CPU; and two metrics, one memory-based and the other
    CPU-based. In the memory-based metric, the first node would be given
    a higher load capacity than the second node. In a CPU-based metric,
    the second node would be given a higher load capacity than the first
    node.

Each load metric contributes a value to the overall load factor of a
node. The load factors from each metric are aggregated according to
their weights.

In general, the load factor contribution of given metric is: (load /
capacity) \* weight / total weight.

The DynamicLoadBalanceFactorProvider applies a time decay function to
the loads returned by each metric. The aggregate load, with respect to
previous load values, can be expressed by the following formula:

L = (L<sub>0</sub>/D<sup>0</sup> + L <sub>1</sub>/D<sup>1</sup> + L<sub>2</sub>/D<sup>2</sup> + L<sub>3</sub>/D<sup>3</sup> + ... + L<sub>H</sub>/D<sup>H</sup>) / (1/D<sup>0</sup> + 1/D<sup>1</sup> + 1/D<sup>2</sup> + 1/D<sup>3</sup> + ... 1/D<sup>H</sup>)

... or more concisely as:

L = (∑<sup>H</sup><sub>i=0</sub> L<sub>i</sub>/D<sup>i</sup>) / (∑<sup>H</sup><sub>i=0</sub> 1/D<sup>i</sup>)

... where D = decayFactor, and H = history.

Setting history = 0 effectively disables the time decay function and
only the current load for each metric will be considered in the load
balance factor computation.

The mod_cluster load balancer expects the load factor to be an integer
between 0 and 100, where 0 indicates max load and 100 indicates zero
load. Therefore, the final load factor sent to the load balancer

L<sub>Final</sub> = 100 - (L \* 100)

While you are free to write your own load metrics, the following
LoadMetrics are available out of the box:

### Web Container metrics 

#### Active Sessions Load Metric

* Requires an explicit capacity
* Uses `SessionLoadMetricSource`{.code} to query session managers
* Analogous to method=S in mod_jk

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="ActiveSessionsLoadMetric" class="org.jboss.modcluster.load.metric.impl.ActiveSessionsLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=ActiveSessionsLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="SessionLoadMetricSource"/></parameter>
  </constructor>
  <property name="capacity">1000</property>
</bean>
<bean name="SessionLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.SessionLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
{% endhighlight %}


#### Busy Connectors Load Metric

* Returns the percentage of connector threads from the thread pool that are busy servicing requests
* Uses ```ThreadPoolLoadMetricSource``` to query connector thread
* Analogous to ```method=B``` in mod_jk
* [BusyConnectorsLoadMetric.java](https://github.com/modcluster/mod_cluster/blob/master/core/src/main/java/org/jboss/modcluster/load/metric/impl/BusyConnectorsLoadMetric.java)

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="BusyConnectorsLoadMetric" class="org.jboss.modcluster.load.metric.impl.BusyConnectorsLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=BusyConnectorsLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="ThreadPoolLoadMetricSource"/></parameter>
  </constructor>
</bean>
<bean name="ThreadPoolLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.ThreadPoolLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
{% endhighlight %}


#### Receive Traffic Load Metric

* Returns the incoming request POST traffic in KB/sec (the application needs to read POST data)
* Requires an explicit capacity
* Uses ```RequestProcessorLoadMetricSource``` to query request processors
* Analogous to ```method=T``` in mod_jk

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="ReceiveTrafficLoadMetric" class="org.jboss.modcluster.load.metric.impl.ReceiveTrafficLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=ReceiveTrafficLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource">
      <inject bean="RequestProcessorLoadMetricSource"/>
    </parameter>
  </constructor>
  <property name="capacity">1024</property>
</bean>
<bean name="RequestProcessorLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
{% endhighlight %}


#### Send Traffic Load Metric

* Returns the outgoing request traffic in KB/sec
* Requires an explicit capacity
* Uses `RequestProcessorLoadMetricSource`{.code} to query request processors
* Analogous to method=T in mod_jk

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="SendTrafficLoadMetric" class="org.jboss.modcluster.load.metric.impl.SendTrafficLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=SendTrafficLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource">
      <inject bean="RequestProcessorLoadMetricSource"/>
    </parameter>
  </constructor>
  <property name="capacity">512</property>
</bean>
{% endhighlight %}


#### Request Count Load Metric

* Returns the number of requests/sec
* Requires an explicit capacity
* Uses ```RequestProcessorLoadMetricSource``` to query request processors
* Analogous to ```method=R``` in mod_jk

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="RequestCountLoadMetric" class="org.jboss.modcluster.load.metric.impl.RequestCountLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=RequestCountLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter class="org.jboss.modcluster.load.metric.impl.RequestProcessorLoadMetricSource">
      <inject bean="RequestProcessorLoadMetricSource"/>
    </parameter>
  </constructor>
  <property name="capacity">1000</property>
</bean>
{% endhighlight %}

### System/JVM metrics 

#### Average System Load Metric

* Returns CPU load
* Requires Java 1.6+
* Uses ```OperatingSystemLoadMetricSource``` to generically read attributes
* Is not available on Windows
* [AverageSystemLoadMetric.java](https://github.com/modcluster/mod_cluster/blob/master/core/src/main/java/org/jboss/modcluster/load/metric/impl/AverageSystemLoadMetric.java)

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="AverageSystemLoadMetric" class="org.jboss.modcluster.load.metric.impl.AverageSystemLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=AverageSystemLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="OperatingSystemLoadMetricSource"/></parameter>
  </constructor>
</bean>
<bean name="OperatingSystemLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.OperatingSystemLoadMetricSource" mode="On Demand">
</bean>
{% endhighlight %}


#### Heap Memory Usage Load Metric

* Returns the heap memory usage as a percentage of max heap size

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="HeapMemoryUsageLoadMetric" class="org.jboss.modcluster.load.metric.impl.HeapMemoryUsageLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=HeapMemoryUsageLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
</bean>
{% endhighlight %}


### Other metrics 

#### ConnectionPoolUsageLoadMetric

* Returns the percentage of connections from a connection pool that are in use
* Uses ConnectionPoolLoadMetricSource to query JCA connection pools

e.g., with JBoss AS 5:

{% highlight xml %}
<bean name="ConnectionPoolUsageMetric" class="org.jboss.modcluster.load.metric.impl.ConnectionPoolUsageLoadMetric" mode="On Demand">
  <annotation>@org.jboss.aop.microcontainer.aspects.jmx.JMX(name="jboss.web:service=ConnectionPoolUsageLoadMetric",exposedInterface=org.jboss.modcluster.load.metric.LoadMetricMBean.class)</annotation>
  <constructor>
    <parameter><inject bean="ConnectionPoolLoadMetricSource"/></parameter>
  </constructor>
</bean>
<bean name="ConnectionPoolLoadMetricSource" class="org.jboss.modcluster.load.metric.impl.ConnectionPoolLoadMetricSource" mode="On Demand">
  <constructor>
    <parameter class="javax.management.MBeanServer">
      <inject bean="JMXKernel" property="mbeanServer"/>
    </parameter>
  </constructor>
</bean>
{% endhighlight %}


## Installing Worker-side Components

First, extract the server-side binary to a temporary directory. The
following assumes it was extracted to /tmp/mod_cluster

Your next step depends on whether your target server is JBoss AS or
JBossWeb/Tomcat.

### Installing in JBoss AS 6.0.0.M1 and up

You don't need to do anything to install the java-side binaries in AS
6.x; it's part of the AS distribution's default, standard and all
configurations.

### Installing in JBoss AS 5.x

Assuming \$JBOSS_HOME indicates the root of your JBoss AS install and
that you want to use mod_cluster in the AS's all config:

{% highlight bash %}
cp -r /tmp/mod_cluster/mod_cluster.sar $JBOSS_HOME/server/all/deploy
{% endhighlight %}


### Installing in Tomcat

Assuming \$CATALINA_HOME indicates the root of your Tomcat install:

{% highlight bash %}
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/jboss-logging.jar $CATALINA_HOME/lib/
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-catalina* $CATALINA_HOME/lib/
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-spi* $CATALINA_HOME/lib/
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-core* $CATALINA_HOME/lib/
{% endhighlight %}

and additionally for Tomcat6:

{% highlight bash %}
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-tomcat6* $CATALINA_HOME/lib
{% endhighlight %}

and additionally for Tomcat7:

{% highlight bash %}
cp /tmp/mod_cluster/JBossWeb-Tomcat/lib/mod_cluster-container-tomcat7* $CATALINA_HOME/lib
{% endhighlight %}


## Using SSL in mod_cluster

[12.3. Forwarding SSL browser informations when using http/https between
httpd and JBossWEB](#d0e3057)

There are 2 connections between the cluster and the front-end. Both
could be encrypted. That chapter describes how to encrypt both
connections.

### Using SSL between JBossWEB and httpd

As the ClusterListener allows to configure httpd it is adviced to use
SSL for that connection. The most easy is to use a virtual host that
will only be used to receive information from JBossWEB. Both side need
configuration.

#### Apache httpd configuration part 

[mod_ssl](http://httpd.apache.org/docs/2.2/mod/mod_ssl.html) of httpd
is using to do that. See in one example how easy the configuration is:

```
 Listen 6666
 <VirtualHost 10.33.144.3:6666>
     SSLEngine on
     SSLCipherSuite AES128-SHA:ALL:!ADH:!LOW:!MD5:!SSLV2:!NULL
     SSLCertificateFile conf/server.crt
     SSLCertificateKeyFile conf/server.key
     SSLCACertificateFile conf/server-ca.crt
     SSLVerifyClient require
     SSLVerifyDepth  10 
 </VirtualHost>
```

The conf/server.crt file is the PEM-encoded Certificate file for the
VirtualHost it must be signed by a Certificate Authority (CA) whose
certificate is stored in the sslTrustStore of the ClusterListener
parameter.

The conf/server.key file is the file containing the private key.

The conf/server-ca.crt file is the file containing the certicate of the
CA that have signed the client certificate JBossWEB is using. That is
the CA that have signed the certificate corresponding to the sslKeyAlias
stored in the sslKeyStore of the ClusterListener parameters.

#### ClusterListener configuration part 

There is a [wiki](http://www.jboss.org/community/docs/DOC-9300)
describing the SSL parameters of the ClusterListener. See in one example
how easy the configuration is:

```
 <Listener className="org.jboss.web.cluster.ClusterListener"
           ssl="true"
           sslKeyStorePass="changeit"
           sslKeyStore="/home/jfclere/CERTS/CA/test.p12"
           sslKeyStoreType="PKCS12"
           sslTrustStore="/home/jfclere/CERTS/CA/ca.p12"
           sslTrustStoreType="PKCS12" sslTrustStorePassword="changeit"
           />
```

The sslKeyStore file contains the private key and the signed certificate
of the client certificate JBossWEB uses to connect to httpd. The
certificate must be signed by a Cerficate Authority (CA) who certificate
is in the conf/server-ca.crt file of the httpd

The sslTrustStore file contains the CA certificate of the CA that signed
the certificate contained in conf/server.crt file.

#### mod-cluster-jboss-beans configuration part 

The mod-cluster-jboss-beans.xml in
\$JBOSS_HOME/server/*profile*/deploy/mod-cluster.sar/META-INF in the
ClusterConfig you are using you should have something like:

```
      <property name="ssl">true</property>
      <property name="sslKeyStorePass">changeit</property>
      <property name="sslKeyStore">/home/jfclere/CERTS/test.p12</property>
      <property name="sslKeyStoreType">pkcs12</property>
      <property name="sslTrustStore">/home/jfclere/CERTS/ca.p12</property>
      <property name="sslTrustStoreType">pkcs12</property>
      <property name="sslTrustStorePassword">changeit</property>
```

##### How the diferent files were created 

The files were created using OpenSSL utilities see
[OpenSSL](http://www.openssl.org/) CA.pl (/etc/pki/tls/misc/CA for
example) has been used to create the test Certificate authority, the
certicate requests and private keys as well as signing the certicate
requests.

##### Create the CA 

1.  Create a work directory and work for there:

    ```
    mkdir -p CERTS/Server
    cd CERTS/Server
    ```

2.  Create a new CA:

    ```
    /etc/pki/tls/misc/CA -newca 
    ```

    That creates a directory for example ../../CA that contains a
    cacert.pem file which content have to be added to the
    conf/server-ca.crt described above.

3.  Export the CA certificate to a .p12 file:

    ```
    openssl pkcs12 -export -nokeys -in ../../CA/cacert.pem -out ca.p12
    ```

    That reads the file cacert.pem that was created in the previous step
    and convert it into a pkcs12 file the JVM is able to read.

    That is the ca.p12 file used in the *sslTrustStore* parameter above.

##### Create the server certificate 

1.  Create a new request:

    ```
    /etc/pki/tls/misc/CA -newreq 
    ```

    That creates 2 files named newreq.pem and newkey.pem. newkey.pem is
    the file conf/server.key described above.

2.  Sign the request:

    ```
     /etc/pki/tls/misc/CA -signreq 
    ```

    That creates a file named newcert.pem. newcert.pem is the file
    conf/server.crt described above. At that point you have created the
    SSL stuff needed for the VirtualHost in httpd. You should use a
    browser to test it after importing in the browser the content of the
    cacert.pem file.

##### Create the client certificate 

1.  Create a work directory and work for there:

    ```
    mkdir -p CERTS/Client
    cd CERTS/Client
    ```

2.  Create request and key for the JBossWEB part.

    ```
    /etc/pki/tls/misc/CA -newreq
    ```

    That creates 2 files: Request is in newreq.pem, private key is in
    newkey.pem

3.  Sign the request.

    ```
    /etc/pki/tls/misc/CA -signreq
    ```

    That creates a file: newcert.pem

4.  Don't use a passphrase when creating the client certicate or remove
    it before exporting:

    ```
    openssl rsa -in newkey.pem -out key.txt.pem
    mv key.txt.pem newkey.pem
    ```

5.  Export the client certificate and key into a p12 file.

    ```
    openssl pkcs12 -export -inkey newkey.pem -in newcert.pem -out test.p12
    ```

    That is the sslKeyStore file described above
    (/home/jfclere/CERTS/CA/test.p12)

#### Using SSL between httpd and JBossWEB

Using https allows to encrypt communications betwen httpd and JBossWEB.
But due to the ressources it needs that no advised to use it in high
load configuration.

(See [Encrypting connection between httpd and
TC](http://www.jboss.org/community/docs/DOC-9701) for detailed
instructions).

httpd is configured to be a client for AS/TC so it should provide a
certificate AS/TC will accept and have a private key to encrypt the
data, it also needs a CA certificate to valid the certificate AS/TC will
use for the connection.

```
SSLProxyEngine On
SSLProxyVerify require
SSLProxyCACertificateFile conf/cacert.pem
SSLProxyMachineCertificateFile conf/proxy.pem
```

conf/proxy.pem should contain both key and certificate. The certificate
must be trusted by Tomcat via the CA in truststoreFile of
\<connector/\>.

conf/cacert.pem must contain the certificat of the CA that signed the
AS/TC certificate. The correspond key and certificate are the pair
specificed by keyAlias and truststoreFile of the \<connector/\>. Of
course the \<connector/\> must be the https one (normally on port 8443).

##### How the diferent files were created 

The files were created using OpenSSL utilities see
[OpenSSL](http://www.openssl.org/) CA.pl (/etc/pki/tls/misc/CA for
example) has been used to create the test Certificate authority, the
certicate requests and private keys as well as signing the certicate
requests.

##### Create the CA 

(See [above](#createca "12.1.4.1. Create the CA"))

##### Create the server certificate 

(See [above](#createsc "12.1.4.2. Create the server certificate"))

The certificate and key need to be imported into the java keystore using
keytool

make sure you don't use a passphare for the key (don't forget to clean
the file when done)

1.  Convert the key and certificate to p12 file:

    ```
    openssl pkcs12 -export -inkey key.pem -in newcert.pem -out test.p12
    ```

    make sure you use the keystore password as Export passphrase.

2.  Import the contents of the p12 file in the keystore:

    ```
    keytool -importkeystore -srckeystore test.p12 -srcstoretype PKCS12
    ```

3.  Import the CA certificate in the java trustore: (Fedora13 example).

    ```
    keytool -import -trustcacerts -alias "caname" \
    -file  ../../CA/cacert.pem -keystore /etc/pki/java/cacerts
    ```

4.  Edit server.xml to have a \<connector/\> similar to:

    ```
    <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
               keyAlias="1" 
               truststoreFile="/etc/pki/java/cacerts"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="true" sslProtocol="TLS" />
    ```

5.  Start TC/AS and use openssl s_client to test the connection:

    ```
    openssl s_client -CAfile /home/jfclere/CA/cacert.pem -cert newcert.pem -key newkey.pem \
    -host localhost -port 8443
    ```

    There shouldn't be any error and you should be able to see your CA
    in the "Acceptable client certificate CA names".

#### Forwarding SSL browser informations when using http/https between httpd and JBossWEB

When using http or https beween httpd and JBossWEB you need to use the
SSLValve and export the SSL variable as header in the request in httpd.
If you are using AJP, mod_proxy_ajp will read the SSL variables and
forward them to JBossWEB automaticaly.

(See [Forwarding SSL environment when using http/https
proxy](http://www.jboss.org/community/docs/DOC-11988) for detailed
instructions).

The SSL variable used by mod_proxy_ajp are the following:

1.  "HTTPS" SSL indicateur.

2.  "SSL_CLIENT_CERT" Chain of client certificates.

3.  "SSL_CIPHER" Cipher used.

4.  "SSL_SESSION_ID" Id of the session.

5.  "SSL_CIPHER_USEKEYSIZE" Size of the key used.

## Migration from mod_jk 

Mod_cluster only support Apache httpd, there are no plan to support IIS
or IPlanet.

The migration from mod_jk to mod_cluster is not very complex. Only
very few worker properties can't be mapped to mod_cluster parameters.

Here is the table of worker properties and how to transfer them in the
ClusterListener parameters.

| mod_jk worker property | ClusterListener parameter      | Remarks
|:-----------------------|:-------------------------------|:-------------------------------------------------------|
|host                    |    -                           | It is read from the \<Connector/\> Address information |
|---
|port                    |    -                           | It is read from the \<Connector/\> Port information |
|---
|type                    |    -                           | It is read from the \<Connector/\> Protocol information |
|---
|route                   |    -                           | It is read from the \<Engine/\> JVMRoute information |
|---
|domain                  |    domain                      | That is not supported in this version |
|---
|redirect                |    -                           | The nodes with loadfactor = 0 are standby nodes they will be used no other nodes are available |
|---
|socket_timeout          |   nodeTimeout                  | Default 10 seconds |
|---
|socket_keepalive        |   -                            | KEEP_ALIVE os is always on in mod_cluster |
|---
|connection_pool_size    |  -                             | The max size is calculated to be AP_MPMQ_MAX_THREADS+1 (max) |
|---
|connection_pool_minsize |  smax                          | The defaut is max |
|---
|connection_pool_timeout |  ttl                           | Time to live when over smax connections. The defaut is 60 seconds |
|---
|-                       |    workerTimeout               | Max time to wait for a free worker default 1 second |
|---
|retries                 |    maxAttempts                 | Max retries before returning an error Default: 3 |
|---
|recovery_options        |   -                            | mod_cluster behave like mod_jk with value 7 |
|---
|fail_on_status          |  -                             | Not supported |
|---
|max_packet_size         |  iobuffersize/receivebuffersize| Not supported in this version. Use ProxyIOBufferSize |
|---
|max_reply_timeouts      |  -                             | Not supported |
|---
|recovert_time           |   -                            | The ClusterListener will tell (via a STATUS message) mod_cluster that the node is up again |
|---
|activation              |    -                           | mod_cluster receives this information via ENABLE/DISABLE/STOP messages |
|---
|distance                |    -                           | mod_cluster handles this via the loadfactor logic |
|---
|mount                   |    -                           | The context "mounted" automaticly via the ENABLE-APP messages. ProxyPass could be used too |
|---
|secret                  |    -                           | Not supported |
|---
|connect_timeout         |   -                            | Not supported. Use ProxyTimeout or server TimeOut (Default 300 seconds) |
|---
|prepost_timeout         |   ping                         | Default 10 seconds |
|---
|reply_timeout           |   -                            | Not supported. Use ProxyTimeout or server TimeOut? directive (Default 300 seconds) |
{: rules="groups"}

## Migration from mod_proxy

As mod_cluster is a sophisticated balancer migration from mod_proxy to
mod_cluster is strait forward. mod_cluster replaces a reverse proxy
with loadbalancing. A reversed proxy is configured like:

```
ProxyRequests Off
<Proxy *>
Order deny,allow
Allow from all
</Proxy>
ProxyPass /foo http://foo.example.com/bar
ProxyPassReverse /foo http://foo.example.com/bar
```

All the general proxy parameters could be used in mod_cluster they work
like in mod_proxy, only the balancers and the workers definitions are
slightly different.

### Workers

| Mod_proxy Parameter | ClusterListener parameter  | Remarks  |
|:--------------------|:---------------------------|:------------------------------|
| min                 |  -                         | Not supported in this version  |
|---
| max                 |  -                         | mod_cluster uses mod_proxy default value  |
|---
| smax                |  smax                      | Same as mod_proxy  |
|---
| ttl                 |  ttl                       | Same as mod_proxy  |
|---
| acquire             |  workerTimeout             | Same as mod_proxy acquire but in seconds  |
|---
| disablereuse        |  -                         | mod_cluster will disable the node in case of error and the ClusterListener will for the reuse via the STATUS message  |
|---
| flushPackets        |  flushPackets              | Same as mod_proxy  |
|---
| flushwait           |  flushwait                 | Same as mod_proxy  |
|---
| keepalive           |  -                         | Always on: OS KEEP_ALIVE is always used. Use connectionTimeout in the \<Connector\> if needed  |
|---
| lbset               |  -                         | Not supported  |
|---
| ping                |  ping                      | Same as mod_proxy Default value 10 seconds  |
|---
| lbfactor            |  -                         | The load factor is received by mod_cluster from a calculated value in the ClusterListener  |
|---
| redirect            |  -                         | Not supported lbfactor sent to 0 makes a standby node  |
|---
| retry               |  -                         | ClusterListener will test when the node is back online  |
|---
| route               |  JVMRoute                  | In fact JBossWEB via the JVMRoute in the Engine will add it  |
|---
| status              |  -                         | mod_cluster has a finer status handling: by context via the ENABLE/STOP/DISABLE/REMOVE application messages. hot-standby is done by lbfactor = 0 and Error by lbfactor = 1 both values are sent in STATUS message by the ClusterListener  |
|---
| timeout             |  nodeTimeout               | Default wait for ever (http://httpd.apache.org/docs/2.2/mod/mod_proxy.html is wrong there)  |
|---
| ttl                 |  ttl                       | Default 60 seconds  |
{: rules="groups"}

### Balancers

| Mod_proxy Parameter | ClusterListener parameter             | Remarks  |
|:--------------------|:--------------------------------------|:---------|
| lbmethod            | -                                     | There is only one load balancing method in mod_cluster "cluster_byrequests"  |
| maxattempts         | maxAttempts                           | Default 1  |
| nofailover          | stickySessionForce                    | Same as in mod_proxy  |
| stickysession       | StickySessionCookie/StickySessionPath | The 2 parameters in the ClusterListener are combined in one that behaves like in mod_proxy  |
| timeout             | workerTimeout                         | Default 1 seconds  |
{: rules="groups"}
