---
layout: post
title: ELK Stack for Improved Support
date: '2017-09-09T15:11:00.000-08:00'
author: David Green
tags:
  - ELK
  - Elasticsearch
  - Logstash
  - Kibana
modified_time: '2017-09-09T16:05:00.000-08:00'
comments: true
---

The ELK stack, composed of [Elasticsearch](https://www.elastic.co/products/elasticsearch), [Logstash](https://www.elastic.co/products/logstash) and [Kibana](https://www.elastic.co/products/kibana), is world-class dashboarding for real-time monitoring of server environments, enabling sophisticated analysis and troubleshooting.  Could we also leverage this great tooling to our advantage in situations where access to the server environment is an impossibility?  Recently while investigating a customer support case, I looked into whether or not we could create a repeatable process to enable analysis of log files provided as part of a support case.  Following are details of the approach that we came up with.

The requirements were pretty straight-forward:

* Enable analysis of multi-GB logs provided as part of a support request
* Use familiar, first-class tooling
* Zero-installation usage by anyone on the support or engineering team
* Zero maintenance (no infrastructure needed)

For this we chose [ELK](https://www.elastic.co/products) and [Docker Compose](https://docs.docker.com/compose/), with the idea that anyone could bring up and tear down an environment with very little effort.  Rather than monitor logs in real time however, we needed to pull in logs from a folder on the local machine.  For this we used [Filebeat](https://www.elastic.co/products/beats/filebeat).

This is the `docker-compose.yml` that we came up with:

    elk:
      image: sebp/elk
      ports:
        - "5601:5601"
        - "9200:9200"
        - "5044:5044"
      volumes:
        - ${PWD}/02-beats-input.conf:/etc/logstash/conf.d/02-beats-input.conf
        - ${PWD}/log:/mnt/log
    filebeat:
      image: docker.elastic.co/beats/filebeat:5.5.1
      links:
        - "elk:logstash"
      volumes:
        - ${PWD}/filebeat.yml:/usr/share/filebeat/filebeat.yml
        - ${PWD}/log:/mnt/log

This Docker Compose file brings up two containers: `elk`, which as you might have guessed runs Elasticsearch, Logstash and Kibana, and `filebeat`, a container for reading log files that feeds the elk container with data.

The filebbeat container is the most interesting one: it reads files from a local folder named `log` in the current directory of the Docker _host_ machine.  With the brilliance of `${PWD}` support in Docker Compose, all we have to do is move support log files into that folder!

The following `filebeat.yml` configuration is needed:

    filebeat.prospectors:
    - input_type: log
      paths:
        - /mnt/log/*
      include_lines: [".*? ERROR "]
      multiline.pattern: ^\s*\d\d\d\d-\d\d-\d\d \d\d:\d\d:\d\d,\d\d\d \[
      multiline.negate: true
      multiline.match: after

    processors:
    - add_cloud_metadata:

    output.logstash:
      # The Logstash hosts
      hosts: ["logstash:5044"]

This one is configured to handle multi-line log entries (including Java stack traces) where the initial line of each log entry starts with a timestmap.  The `multiline.pattern` above may need adjusting to suit your log files.

All that remains to get this working is the beats configuration, `02-beats-input.conf`, which uses a bit of filtering hackery to split up the unstructured log entries into structured data before it's added to Elasticsearch:

    input {
      beats {
        port => 5044
      }
    }
    filter {
      grok {
        match => {
          "message" => "\s*(?<entry_date>\d\d\d\d-\d\d-\d\d) (?<entry_time>\d\d:\d\d:\d\d),(?<entry_time_millis>\d\d\d) \[(?<thread_id>[^\]]+)\] (?<severity>[^\s]+) (?<category>[^\s]+) - (?:(?<error_code>CCRRTT-\d+(E|W)):\s+)?(?<message_text>.*)"
        }
      }
      mutate {
        add_field => {
          "entry_timestamp" => "%{entry_date}T%{entry_time}.%{entry_time_millis}Z"
        }
        remove_field => ["entry_date", "entry_time", "entry_time_millis"]
      }
      mutate {
        remove_field => ["message"]
      }
      mutate {
        add_field => {
          "message" => "%{message_text}"
        }
        remove_field => ["message_text"]
      }
      grok {
        match => {
          "message" => "\s*(?<message_summary>.*?) Cause Context:.*"
        }
      }
      grok {
        match => {
          "message_summary" => "\s*(?<message_first_sentence>.*?\.).*"
        }
      }
    }

After creating those files I ended up with the following:

    ./
    ./docker-compose.yml
    ./logs/
    ./02-beats-input.conf
    ./filebeat.yml

With a simple `docker-compose up`, I moved over 56GB of log files into the `logs` folder and grabbed coffee.  After a few minutes I was happily analyzing the situation using a Kibana dashboard:

<img alt="Kibbana Dashboard" src="/images/blog/2017-09/kibana-dashboard.png" class="border center" />

In this example, we can see a chart of error codes and distinct messages over time.

To make this process even smoother, we used [elasticdump](https://www.npmjs.com/package/elasticdump) to export our Kibana dashboards for other support cases.

To export dashboards:

    elasticdump --input=http://localhost:9200/.kibana --output=$ --type=data > kibana-settings.json

To import dashboards:

    elasticdump --input=./kibana-settings.json --output=http://localhost:9200/.kibana --type=data

Using ELK for post-mortem analysis of log files is a snap.  The approach outlined above makes the process repeatable with trivial steps that anyone can follow, with no need to maintain ELK infrastructure.
