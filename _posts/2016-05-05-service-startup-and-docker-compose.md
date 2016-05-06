---
layout: post
title: Service Startup and Docker Compose
date: '2016-05-05T12:00:00.000-08:00'
author: David Green
tags:
  - Docker
  - Docker Compose
modified_time: '2016-05-05T12:00:00.000-08:00'
original_url: https://www.tasktop.com/blog/service-startup-and-docker-compose/
comments: true
---

We make extensive use of Docker and Docker Compose at [Tasktop](http://tasktop.com) for building, testing and deploying our products.  One of the pain points that we run in to is services that fail to start when other services are not available - for example a web app that fails to start unless the database is already up and running.  Apparently [we're not the only ones](https://github.com/docker/compose/issues/374), so here's a bit of bash hackery to work around the problem:


    until nc -z -v -w5 $SERVICE_HOST $SERVICE_PORT;
    do
        echo "waiting for service to become available on $SERVICE_HOST:$SERVICE_PORT"
        sleep 1
    done

This hack uses netcat (nc) to wait for a socket accepting connections on the specified host and port.  The `-z` flag causes netcat to detect the listening socket without actually sending any data.

To make this work in a Docker container, you'll have to ensure that netcat is available:

    RUN apt-get update && \
        apt-get install -y \
            netcat && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/*

There are a few down sides to this approach, such as:

* the script will not terminate if the service isn't available - there should be some kind of hard failure if the dependent service doesn't show up after awhile
* this hack ends up adding complexity to each Dockerfile that uses it
* some dependent services are still not "ready" even though they listen on a port - those need a more sophisticated approach, something better than `sleep 30`

Despite the shortcomings, the approach works well enough to get us over the hump of service dependencies.  Somehow, Docker has helped to reinvigorate shell scripting.
