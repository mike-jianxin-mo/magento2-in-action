---
title: CoudWatch Setup (2)
author: Mike Mo
date: 2021-10-16
category: [AWS, Data]
layout: post
---

##### Great AWS Knowledge Source: [1]

#### Apache commands

sudo apachectl configtest

sudo service apache2 reload

sudo tail -f /var/log/apache2/access.log

##### JSON format apache log

###### Current Log:

LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined

# add the forward ip address to log message

LogFormat "%{X-Forwarded-For}i %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined

# LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined

LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

###### JSON format

Replace the current log shown above to the following:

LogFormat "{ \"time\":\"%{%Y-%m-%d}tT%{%T}t.%{msec_frac}tZ\", \"forwardIP\":\"%{X-Forwarded-For}i\", \"remoteIP\":\"%a\", \"vhost\":\"%v:%p\", \"host\":\"%V\", \"request\":\"%U\", \"query\":\"%q\", \"method\":\"%m\",\"process\":\"%D\", \"status\":\"%>s\", \"userAgent\":\"%{User-agent}i\", \"referer\":\"%{Referer}i\" }" combined

#### Log details

Ref: https://www.loggly.com/ultimate-guide/apache-logging-basics/

LogFormat "%h %l %u %t \"%r\" %>s %b" common
CustomLog "logs/access_log" common

You can find a full list of fields in the Apache log documentation. We recommend using at least the following five fields, as they are important for monitoring server health and for troubleshooting issues:

-   %>s – The HTTP status code for the request. This shows the final request status after any internal redirection; for the original status, use %s.
-   %U – The URL path requested, excluding any additional URL parameters such as a query string.
-   %a – The IP address of the client making the request. This is useful for identifying traffic from a particular source.
-   %T – How long it took to process the request in seconds. This is useful for measuring the speed of your site. Use %D to make the same measurement in microseconds.
-   %{UNIQUE_ID}e – Also commonly known as the request ID, this logs a unique identifier with each request. This is useful for tracing a request from Apache to your web application server.

[1]: https://aws.amazon.com/blogs/mt/simplifying-apache-server-logs-with-amazon-cloudwatch-logs-insights/
