---
layout: page
title: Change log for all releases
modified: 2015-12-26T16:44:17+01:00
excerpt: "A list of all mod_cluster releases"
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

**This site is work in progress** {{ site.time | date: '%Y-%m-%d' }}: Please, refer to [http://mod-cluster.jboss.org/](http://mod-cluster.jboss.org/) until the content here is complete.
{: .notice}

---

The following log contains the most noteworthy fixed issues and changes made in the particular release.

## 1.3.1.Final (6 May 2015)
<a href="{{ site.owner.downloads }}/tag/1.3.1.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [MODCLUSTER-344](https://issues.jboss.org/browse/MODCLUSTER-344) Pings for HTTP/HTTPS connections are now sent by default, can be disabled by "EnableOptions Off"

## 1.3.0.Final (06 February 2014)
<a href="{{ site.owner.downloads }}/tag/1.3.0.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [MODCLUSTER-288](https://issues.jboss.org/browse/MODCLUSTER-288) Drop SystemMemoryUsageLoadMetric altogether

## 1.2.6.Final (07 September 2013) 
<a href="{{ site.owner.downloads }}/tag/1.2.6.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [MODCLUSTER-360](https://issues.jboss.org/browse/MODCLUSTER-360) mod_proxy_cluster.h doesn't contain the right information

## 1.2.5.Final (30 August 2013)
<a href="{{ site.owner.downloads }}/tag/1.2.5.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [MODCLUSTER-334](https://issues.jboss.org/browse/MODCLUSTER-334) mod_cluster core when use ProxyPass / balancer://bal and CreateBalancers 1
* [MODCLUSTER-335](https://issues.jboss.org/browse/MODCLUSTER-335) Regression in ProxyPass integration
* [MODCLUSTER-339](https://issues.jboss.org/browse/MODCLUSTER-339) "proxy: DNS lookup failure" with IPv6 on Solaris
* [MODCLUSTER-345](https://issues.jboss.org/browse/MODCLUSTER-345) http_handle_cping_cpong with EnableOptions and HTTPS connector
* [MODCLUSTER-347](https://issues.jboss.org/browse/MODCLUSTER-347) mod_cluster-manager may break up aliases from a single VirtualHost, causing a messy page
* [MODCLUSTER-332](https://issues.jboss.org/browse/MODCLUSTER-332) Add Engine.getExecutor() to container SPI
* HTTP httpd-2.4.x doesn't like the extra CRLF.
* [MODCLUSTER-349](https://issues.jboss.org/browse/MODCLUSTER-349) mod_manager truncates ENABLE-APP messages.
* Fix ap_proxy_make_fake_req in 2.4.x

## 1.2.4.Final (24 April 2013)
<a href="{{ site.owner.downloads }}/tag/1.2.4.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [MODCLUSTER-315](https://issues.jboss.org/browse/MODCLUSTER-315) Convert project to use i18n logging and exceptions
* [MODCLUSTER-328](https://issues.jboss.org/browse/MODCLUSTER-328) mod_cluster doesn't recognize ? as a proper context delimiter causing 503s on requests with query strings
* [JBPAPP-9493](https://issues.jboss.org/browse/JBPAPP-9493) mod_cluster returns "Bad Gateway" HTTP ErrorCode 502 with https
* [MODCLUSTER-326](https://issues.jboss.org/browse/MODCLUSTER-326) rename NMAKEmakefile to NMAKEmakefile.example

## 1.2.3.Final (18 October 2012)
<a href="{{ site.owner.downloads }}/tag/1.2.3.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [JBPAPP-9788](https://issues.jboss.org/browse/JBPAPP-9788) Tomcat with mod_cluster refuses to shut down
* [JBPAPP-10168](https://issues.jboss.org/browse/JBPAPP-10168) Option sessionDrainingStrategy in mod_cluster listener in tomcat causes an exception
* [JBPAPP-10147](https://issues.jboss.org/browse/JBPAPP-10147) The shutdown port of tomcat is released before performing actual shut down

## 1.2.2.Final (06 September 2012)
<a href="{{ site.owner.downloads }}/tag/1.2.2.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [JBPAPP-9493](https://issues.jboss.org/browse/JBPAPP-9493) mod_cluster returns "Bad Gateway" HTTP ErrorCode 502 with https
* [JBPAPP-9791](https://issues.jboss.org/browse/JBPAPP-9791) mod_rewrite + mod_cluster: one kind of "proxying" RewriteRules don't work on Windows
* [MODCLUSTER-224](https://issues.jboss.org/browse/MODCLUSTER-224) when using tomcat manager webapp to stop/start an application the start is ignored by mod_cluster
* [MODCLUSTER-322](https://issues.jboss.org/browse/MODCLUSTER-322) Using AverageSystemLoadMetric can improperly cause a Load Factor of 0
* [MODCLUSTER-312](https://issues.jboss.org/browse/MODCLUSTER-312) port [JBPAPP-8956](https://issues.jboss.org/browse/JBPAPP-8956) to 1.2.x
* [MODCLUSTER-311](https://issues.jboss.org/browse/MODCLUSTER-311) mod_manager doesn't handle multiple virtualhosts per node
* [MODCLUSTER-309](https://issues.jboss.org/browse/MODCLUSTER-309) mod_proxy_cluster not checking all available balancers (but only the first one) for an available route
* [JBPAPP-8957](https://issues.jboss.org/browse/JBPAPP-8957) Segmentation Faut when you run two or more JBoss Server Cluster behind \*ONE\* Apache Server on RHEL 6

## 1.2.1.Final (03 May 2012)
<a href="{{ site.owner.downloads }}/tag/1.2.1.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* [MODCLUSTER-304](https://issues.jboss.org/browse/MODCLUSTER-304) Port to httpd 2.4
* [MODCLUSTER-305](https://issues.jboss.org/browse/MODCLUSTER-305) ProxyPass can break StickySession
* [MODCLUSTER-297](https://issues.jboss.org/browse/MODCLUSTER-297) Warnings when compiling mod_cluster
* [MODCLUSTER-192](https://issues.jboss.org/browse/MODCLUSTER-192) Add SSL stuff in the FAQ
* [MODCLUSTER-298](https://issues.jboss.org/browse/MODCLUSTER-298) mod_cluster doesn't work with IPv6.
* [MODCLUSTER-289](https://issues.jboss.org/browse/MODCLUSTER-289) MemManagerFile creates directory but put files in ..
* [MODCLUSTER-290](https://issues.jboss.org/browse/MODCLUSTER-290) mod_cluster's mod_advertise can not start on IPv6-only box
* [MODCLUSTER-279](https://issues.jboss.org/browse/MODCLUSTER-279) mod_cluster returns 503s after STATUS
* [MODCLUSTER-281](https://issues.jboss.org/browse/MODCLUSTER-281) Make stickySessionForce default to false in native part from 1.2.0 release

## 1.2.0.Final (06 February 2012)
<a href="{{ site.owner.downloads }}/tag/1.2.0.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Fix CVE-2011-4608: Add EnableMCPMReceive in configuration. ([JBPAPP-7708](https://issues.jboss.org/browse/JBPAPP-7708))
* mod_proxy code for old httpd versions needs to be updated. ([MODCLUSTER-227](https://issues.jboss.org/browse/MODCLUSTER-227))
* Missing Documentation For mod_cluster with JBoss AS5. ([MODCLUSTER-248](https://issues.jboss.org/browse/MODCLUSTER-248))
* sticky-session-force change requires httpd restart. ([MODCLUSTER-273](https://issues.jboss.org/browse/MODCLUSTER-273))
* mod_cluster management console/Status does not show exact mod_cluster version. ([MODCLUSTER-256](https://issues.jboss.org/browse/MODCLUSTER-256))

## 1.2.0.Beta4 (26 January 2012)
<a href="{{ site.owner.downloads }}/tag/1.2.0.Beta4" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* read-attribute and :write-attribute Error in AS7. ([MODCLUSTER-246](https://issues.jboss.org/browse/MODCLUSTER-246))
* routing table lookup creates performance issues.. ([MODCLUSTER-252](https://issues.jboss.org/browse/MODCLUSTER-252))
* Error 503 using mod_cluster: CVE-2011-3348. ([MODCLUSTER-258](https://issues.jboss.org/browse/MODCLUSTER-258))
* warning while compiling on some lab boxes. ([MODCLUSTER-266](https://issues.jboss.org/browse/MODCLUSTER-266))
* String index out of range: -1 with a lot of nodes/deploymentswarning while compiling on some lab boxes. ([MODCLUSTER-266](https://issues.jboss.org/browse/MODCLUSTER-266))
* 404 while/just after removing a node. ([MODCLUSTER-272](https://issues.jboss.org/browse/MODCLUSTER-272))
* ProxyPass doesn't work with mod_cluster whne mixing mod_cluster and other scheme handlers404 while/just after removing a node. ([MODCLUSTER-274](https://issues.jboss.org/browse/MODCLUSTER-274))
* mod_cluster forgets removed node too fast. ([MODCLUSTER-231](https://issues.jboss.org/browse/MODCLUSTER-231))
* Allow context lengths greater than 40 ([MODCLUSTER-236](https://issues.jboss.org/browse/MODCLUSTER-236))
* Increase the MAXMESSSIZE and allow it to be configurable via a parameter. ([MODCLUSTER-244](https://issues.jboss.org/browse/MODCLUSTER-244))
* Calculate the MAXMESSSIZE dynamically. ([MODCLUSTER-245](https://issues.jboss.org/browse/MODCLUSTER-245))

## 1.2.0.Beta3 (03 January 2012)
<a href="{{ site.owner.downloads }}/tag/1.2.0.Beta3" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

Code reorganisation

## 1.2.0.Beta2 (11 December 2011)
<a href="{{ site.owner.downloads }}/tag/1.2.0.Beta2" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Load-demo application does not work with AS7. ([MODCLUSTER-259](https://issues.jboss.org/browse/MODCLUSTER-259))

## 1.2.0.Beta1 (06 December 2011)
<a href="{{ site.owner.downloads }}/tag/1.2.0.Beta1" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Modify jbossweb metrics to use service provider spi, instead of jmx. ([MODCLUSTER-151](https://issues.jboss.org/browse/MODCLUSTER-151))
* mod_cluster throws warnings at shutdown in AS7. ([MODCLUSTER-143](https://issues.jboss.org/browse/MODCLUSTER-143))
* Refactor mod_cluster to allow implementations for other application servers. ([MODCLUSTER-105](https://issues.jboss.org/browse/MODCLUSTER-105))
* AS7 integration. ([MODCLUSTER-219](https://issues.jboss.org/browse/MODCLUSTER-219))
* Refactor project into modules for HA, catalina, core, etc. ([MODCLUSTER-112](https://issues.jboss.org/browse/MODCLUSTER-112))

## 1.1.3.Final (12 August 2011)
<a href="{{ site.owner.downloads }}/tag/1.1.3.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* request hang with a node is stopped in EC2 ([MODCLUSTER-217](https://issues.jboss.org/browse/MODCLUSTER-217)) 
* Extra ENABLE-APP/REMOVE-APP event after failover ([MODCLUSTER-220](https://issues.jboss.org/browse/MODCLUSTER-220)) 
* mod_cluster does not work with Tomcat 7 due to API Change in Connector ([MODCLUSTER-240](https://issues.jboss.org/browse/MODCLUSTER-240)) 
* kill -HUP httpd process increase the number of open locks ([MODCLUSTER-241](https://issues.jboss.org/browse/MODCLUSTER-241)) 

## 1.1.2.Final (21 April 2011)
<a href="{{ site.owner.downloads }}/tag/1.1.2.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* mod_cluster failover does not work for a /webappcontext when the / root context exists ([MODCLUSTER-188](https://issues.jboss.org/browse/MODCLUSTER-188)) 
* UseAlias broken ([MODCLUSTER-212](https://issues.jboss.org/browse/MODCLUSTER-212)) 
* mod_rewrite PT doesn't work ([MODCLUSTER-213](https://issues.jboss.org/browse/MODCLUSTER-213)) 
* memory usage growning in httpd ([MODCLUSTER-214](https://issues.jboss.org/browse/MODCLUSTER-214)) 
* Second attempt to connect from Jboss to apache module sends incomplete Host Header ([MODCLUSTER-216](https://issues.jboss.org/browse/MODCLUSTER-216)) 
* when using tomcat manager webapp to stop/start an application the start is ignored by mod_cluster ([MODCLUSTER-224](https://issues.jboss.org/browse/MODCLUSTER-224)) 
* Unable to override load properties when running in Tomcat 6 ([MODCLUSTER-232](https://issues.jboss.org/browse/MODCLUSTER-232)) 
* Documentation contains wrong default loadMetric class name ([MODCLUSTER-233](https://issues.jboss.org/browse/MODCLUSTER-233)) 
* mod_cluster should issue a warning when Maxcontext is reached and no more context will be taken ([MODCLUSTER-223](https://issues.jboss.org/browse/MODCLUSTER-223)) 
* add a note explaining where Maxcontext must be put in httpd.conf and issue error if in wrong location ([MODCLUSTER-225](https://issues.jboss.org/browse/MODCLUSTER-225)) 

## 1.1.1.Final (31 January 2011)
<a href="{{ site.owner.downloads }}/tag/1.1.1.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* NPE when overriding default load metric in ModClusterListener ([MODCLUSTER-183](https://issues.jboss.org/browse/MODCLUSTER-183)) 
* mod_cluster failover does not work for a /webappcontext when the / root context exists ([MODCLUSTER-188](https://issues.jboss.org/browse/MODCLUSTER-188)) 
* mod_cluster issues an ENABLE-APP too early in the webapp lifecycle ([MODCLUSTER-190](https://issues.jboss.org/browse/MODCLUSTER-190)) 
* mod_cluster 1.1.0 docs step 11.1.4.3 is wrong ([MODCLUSTER-193](https://issues.jboss.org/browse/MODCLUSTER-193)) 
* Incorrect routing of requests when one context root is the prefix of another ([MODCLUSTER-196](https://issues.jboss.org/browse/MODCLUSTER-196)) 
* Can only rewrite from the root context in httpd if there is a root context deployed in JBoss ([MODCLUSTER-198](https://issues.jboss.org/browse/MODCLUSTER-198)) 
* the STATUS MCMP message is send before the connector is started ([MODCLUSTER-202](https://issues.jboss.org/browse/MODCLUSTER-202)) 
* The windoze bundle don't have the default configuration ([MODCLUSTER-205](https://issues.jboss.org/browse/MODCLUSTER-205)) 
* httpd cores after graceful restart after the mod_cluster configuration is added ([MODCLUSTER-206](https://issues.jboss.org/browse/MODCLUSTER-206)) 
* Make logging on Tomcat use JDK14LoggerPlugin by default ([MODCLUSTER-185](https://issues.jboss.org/browse/MODCLUSTER-185)) 
* update mod_cluster to use HTTP/1.1 ([MODCLUSTER-201](https://issues.jboss.org/browse/MODCLUSTER-201)) 
* Add support for system properties used in 1.0.x ([MODCLUSTER-207](https://issues.jboss.org/browse/MODCLUSTER-207)) 

## 1.1.0.Final (13 August 2010)
<a href="{{ site.owner.downloads }}/tag/1.1.0.Final" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Demo servlets throw InstanceNotFoundException against EAP5 ([MODCLUSTER-170](https://issues.jboss.org/browse/MODCLUSTER-170)) 
* mod_advertise: Invalid ServerAdvertise Address too often ([MODCLUSTER-172](https://issues.jboss.org/browse/MODCLUSTER-172)) 
* Fix class name of MBeanAttributeRatioLoadMetric in MC config ([MODCLUSTER-174](https://issues.jboss.org/browse/MODCLUSTER-174)) 
* Wrong configuration could cause an httpd core ([MODCLUSTER-175](https://issues.jboss.org/browse/MODCLUSTER-175)) 
* Advertise not configured error message in log is actually a warning ([MODCLUSTER-176](https://issues.jboss.org/browse/MODCLUSTER-176)) 
* Better no servers connected message ([MODCLUSTER-165](https://issues.jboss.org/browse/MODCLUSTER-165)) 
* How does the UI deal with 20 or more servers ([MODCLUSTER-165](https://issues.jboss.org/browse/MODCLUSTER-165)) 
* mod_cluster should use hostname provided in address instead a IP address ([MODCLUSTER-168](https://issues.jboss.org/browse/MODCLUSTER-168)) 
* Read only view of mod_cluster-manager ([MODCLUSTER-181](https://issues.jboss.org/browse/MODCLUSTER-181)) 
* Demo client throws Bus Error when run with JDK 1.6 on OSX ([MODCLUSTER-169](https://issues.jboss.org/browse/MODCLUSTER-169)) 
* Use versioned docs ([MODCLUSTER-141](https://issues.jboss.org/browse/MODCLUSTER-141))
* Deprecate use of term "domain" ([MODCLUSTER-177](https://issues.jboss.org/browse/MODCLUSTER-177))

## 1.1.0.CR3 (15 June 2010)
<a href="{{ site.owner.downloads }}/tag/1.1.0.CR3" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Rpc failure can lead to failure to deploy a webapp ([MODCLUSTER-140](https://issues.jboss.org/browse/MODCLUSTER-140)) 
* Quotes in jsessionId causing sticky sessions to fail ([MODCLUSTER-146](https://issues.jboss.org/browse/MODCLUSTER-146)) 
* ManagerBalancerName doesn't work ([MODCLUSTER-153](https://issues.jboss.org/browse/MODCLUSTER-153)) 
* Parsing of IPv6 loopback address fails ([MODCLUSTER-156](https://issues.jboss.org/browse/MODCLUSTER-156)) 
* SystemMemoryUsageLoadMetric returns wrong load metric ([MODCLUSTER-157](https://issues.jboss.org/browse/MODCLUSTER-157)) 
* Clean shutdown logic can still inadvertently kill requests for non-distributed contexts. ([MODCLUSTER-159](https://issues.jboss.org/browse/MODCLUSTER-159)) 
* NoClassDefFoundError running demo app against AS6 ([MODCLUSTER-161](https://issues.jboss.org/browse/MODCLUSTER-161)) 
* Allow override of default clean shutdown behavior ([MODCLUSTER-139](https://issues.jboss.org/browse/MODCLUSTER-139)) 
* Avoid unnecessary open sockets for non-master nodes ([MODCLUSTER-158](https://issues.jboss.org/browse/MODCLUSTER-158)) 

## 1.1.0.CR2 (11 May 2010)
<a href="{{ site.owner.downloads }}/tag/1.1.0.CR3" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Add the lifecycle listener dynamically ([MODCLUSTER-20](https://issues.jboss.org/browse/MODCLUSTER-20)) 
* Use UUID for auto-generated jvmRoute ([MODCLUSTER-142](https://issues.jboss.org/browse/MODCLUSTER-142)) 
* improve packaging so that the bundle can run in \~ too ([MODCLUSTER-150](https://issues.jboss.org/browse/MODCLUSTER-150)) 
* jboss.mod_cluster.proxyList: invalid hosts cause mod-cluster startup to be delayed ([MODCLUSTER-155](https://issues.jboss.org/browse/MODCLUSTER-155)) 
* mod_cluster 1.1.0.CR1 doesn't work with Tomcat ([MODCLUSTER-143](https://issues.jboss.org/browse/MODCLUSTER-143)) 
* Allow configuration of stopContextTimeout units ([MODCLUSTER-138](https://issues.jboss.org/browse/MODCLUSTER-138)) 
* Add a solaris10 64 bits sparc in the bundles ([MODCLUSTER-137](https://issues.jboss.org/browse/MODCLUSTER-137)) 
* INFO and mod_cluster_manager/ displays milliseconds and DUMP second ([MODCLUSTER-128](https://issues.jboss.org/browse/MODCLUSTER-128)) 
* Make mod_cluster manager tolerant to F5 page refresh when disabled context ([MODCLUSTER-124](https://issues.jboss.org/browse/MODCLUSTER-124)) 

## 1.1.0.CR1 (22 March 2010)
<a href="{{ site.owner.downloads }}/tag/1.1.0.CR1" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Update httpd to 2.2.25. ([MODCLUSTER-134](https://issues.jboss.org/browse/MODCLUSTER-134)) 
* Apache with mod_cluster refuses to start at first, but after 7 retries it starts up ([MODCLUSTER-120](https://issues.jboss.org/browse/MODCLUSTER-120)) 
* Add getLoad() to load metric mbean interface ([MODCLUSTER-130](https://issues.jboss.org/browse/MODCLUSTER-130)) 
* Disable "cnone" request parameter to ease remote invocation on mod_cluster-manager ([MODCLUSTER-127](https://issues.jboss.org/browse/MODCLUSTER-127)) 
* Microcontainer does not always choose the right constructor when creating ModClusterService ([MODCLUSTER-116](https://issues.jboss.org/browse/MODCLUSTER-116)) 
* Microcontainer does not choose the right constructor when creating RequestCountLoadMetric ([MODCLUSTER-126](https://issues.jboss.org/browse/MODCLUSTER-126)) 
* Mod_cluster does support more that 3 Alias in \<Host/\> ([MODCLUSTER-121](https://issues.jboss.org/browse/MODCLUSTER-121)) 
* Allow toggling of context auto-enable during mod_cluster startup. ([MODCLUSTER-125](https://issues.jboss.org/browse/MODCLUSTER-125)) 
* STATUS should retry the worker even if there was an error before. ([MODCLUSTER-133](https://issues.jboss.org/browse/MODCLUSTER-133)) 
* Split ModClusterServiceMBean.ping(String) into 3 methods ([MODCLUSTER-110](https://issues.jboss.org/browse/MODCLUSTER-110))
* Use clean shutdown by default, leveraging STOP-APP-RSP for \<distributable/\> contexts and session draining for non-distributable contexts.
* mod_cluster shutdown now triggered earlier via Connector JMX notification.  ([MODCLUSTER-131](https://issues.jboss.org/browse/MODCLUSTER-131))
* move the web site to magnolia ([MODCLUSTER-114](https://issues.jboss.org/browse/MODCLUSTER-114)) (.org team)
* ping and nodeTimeout interact. ([MODCLUSTER-132](https://issues.jboss.org/browse/MODCLUSTER-132)) 
* update mod_jk to 1.2.30  ([MODCLUSTER-138](https://issues.jboss.org/browse/MODCLUSTER-138)) 
* query string is truncated to ([MODCLUSTER-118](https://issues.jboss.org/browse/MODCLUSTER-118)) 
* AdvertiseBindAddress does not default to the 23364 port ([MODCLUSTER-119](https://issues.jboss.org/browse/MODCLUSTER-119)) 
* Skip load balance factor calculation if there are no proxies to receive status message ([MODCLUSTER-103](https://issues.jboss.org/browse/MODCLUSTER-103)) 
* Disabling contexts does not work ([MODCLUSTER-123](https://issues.jboss.org/browse/MODCLUSTER-123)) 
* advertise doesn't use new AdvertiseSecurityKey on graceful restarts. ([MODCLUSTER-129](https://issues.jboss.org/browse/MODCLUSTER-129)) 
* Load-demo.war specifies obsolete servlet in web.xml  ([MODCLUSTER-113](https://issues.jboss.org/browse/MODCLUSTER-113)) 

## 1.1.0.Beta1 (30 October 2009)
<a href="{{ site.owner.downloads }}/tag/1.1.0.Beta1" target="_blank"><i class="fa fa-fw fa-arrow-circle-down"></i>Download archives</a>

* Interaction with mod_rewrite looks weird for end-users. ([MODCLUSTER-86](https://issues.jboss.org/browse/MODCLUSTER-86)) 
* admin-console should be in the excludedContexts. ([MODCLUSTER-87](https://issues.jboss.org/browse/MODCLUSTER-87)) 
* ClassCastException upon redeploy after mod-cluster-jboss-beans.xml modification. ([MODCLUSTER-88](https://issues.jboss.org/browse/MODCLUSTER-88)) 
* Alias from webapps/jboss-web.xml are not handled correctly in mod_cluster. ([MODCLUSTER-89](https://issues.jboss.org/browse/MODCLUSTER-89)) 
* Display version. ([MODCLUSTER-90](https://issues.jboss.org/browse/MODCLUSTER-90)) 
* Connector bind address of 0.0.0.0 propagated to proxy. ([MODCLUSTER-91](https://issues.jboss.org/browse/MODCLUSTER-91)) 
* Display status of the worker. ([MODCLUSTER-92](https://issues.jboss.org/browse/MODCLUSTER-92)) 
* Update httpd to lastest version. ([MODCLUSTER-93](https://issues.jboss.org/browse/MODCLUSTER-93)) 
* getProxyInfo failed when there are too many nodes. ([MODCLUSTER-94](https://issues.jboss.org/browse/MODCLUSTER-94)) 
* mod_cluster-manager display corrupted with jboss starting. ([MODCLUSTER-95](https://issues.jboss.org/browse/MODCLUSTER-95)) 
* DISABLE application active as STOPPED. ([MODCLUSTER-96](https://issues.jboss.org/browse/MODCLUSTER-96)) 
* Httpd should remove workers it can't ping. ([MODCLUSTER-97](https://issues.jboss.org/browse/MODCLUSTER-97)) 
* Linux mod_cluster_manager display zero instead values. ([MODCLUSTER-98](https://issues.jboss.org/browse/MODCLUSTER-98)) 
* mod_cluster_manager doesn't seem to ENABLE/DISABLE the right context. ([MODCLUSTER-99](https://issues.jboss.org/browse/MODCLUSTER-99)) 
* load balancing logic doesn't allow manual demo of load-balancing. ([MODCLUSTER-100](https://issues.jboss.org/browse/MODCLUSTER-100)) 
* 404 errors when load is increasing. ([MODCLUSTER-102](https://issues.jboss.org/browse/MODCLUSTER-102)) 
* Advertise security key verification does not work. ([MODCLUSTER-104](https://issues.jboss.org/browse/MODCLUSTER-104)) 
* Allow advertise listener to listen on a specific network interface. ([MODCLUSTER-106](https://issues.jboss.org/browse/MODCLUSTER-106)) 
* Allow thread factory injection for advertise listener. ([MODCLUSTER-108](https://issues.jboss.org/browse/MODCLUSTER-108)) 
* Create SPI and isolate tomcat/jbossweb usage into service provider implementation. ([MODCLUSTER-111](https://issues.jboss.org/browse/MODCLUSTER-111)) 
