---
layout: post
title: Inject the host IP with Gradle
comments: true
---

*This article is a translation of an article I wrote in French and published on [Xebia blog](http://blog.xebia.fr/2015/05/05/injecter-lip-host-avec-gradle/).*

When developing an Android application, we often, if not always, have to make network calls. So we regularly need to launch ourselves the backend application on our computer to develop and test.

However, our IP address is unique to each developer and forced them to change their `build.gradle`. This change presents a risk since such modifications can be pushed on the Git repository and cause problems to the environment of other developers. Therefore, how do we make our Android app directly know our backend and therefore our local IP address?

Even better: is it possible to handle this directly on the build? This is what we will see in this short article.

<!-- more -->

## What do we find by looking at the documentation ?

First (good) reflex, look in the JDK. We find the method `InetAddress.getLocalHost()` which retrieves the local IP address of the host. However, if we keep looking, we realize that this method presents some ambiguities on UNIX systems and  127.0.0.1 is often returned.

## What do we find if we ask Google ?

With a quick Google search, we find a [Dominic Bartl article](http://bartinger.at/inject-dynamic-host-ip-address-with-gradle/). His solution is to give, at the time of the build, the connected network interface in order to retrieve the IP address, ensuring it is not in IPv6 format. Its code is as follows:

{% highlight groovy %}
// Get the ip address by interface name
def getLocalIp(String interfaceName) {
    NetworkInterface iface =  NetworkInterface.getByName(interfaceName);
    for (InterfaceAddress address : iface.getInterfaceAddresses()) {
        String ip =  address.getAddress().getHostAddress()
        if (ip.length() <= 15) {
            return ip;
        }
    }
}
{% endhighlight %}

This code is a very good first step but it has several problems:

 1. the network interface may vary from one computer to the other,

 2. the network interface varies depending on the connection type (Ethernet, WIFI, ...).

Therefore, each developer needs to configure the build and we fall back on the same problem as the manual change of the IP address.

## What should we do ?

The solution we have implemented is :

{% highlight groovy %}
def getIP() {
    InetAddress result = null;
    Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
    while (interfaces.hasMoreElements()) {
        Enumeration<InetAddress> addresses = interfaces.nextElement().getInetAddresses();
        while (addresses.hasMoreElements()) {
            InetAddress address = addresses.nextElement();
            if (!address.isLoopbackAddress()) {
                if (address.isSiteLocalAddress()) {
                    return address.getHostAddress();
                } else if (result == null) {
                    result = address;
                }
            }
        }
    }
    return (result != null ? result : InetAddress.getLocalHost()).getHostAddress();
}
{% endhighlight %}


It is heavily inspired by what we saw previously. We go through all the network interfaces and all of their IP addresses until we find a non-loopback IP address and local site. If no address is found, we will return the first non-loopback address encountered. Finally, as a last resort, we use `InetAddress.getLocalHost()` method of the JDK.

This solution was tested on different environments and also on the Genymotion simulator.

## How do we use it ?

To fully take advantage of Gradle and its good practices, we will declare our IP address (and therefore our server URL) directly into the `BuildConfig` :

{% highlight groovy %}
buildTypes {
  debug {
    buildConfigField 'String', 'URL_SERVER', '"http://${getIP()}"'
  }
  release {
    buildConfigField 'String', 'URL_SERVER', '"http://www.prod.com"'
  }
}
{% endhighlight %}

So now you have a quick and easy way to retrieve and give the IP address of your local backend to your Android application.
