---
layout: page
title: FAQ
modified: 2016-01-04T00:41:46+01:00
excerpt: "Frequently asked questions on mod_cluster"
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

## What is Advertise

Advertise allows autodiscovery of httpd proxies by the cluster nodes. It is done by sending multicast messages from httpd to the cluster.
The httpd specialized module: mod_advertise sends UDP message on a multicast group, both mod_advertise and the cluster listener joined the
multicast group and the cluster receives the messages.

Example of a mod_advertise message:

{% highlight text %}
HTTP/1.0 200 OK
Date: Wed, 08 Apr 2009 12:26:32 GMT
Sequence: 16
Digest: f2d5f806a53effa6c67973d2ddcdd233
Server: 1b60092e-76f3-49fd-9f99-a51c69c89e2d
X-Manager-Address: 127.0.0.1:6666
X-Manager-Url: /bla
X-Manager-Protocol: http
X-Manager-Host: 10.33.144.3
{% endhighlight %}

The X-Manager-Address header value is used by the cluster logic to send information about the cluster to the proxy.
It is the IP and port of the VirtualHost where mod_advertise is configured or URL parameter of the ServerAdvertise directive.

See [mod_advertise configuration]({{ site.url }}/documentation#modadvertise).

## What to do if I don't want to use Advertise (UDP multicast)

In the VirtualHost receiving the MCPM of httpd.conf don't use any
Advertise directive or set explicitly:

{% highlight apacheconf %}
ServerAdvertise Off
{% endhighlight %}

On the worker side, add the addresses and ports of the VirtualHost receiving MCMP (having ```EnableMCPMReceive```) to the ```proxyList```
property and set advertise to false, for example:

### JBoss AS 5

{% highlight xml %}
<property name="proxyList">10.33.144.3:6666,10.33.144.1:6666</property>
<property name="advertise">false</property>
{% endhighlight %}

in *jboss-as/server/production/deploy/mod-cluster.sar/META-INF/mod-cluster-jboss-beans.xml*.

### Tomcat 6/7/8 and JBossWeb

In ```server.xml```, change

{% highlight xml %}
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener"
  stickySession="true"
  stickySessionForce="false"
  stickySessionRemove="true"
/>
{% endhighlight %}
to
{% highlight xml %}
<Listener className="org.jboss.modcluster.container.catalina.standalone.ModClusterListener"
  stickySession="true"
  stickySessionForce="false"
  stickySessionRemove="true"
  advertise="false"
  proxyList="10.33.144.3:6666,10.33.144.1:6666"
/>
{% endhighlight %}

## It is not working, what should I do?

Please, at first, go through the following check-list. Set Apache HTTP Server's ```LogLevel debug``` in ```httpd.conf``` and
read the ```error_log```. If you get stuck, you are welcome to
{% if page.author %}
  {% assign author = site.data.authors[page.author] %}{% else %}{% assign author = site.owner %}
{% endif %}
{% if author.user-forum %}* <a href="{{ author.user-forum }}" target="_blank"><i class="fa fa-fw fa-commenting"></i>post on JBoss user forums</a>{% endif %}
{% if author.mailing-list %}* <a href="{{ author.mailing-list }}" target="_blank"><i class="fa fa-fw fa-send-o"></i>join JBoss mailing list</a>{% endif %}
{% if author.email %}* <a href="mailto:{{ author.email }}" target="_blank"><i class="fa fa-fw fa-envelope-o"></i>and drop us a line</a>{% endif %}

### There is no error in the error_log

That happens when Advertise is not working and no Proxy List is configured: The worker nodes do not get the from Apache HTTP Server.

TODO: Links to docs; explain terms.

1. Check that the modules are loaded and Advertise is started. In httpd.conf activate extended information display, add:
   
   {% highlight apacheconf %}
   AllowDisplay On
   {% endhighlight %}

   When accessing the mod_cluster_manager console you should get something like: TODO: Image.
   If not, go to the [Minimal Example]({{ site.url }}/documentation#minimal-example) and add the missing directive(s).

1. Check that Advertise message are received on the cluster node.
   A [small Java utility](https://github.com/modcluster/mod_cluster/blob/master/test/java/Advertize.java)
   could be used to check Advertise. It is in the mod_cluster repository and can be compiled using javac.
   The output should be something like:

   {% highlight text %}
   [jfclere@jfcpc java]$ java Advertize 224.0.1.105 23364
   ready waiting...
   received: HTTP/1.0 200 OK
   Date: Mon, 28 Jun 2010 07:30:31 GMT
   Sequence: 1
   Digest: df8a4321fa99e5098174634f2fe2f87c
   Server: 1403c3be-837a-4e76-85b1-9dfe5ddb4378
   X-Manager-Address: test.example.com:6666
   X-Manager-Url: /1403c3be-837a-4e76-85b1-9dfe5ddb4378
   X-Manager-Protocol: http
   X-Manager-Host: test.example.com
   {% endhighlight %}

1. If there are no Advertise messages, check the firewall.
   Advertise uses UDP port 23364 and multicast address 224.0.1.105 by default.
   Furthermore, **do not bind to localhost/127.0.0.1**, use a non-localhost IP address for your balancer &mdash; workers UDP advertisement test.

1. If you are unable to get the Advertisement to work, try it first without it, with a 
   [static configuration without UDP multicast]({{ site.url }}/faq#what-to-do-if-i-dont-want-to-use-advertise-udp-multicast).

### Error in server.log or catalina.out

1. ```IO error sending command```:
   Check the firewall and error_log, if there is nothing in the error_log then it
   is a firewall problem. If you have something like:

   {% highlight text %}
   INFO  [DefaultMCMPHandler] IO error sending command INFO to proxy jfcpc/10.33.144.3:8888
   {% endhighlight %}

   it means that the worker was unable to **contact the balancer**. Keep in mind that the communication is **bidirectional**.

   You can use telnet hostname/address port to check that it is OK, e.g.:

   {% highlight text %}
   [jfclere@jfcpc docs]$ telnet 10.33.144.3 8888
   Trying 10.33.144.3...
   Connected to jfcpc.
   Escape character is '^]'.
   GET /
   <html><body><h1>It works!</h1></body></html>Connection closed by foreign host.
   {% endhighlight %}

   Check that the address and port are the expected ones you may use
   ServerAdvertise directive in you mod_cluster httpd configuration:

   {% highlight apacheconf %}
   ServerAdvertise On http://localhost:6666
   {% endhighlight %}

### Error in error_log 

1. ```client denied by server configuration```:
    The directory in the VirtualHost is not allowed for the client. If you have something like:

    {% highlight text %}
    [error] [client 10.33.144.3] client denied by server configuration: /
    {% endhighlight %}

    You need to have something like the undermentioned authentication configured in the ```EnableMCPMReceive``` marked VirtualHost:

    {% highlight apacheconf %}
    # httpd 2.4.x
    <Location />
        Require ip 10.33.144.3
    </Location>
    {% endhighlight %}

    {% highlight apacheconf %}
    # httpd 2.2.x
    <Location />
        Order deny,allow
        Deny from all
        Allow from 10.33.144.3
    </Location>
    {% endhighlight %}


## I started mod_cluster and it looks like it's using only one of the workers

One must give the system some time, in matter of the amount of new sessions created, to settle and pick other nodes.
An example from an actual environment: You have 3 nodes with the following Load values:

    Node jboss-6,   Load: 20
    Node jboss-6-2, Load: 90
    Node jboss-6-3, Load:  1

Yes, this means that jboss-6-2 is almost not loaded at all whereas jboss-6-3 is desperately overloaded.
Now, I send 1001 requests, each representing a new session (the client is forgetting cookies). The distribution of the requests will be as follows:

    Node jboss-6   served 181 requests
    Node jboss-6-2 served 811 requests
    Node jboss-6-3 served   9 requests

So, generally, yes, the least loaded box received by far the greatest amount of requests, but it did not receive them all. Furthermore, and this concerns your case, for some time from the start, it was jboss-6 who was getting requests.

This whole magic is in place in order to prevent congestion.

## Keep seeing "HTTP/1.1 501 Method Not Implemented"

One needs to configure [EnableMCPMReceive]({{ site.url }}/documentation#enablemcpmreceive) in
the VirtualHost where you received the MCMP elements in the Apache httpd configuration.
Something like in the aforementioned [Minimal Example]({{ site.url }}/documentation#minimal-example).

## Redirect is not working (Tomcat, JBossWeb):

When using http/https instead of AJP, proxyname, proxyhost and redirect must be configured in the Tomcat Connector. Something like:

{% highlight xml %}
<Connector port="8080" protocol="HTTP/1.1" 
    connectionTimeout="20000"
    proxyName="httpd_host_name"
    proxyPort="8000"
    redirectPort="443"
/>
{% endhighlight %}


## I have more than one Tomcat/JBossWeb Connector

mod_cluster tries to use the first AJP connector configured. If there
is not any AJP connector, it uses the http or https that has the biggest
maxthreads value. That is ```maxThreads``` in Tomcat 6/7/8 and JBoss AS 5/6:

{% highlight xml %}
<Connector port="8080" protocol="HTTP/1.1" maxThreads="201" />
{% endhighlight %}

Or ```max-connections``` in JBoss AS7: (32 \* processor + 1 for native and 512 \* processor + 1 for JIO).

In Web subsystem:

{% highlight xml %}
<connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http" max-connections="513" />
{% endhighlight %}


## Chrome does not display /mod_cluster_manager page 

When using Chrome with mod_cluster_manager, the page is not displayed
and the following error is displayed instead:

{% highlight text %}
Error 312 (net::ERR_UNSAFE_PORT): Unknown error.
{% endhighlight %}

you can change the port of the VirtualHost to 7777 or any value chrome accepts or add:

{% highlight text %}
 --explicitly-allowed-ports=6666
{% endhighlight %}

to the start parameters of Chrome.

## Using mod_cluster and SELinux.

mod_cluster needs to open port and create shared memory and files, therefore some permissions have to be added,
you need to configure something like:

{% highlight text %}
policy_module(mod_cluster, 1.0)

require {
        type unconfined_java_t;
        type httpd_log_t;
        type httpd_t;
        type http_port_t;
        class udp_socket node_bind;
        class file write;
}

#============= httpd_t ==============

allow httpd_t httpd_log_t:file write;
corenet_tcp_bind_generic_port(httpd_t)
corenet_tcp_bind_soundd_port(httpd_t)
corenet_udp_bind_generic_port(httpd_t)
corenet_udp_bind_http_port(httpd_t)

#============= unconfined_java_t ==============
allow unconfined_java_t http_port_t:udp_socket node_bind;
{% endhighlight %}

Put the above in a file for example mod_cluster.te and generate the
mod_cluster.pp file (for example in Fedora 16):

{% highlight bash %}
[jfclere@jfcpc docs]$ make -f /usr/share/selinux/devel/Makefile
Compiling targeted mod_cluster module
/usr/bin/checkmodule:  loading policy configuration from tmp/mod_cluster.tmp
/usr/bin/checkmodule:  policy configuration loaded
/usr/bin/checkmodule:  writing binary representation (version 14) to tmp/mod_cluster.mod
Creating targeted mod_cluster.pp policy package
rm tmp/mod_cluster.mod.fc tmp/mod_cluster.mod
{% endhighlight %}


The mod_cluster.pp file should be proceeded by semodule as root:

{% highlight bash %}
root@jfcpc docs]# semodule -i mod_cluster.pp
[root@jfcpc docs]# 
{% endhighlight %}

Alternatively, one may use semanage and add ports and paths labels manually.

## System properties to modify behaviour

**org.jboss.modcluster.container.catalina.status-frequency** &mdash; *(default: 1)*
makes worker to send STATUS MCMP messages only 1/n periodic event.
The events occur every ```backgroundProcessorDelay``` *(default 10 seconds)*.
