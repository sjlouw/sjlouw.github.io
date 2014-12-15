---
layout: post
title: "Syslog with Cisco devices"
date: 2010-04-24
comments: true
categories: cisco
---
{% img left /images/blog_posts/cisco5.png %}

Configuring a Cisco router to log to a central Syslog server. The options include configuring the local buffer size, log levels and the transport protocol being used, UDP or TCP.
<!--more-->
<br>
<br>
<br>

##Setting local buffer size:

{% codeblock %}
Router(config)#logging buffered 4096
{% endcodeblock %}

##Setting the logging level for the local console:

{% codeblock %}
Router(config)#logging console informational
{% endcodeblock %}

##Setting the logging level for remote syslog:

{% codeblock %}
Router(config)#logging trap <severity level>

Router(config)#logging trap ?
  <0-7>          Logging severity level
  alerts         Immediate action needed           (severity=1)
  critical       Critical conditions               (severity=2)
  debugging      Debugging messages                (severity=7)
  emergencies    System is unusable                (severity=0)
  errors         Error conditions                  (severity=3)
  informational  Informational messages            (severity=6)
  notifications  Normal but significant conditions (severity=5)
  warnings       Warning conditions                (severity=4)
{% endcodeblock %}

##Configuring the syslog messages to include the source device name:

{% codeblock %}
Router(config)#logging origin-id hostname
{% endcodeblock %}

##Configuring the logging transport protocol:

**Setting transport protocol to UDP (Default):**
{% codeblock %}
Router(config)#logging host <syslog_server_ip>
{% endcodeblock %}

or

**Setting transport protocol to TCP (Only supported in later IOS versions):**
{% codeblock %}
Router(config)#logging host <syslog_server_ip> transport tcp port 514
{% endcodeblock %}
