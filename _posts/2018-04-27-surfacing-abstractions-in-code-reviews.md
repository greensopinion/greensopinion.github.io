---
layout: post
title: Surfacing Abstractions in Code Reviews
date: '2018-04-27T13:30:00.000-08:00'
author: David Green
tags:
  - Architecture
modified_time: '2018-04-27T13:30:00.000-08:00'
comments: true
---

Code review tools such as [GitHub PRs](https://github.com/features/code-review) and [Gerrit](https://www.gerritcodereview.com) do a a great job of enabling developers to collaborate, but their file- and line-oriented approach leaves abstractions out of the picture.  Developers must have the capacity to see the abstractions by reading the code, or risk having code reviews devolve into a critique of syntax, whitespace and implementation detail. The intrinsic ability to see abstractions comes easily for some, but others are at a disadvantage especially when less familiar with the codebase.  To address this issue, I conducted an experiment in surfacing code abstractions as a diagram in code reviews.

My premise was that a static class diagram would enable developers to see the major abstractions and their relationships independently of the implementation detail.  By having this higher level view of the code related to a change, developers would have a deeper understanding when performing a code review.  My goal was two-fold: to enable ramp-up of less experienced engineers, and to draw more attention to the abstractions since in many cases they as if not more important than the implementation.  (A poor abstraction implemented perfectly is in many cases less valuable than a great abstraction with a sub-optimal implementation.)

To run this experiment, I created [a plug-in for Gerrit](https://github.com/greensopinion/gerrit-class-diagram-plugin) that dynamically generates UML static class diagrams for any code review.  These diagrams are intentionally missing detail, taking the [UML-as-sketch](https://martinfowler.com/bliki/UmlAsSketch.html) approach to avoid overly-complicated diagrams.

I was looking for answers to the following questions in this experiment:

* Could diagrams be automatically generated that were good enough to communicate the major abstractions?
* Is it possible to pull in more of the surrounding context so that the changed code can be evaluated within the context of a larger codebase?
* Would developers find these diagrams useful?
* Would these diagrams affect the code review process sufficiently to driver higher quality changes to non-trivial systems?

The result of this experiment was surprising.  A simple, well-structured code review produced the following diagram:

![Example Generated Diagram](/images/blog/2018-04/simple-generated-diagram-detail.png)

This is what it looks like on the Gerrit code review:

![Example Generated Diagram](/images/blog/2018-04/simple-generated-diagram.png)

So far, so good.  But, what about a more complicated example?  Following is a diagram generated from a real contribution:

![Another Example Generated Diagram](/images/blog/2018-04/complex-generated-diagram.png)

This diagram is much harder to read, even when zoomed in.  My initial reaction was that this diagram was not useful at all. The code and its surrounding context were simply too complicated to enable generation of a useful diagram.  After saying that to myself, it dawned on me: This is exactly why the diagram is useful.  The generated diagram is exposing code for what it is: overly complicated with poor abstractions and too many dependencies.

In my experience, most developers aren't as practiced at reading static class diagrams as they are at reading code. For these diagrams to be useful, developers would need to become proficient at reading them.  Proficiency comes with practice, but for that to happen, developers would have to be willing to try out the approach.

This experiment isn't over.  The next step is to try it out for a few weeks and get feedback from developers.  
What do you think, would this be helpful for you or your team?  If you're interested to try it out, the project has been [published on GitHub](https://github.com/greensopinion/gerrit-class-diagram-plugin).
