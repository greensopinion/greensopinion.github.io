---
layout: post
title: Learning New Things
date: '2016-03-29T17:18:00.000-08:00'
author: David Green
tags:
modified_time: '2016-03-29T17:18:00.000-08:00'
original_url: http://www.tasktop.com/content/blog-entry/learning-new-things
comments: true
---
It's mid-2015. Our product and platform had evolved from a pure Java application to a mixed-technology web-based solution, and I found myself realizing that I had to up my game. I'd built many web applications before. But my drive to hone my skills was coming from the company's need to move ahead with a faster, better, more modern product development approach using technologies that were somewhat new to me: npm, Bower, AngularJS, Grunt, Karma, Jasmine... there was a good-sized list of them.

As a software engineer I've always believed that it's my responsibility to contribute powerfully to the team's delivery. By that I mean that I should be able to tackle any problem with confidence, knowing that the solution would flow from my keyboard as fast as I could think it up. That was definitely not happening with this web stack, so it was time to do something about it. I'm a big fan of learning by doing, so I decided to take it on by creating a small application to exercise my weaknesses using the very technology stack that made me uncomfortable.

Fast-forward to 2016. Having spent many evenings and weekends buffing up my skills, it's been over 9 months now since I've had any misgivings about my ability to deliver and it feels great! In the process I've created [a personal finance application](https://github.com/greensopinion/greenbeans) using Java and AngularJS.

<img src="/images/blog/greenbeans-report.png" alt="Personal Finance Application" width="400px"/>

I've open-sourced my finance application not only because I find it useful, but because it's a great showcase of an architecture and many of the techniques and technologies of Tasktop's integration platform.  Feel free to take a look - you'll get a sense as to the technology stack that we use at Tasktop, and what it might be like to work with our platform codebase ([we're hiring!](http://www.tasktop.com/careers)).

Things to look out for include the testing approach, and how to write testable software.  Take a look at how dependency injection, Guice, JUnit and Mockito can be used to test units of code while minimizing dependencies.  See how the the JavaScript and UI components can be tested independently of the back end by mocking up web services.

The client-side architecture looks something like this:

![Client-Side Architecture](/images/blog/greenbeans-client-side-architecture.png)

The somewhat typical AngularJS stack includes bootstrap, sass, font-awesome and chart.js.  Testing dependencies include jasmine, karma and phantomjs, built with npm, Grunt and Bower.

The server-side architecture looks something like this:

![Server-Side Architecture](/images/blog/greenbeans-server-architecture.png)

Unlike the Tasktop platform, which is a proper server-side web application, this is a desktop application.  There's a somewhat novel approach to hosting this all inside of a single JavaFX process with a WebView while still using web services:

![JavaFX, WebView and JavaScript / JAX-RS Bridge](/images/blog/greenbeans-javascript-bridge.png)

To make this work, the JAX-RS services are actually just bridged to the JavaScript application in-memory - no networking needed!

This approach enables coding of a "normal" AngularJS application using standard JAX-RS REST services, but as a desktop application.  This has the benefit of leveraging a solid modern web architecture with good separation of concerns, with great web technologies for UI development.

I hope that you [enjoy having a look at the souce](https://github.com/greensopinion/greenbeans).  If you find it interesting and if you're inspired by Tasktop's mission to connect the world of software delivery, please [check out our openings - we are hiring!](http://www.tasktop.com/careers)
