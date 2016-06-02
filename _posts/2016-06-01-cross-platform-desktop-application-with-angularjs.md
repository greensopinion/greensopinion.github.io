---
layout: post
title: Building a Cross-Platform Desktop Application with AngularJS and Java
date: '2016-06-01T12:00:00.000-08:00'
author: David Green
tags:
  - AngularJS
  - Java
  - WebView
  - JavaFX
modified_time: '2016-06-01T12:00:00.000-08:00'
comments: true
---
Building cross-platform desktop applications usually means compromising on user experience, ease of development or both.
Below I detail the reasoning and trade-offs behind an approach that involves state of the art technologies and development methodology with fewer compromises.

Web technologies (HTML, CSS and JavaScript) are arguably the best choice for a cross-platform UI in most situations.   With numerous great choices for frameworks, skills commonly available, and unparalleled flexibility in styling, it's hard to imagine choosing anything else as long as you don't mind your app looking like a web application.

Creating applications with the likes of [Electron](http://electron.atom.io/) or [MacGap](https://github.com/MacGapProject) is nothing new - there are several successful such applications in widespread use, such as [Slack](https://slack.com/), [VisualStudio Code](https://code.visualstudio.com/) and [WordPress Desktop](https://desktop.wordpress.com/).  But building a JavaScript-only application isn't for everyone, especially in cases where reuse of existing functionality is important.

Many organizations have substantial functionality available in the form of Java libraries that they would like to reuse and the corresponding skills, infrastructure and methods to build and maintain application logic in Java.  Taking an Electron-like approach to building such applications but with an emphasis on Java for the functional portion of the application could be a good choice in these cases.

Java ships with a WebKit-based WebView for JavaFX.  This made a bit of a splash, as previous attempts to embed a browser in Java apps were either using a sub-par Java-based browser, or had crazy dependencies and complex setup.

Creating a JavaScript application using a JavaFX WebView [has been done before](http://www.salidasoftware.com/using-java8-build-deploy-native-html5-applications/).  On the surface this approach looks great - with the benefits of a state-of-the-art browser engine, standard APIs, no extra dependencies and tools to create platform-specific installers, many of the previous problems are solved.  But the approach hasn't seen wide adoption, in part due to these shortcomings:

### 1. HTML and CSS Tooling

Web developers rely heavily on browser-based tools to develop the look and feel of an application.  Modern web browsers provide integrated tooling that provide a way to introspect and modify CSS and HTML.  With a couple of clicks, developers can troubleshoot and try out styles and HTML, live in the application. This ability to change an application as it's running is one of the cornerstones behind web developer productivity.

A JavaFX WebView doesn't have such tooling.  The WebView is a black-box, essentially locking developers out from their most powerful development approach.  It's like going back to the mid '90s.

One approach to overcoming this limitation is to include [Firebug Lite](https://getfirebug.com/firebuglite), which enables inspection of HMTL and CSS in almost any browser, including the JavaFX WebView.  I did get it working, and though it's better than not having any tooling, not all versions of Firebug Lite worked for me and I was unable to edit CSS properties live in the application.  To be fair, Firebug Lite is not intended as a replacement for Firebug or Chrome development tools.  In the end I found that it was insufficient for my needs.

### 2. JavaScript Debugging

No matter how good your JavaScript automated testing approach, there is no substitute for debugging JavaScript live in a running application.  It's the last-resort fall-back to figuring out what's really happening in your application.  With no built-in JavaScript debugger, the JavaFX WebView left me feeling like it was the early days of the web.

### 3. JavaScript to Java and Back Again

With the UI in a WebView, calling out to your Java application (or the reverse, calling into the JavaScript web UI) is an important aspect of this approach.  Unfortunately crossing the JavaScript-Java boundary is not entirely straight-forward.  Though it's possible to invoke JavaScript in the WebView from Java, and call back to Java from JavaScript, it's not transparent.

Data type conversion is very limited; most non-trivial use cases will require Java code that introspects ```JSObject```, a generic JavaScript type wrapper.  This makes for clumsy Java code and in most cases reduces the Java-to-JavaScript boundary to a message-passing paradigm.

### 4. WebKit Version

Having a WebKit-based browser ship with the Java platform is great, but which version is it?  It appears that updating WebKit on the Java platform is [quite a bit of work](https://bugs.openjdk.java.net/secure/attachment/49253/webkit-merge-journal.txt), since Oracle is not updating the WebKit version very frequently.  As of this writing, Java 8 ships with WebKit version 537.44, which is from 2013.

If the embedded WebKit is used as a web browser - that is, to browse the web, not just for your application - then having an up-to-date WebKit is critical for security.

### 5. Download Size

Distributing a JRE with the application is needed to ensure that the application runs consistently and to keep installation simple (avoiding the need to install a JRE separately).  As a result the application size is over 100MB, which could be problematic for some situations.

## Mitigating the Drawbacks

With a laundry list of drawbacks our approach of using a JavaFX WebView to host the UI for a desktop Java application is sounding pretty bad so far.  But, with a concrete mitigation strategy for each issue, it could be worth exploring further:

### Web Development Approach

Web development tooling leverages our team's expertise and is key to productivity - so how can we come up with a development approach that enables this?  

It turns out that we can build a traditional web application by implementing REST services as the interface between our UI and our application.  By having the UI call out to REST services, we can have our cake and eat it too.  

With this approach our application architecture looks something like this:

![JavaFX Application Architecture](/images/blog/2016-06/greenbeans-application-architecture.png)

In development we can deploy the application as a normal web application like this:

![Web Application Architecture](/images/blog/2016-06/greenbeans-dev-architecture.png)

By providing an in-memory HTTP/REST bridge to these services we can run the UI and application logic in one process, while in development we can split the application from the UI by using a web browser and web container.  This means that we get the single-process deployment profile of a desktop application, while retaining the web development tools and techniques that we value so much.

### Java to JavaScript Impedance Mismatch

To overcome the difficulty of type compatibility and `JSObject` called out above, we turn to our REST services again.  JavaScript with frameworks like AngularJS is purpose-built to call REST services.  And, by passing JSON data over HTTP (or over the REST bridge) all of our data is serialized to a standard format (JSON) which is trivial to deserialize on the Java end.  In fact, the impedance mismatch completely disappears by implementing REST services and it feels like developing a "normal" web application.

## Build System

A major problem for any project is the build system.  Maven, Ant, Gradle - whatever you choose, we still need something to build the webby front-end.  For this application I chose Grunt with Node, Karma, Jasmine... it's all there.  Ultimately this is the same as any web application using more than one implementation language, there are two build technologies involved, which complicates things significantly.  

## Testing Strengths

Web technologies for testing the UI are fairly mature and enable testing on several levels:

* Unit testing is pretty straight-forward when using AngularJS, Karma and Jasmine, enabling a high level test coverage for JavaScript backing the UI with a reasonable effort.
* Karma and Jasmine can also be used to test for elements in the DOM.
* Though I haven't done it here, with use of [Selenium WebDriver](http://www.seleniumhq.org/projects/webdriver/) it's possible to test application functionality via the UI.
* Using a Java back-end, it's really simple to use normal unit testing techniques and frameworks, for example with JUnit and Mockito.

Selecting web technologies for the UI makes thorough testing a possibility, covering all aspects of our application architecture.  

The only real problem here is that our UI tests will be running using a web browser rather than the JavaFX WebView.  It's hard to say if that's a major gap or if it's something that we could live with - I think only experience would shed light in that area.

## Migration to Web

If future application deployment as a real web application rather than a desktop application is a consideration, then this approach could be a great way to move forward.  The application architecture being a traditional web stack lends itself very well to a web deployment.

## Working Example

A working example of such an application is [published on GitHub](https://github.com/greensopinion/greenbeans).  I'd love to hear your feedback and thoughts on the application and the approach.

# Summary

Building a desktop application using a JavaFX WebView with an embedded AngularJS app is not simple to set up, but does enable reuse of existing Java services and business logic.  For teams that are want to leverage existing skills (e.g. Java and HTML/JavaScript/CSS) this could be a good choice, as long as having a desktop application look like a web application is OK.

Many, but not all of the development difficulties introduced by running a web application in a JavaFX WebView instead of a standard web browser can be mitigated by supporting a normal web development flow as outlined above.

For me this was [a great learning experience](/2016/03/30/learning-new-things.html) and potentially worthy of further exploration.  Would I use this as a product development approach for a real application?  Maybe, if the trade-offs were to weigh in favour for the needs of the organization and application under consideration.  But I would proceed with caution, conscious of the tradeoffs and always looking for opportunities to improve.
