#Varnish Cache
Varnish is an HTTP accelerator and a useful tool for speeding up a server, especially during a times when there is high traffic to a site. It works by redirecting visitors to static pages whenever possible and only drawing on the virtual private server itself if there is a need for an active process.

#Table of contents
[TOC]

##How to Install
###Download and build
*Latest varnish version*: 4.1.3


----------


####Install using package-manager:

    apt-get install apt-transport-https
    curl https://repo.varnish-cache.org/GPG-key.txt | apt-key add -
    echo "deb https://repo.varnish-cache.org/ubuntu/ trusty varnish-4.1" \
     >> /etc/apt/sources.list.d/varnish-cache.list
     apt-get update
     apt-get install varnish
    
####Build from source:
Follow instructions on [varnish website](https://www.varnish-cache.org/docs/trunk/installation/install.html)
Dependencies:

    sudo apt-get install automake autotools-dev libedit-dev libjemalloc-dev libncurses5-dev libncurses5 libncurses-dev libpcre3-dev libtool pkg-config  autoconf python-docutils python-sphinx graphviz
   Download:
   `git clone https://github.com/varnishcache/varnish-cache.git`

Configure:

    cd varnish-cache
    sh autogen.sh
    sh configure
    make

Before you install, you may want to run the test suite, make a cup of tea while it runs, it usually takes a couple of minutes:

    make check
Install:

    sudo make install
Varnish will now be installed in `/usr/local`. The varnishd binary is in `/usr/local/sbin/varnishd`. To make sure that the necessary links and caches of the most recent shared libraries are found, run `sudo ldconfig`.

###Fix systemd scripts 
#### <i class="icon-attention"></i>only applicable to Debian (v8+) / Ubuntu (v15.04+)
If you have installed varnish through the package-manager, `sudo service varnish start` won't use `/etc/default/varnish` script to run, and will use a predefined set of configurations(port 6080, 256mb)

Try the [official docs](https://www.varnish-cache.org/docs/trunk/tutorial/putting_varnish_on_port_80.html).
For more detailed instructions follow [these](http://deshack.net/how-to-varnish-listen-port-80-systemd/).
###Varnish configuration
Use cafe_varnish.vcl as the default template. It has the following features:

 - Unset any cookies
 - sort / remove query strings (e.g. ?RANDOM=...)
 - use streaming mode to deliver (`set beresp.do_stream = true;`)
 - headers indicating hit count
 
 <i class="icon-attention"></i>Note:
 - You want to configure the back-ends. 
	 - [vcl-backends](https://www.varnish-cache.org/docs/trunk/users-guide/vcl-backends.html)
	 - [directors](https://www.varnish-cache.org/docs/trunk/reference/vmod_directors.generated.html) (can support fallback servers)
 - you might want to bypass very large files from the cache, and send requests directly to back-end servers.
 - You may want to change hash key.
 - You may want to remove cache-state headers in production.
###Start varnish
[Put](https://www.varnish-cache.org/docs/trunk/tutorial/putting_varnish_on_port_80.html) varnish on port 80	.
Edit the configuration file that starts Varnish. On older Debian/Ubuntu this is `/etc/default/varnish`. 

    DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
On Debian (v8+) / Ubuntu (v15.04+) [fix systemd scripts](#fix-systemd-scripts).

Manual start:

    sudo varnishd -a :80 -T localhost:6082 -f /etc/varnish/default.vcl -S /etc/varnish/secret -s malloc,256m

###Tuning
Read [this](http://book.varnish-software.com/4.0/chapters/Tuning.html).

> Note If you run across the tuning advice that suggests to have a thread pool per CPU core, rest assured that this is old advice. We recommend to have at most 2 thread pools, but you may increase the number of threads per pool.

 - at least 100 [threads] * 2 [pools] worker threads at any given time
 - no more than 5000 [threads] * 2 [pools] worker threads ever

<i class="icon-attention"></i> Remember to monitor syslog 

> The varnish architecture ensures that even if Varnish does contain a critical bug, it will start back up again fast. Usually within a few seconds, depending on the conditions.
> 
> All of this is logged to syslog. This makes it crucially important to monitor the syslog, otherwise you may never even know unless you look for them, because the perceived downtime is so short.

###Monitoring

> Quick hints:
> check if varnish process is up: `ps aux | grep varnish`
> check if varnish is listening on port 80: `sudo netstat -plnt`
####Varnish tools
 - Controlling a running Varnish instance is accomplished with the `varnishadm` tool, which talks to the management process through the CLI interface.
 - `varnishstat` is the simplest, yet one of the most useful log-related tools. 
 - `varnishlog` is a simple way to view and work with the rest of the shmlog.
*Options: backend-traffic (-b), client-traffic (-c) or everything.*
 - The `varnishtop` utility reads varnishd(1) shared memory logs and
presents a continuously updated list of the most commonly occurring
log entries.
 - The `varnishhist` tool can draw a histogram of response time distribution, size distribution or any other number-based tag. It can make for an interesting demo, but is not particularly useful unless you have very specific questions that you need answered
####Varnish dashboard
[github](https://github.com/brandonwamboldt/varnish-dashboard) (based on [varnish-agent](https://github.com/varnish/vagent2))
####Munin plugins:
<font color=red> *Untested*: </font> 
[Munin Plugin for Varnish 4](https://github.com/munin-monitoring/contrib/tree/master/plugins/varnish4)
[Varnish plugins for munin monitoring](https://github.com/basiszwo/munin-varnish)

###VMODS
Varnish supports a host of different extensions found [here](https://varnish-cache.org/vmods/).
The [Varnish Security Firewall](https://github.com/comotion/VSF) is a Web Application Firewall written using the Varnish Control Language and a varnish module.

> Written by [Ehsan Hajyasini](mailto:ehsan@cafebazaar.ir)
> Written with [StackEdit](https://stackedit.io/).
