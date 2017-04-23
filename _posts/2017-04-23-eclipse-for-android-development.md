---
layout: post
title: Using Eclipse for Android Development in 2017
date: '2017-04-23T12:05:00.000-08:00'
author: David Green
tags:
  - WikiText
modified_time: '2017-04-23T12:05:00.000-08:00'
comments: true
---

Innovation in platforms and tooling is necessary for companies to remain competitive, and Android development is no exception.  The Android team at Google has been working hard in this department, with their latest major new thing being [improved Java 8 support](https://android-developers.googleblog.com/2017/03/future-of-java-8-language-feature.html).  This involves some big changes to the [Android build system](http://tools.android.com/tech-docs/new-build-system), which unfortunately has caused some casualties in tooling such as [this previous approach to using Eclipse for Android development](/2016/05/15/eclipse-for-android-development.html).

With the prospect of cleaning up my Android codebase with Java 8 lambdas, I was unable to resist the new Java 8 tooling.  A suggestion from [Johannes Brodwall](http://johannesbrodwall.com/) got me thinking about a Gradle plug-in to simplify generating an Eclipse `.classpath` in a way that is compatible with the innovation going on in Android tooling.  Feeling inspired, I created [this plugin](https://github.com/greensopinion/gradle-android-eclipse) which does the heavy lifting, adding the necessary classpath entries and expanding Android Archive (AAR) files so that their nested jars can be included.

To use the plugin, add the following to your build.gradle:

    apply plugin: 'com.greensopinion.gradle-android-eclipse'
    apply plugin: 'eclipse'

	buildscript {
	    repositories {
	        maven {
	          url "https://plugins.gradle.org/m2/"
	        }
	    }
	    dependencies {
	      classpath "gradle.plugin.com.greensopinion.gradle-android-eclipse:android-eclipse:1.0"
	    }
	}

	eclipse {
	  classpath {
	    plusConfigurations += [ configurations.compile, configurations.testCompile ]
	    downloadSources = true
	  }
	}

Then from the command-line run:

    $ gradle eclipse

When done, a `.classpath` and `.project` file should be in the current folder.  You'll need to run `gradle eclipse` whenever changes are made to your classpath.

To find out more about the plugin, head on over to [https://github.com/greensopinion/gradle-android-eclipse](https://github.com/greensopinion/gradle-android-eclipse)

Enjoy!
