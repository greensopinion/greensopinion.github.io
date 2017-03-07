---
layout: post
title: Mylyn WikiText on Maven Central
date: '2017-03-06T12:00:00.000-08:00'
author: David Green
tags:
  - WikiText
modified_time: '2017-03-06T12:00:00.000-08:00'
comments: true
---

I'm excited to announce that for the first time, [Mylyn WikiText](https://wiki.eclipse.org/Mylyn/WikiText) is now available on Maven Central.

This release enables use of Mylyn WikiText within a normal Maven project; previously Mylyn WikiText could only readily be consumed by downloading the jar files or by resolving dependencies using Tycho.

Now dependencies can be resolved at the following coordinates:

    <dependencies>
        <dependency>
            <groupId>org.eclipse.mylyn.docs</groupId>
            <artifactId>org.eclipse.mylyn.wikitext</artifactId>
            <version>3.0.2</version>
        </dependency>
        ...
    </dependencies>

Mylyn WikiText provides several additional components (e.g. for Markdown, AsciiDoc, etc.) which are [listed here](https://search.maven.org/#search%7Cga%7C1%7Cmylyn.docs).

To celebrate the new release I've created an examples project on GitHub, which shows how to use Mylyn WikiText in a few different ways:

[https://github.com/greensopinion/wikitext-examples](https://github.com/greensopinion/wikitext-examples)

The examples demonstrate:

* Mylyn WikiText Java APIs
* Mylyn WikiText Maven mojo to generate HTML help using a pom
* Mylyn WikiText Ant tasks from within Maven

From now on Mylyn WikiText releases will provide Maven artifacts from Maven Central, and p2 artifacts for OSGi and Eclipse applications.

Enjoy!
