---
layout: post
title: Faster Feedback Loop on Code Reviews
date: '2018-06-13T12:30:00.000-08:00'
author: David Green
tags:
  - Code Review
  - Build Time
modified_time: '2018-06-13T12:30:00.000-08:00'
comments: true
---

We've all seen those projects that start with fast builds. Fast builds that build everything and run all of the tests are a great way to get feedback on your code review, whether it be [Travis commenting on your GitHub Pull Request](https://docs.travis-ci.com/user/pull-requests/) or [Jenkins voting on your Gerrit code review](https://wiki.jenkins.io/display/JENKINS/Gerrit+Trigger). It's almost inevitable that successful projects end up with growing build times, often to the point where the feedback loop on our changes is out of control.  This post is about solving that problem.

### Traditional Approach

I've seen builds go from a few minutes, to 8 minutes, to 12, 15, and before you know it builds are taking more than 40 minutes.  Along the way we convince ourselves that it's OK, until eventually build times are out of control and review feedback cycle times are out the window.  Keeping build times to under 20 minutes is key to any kind of efficiency, with sub-10 minute builds being the sweet spot.

Fixing the problem becomes a monumental task as the complexity of the project grows, and prioritizing that work always seems to take a back seat to more urgent feature delivery.  To make it worse, as the team grows and one team becomes two or more, the responsibility of fixing the issue is not squarely within a single team and so the problem continues to go unsolved.

Our initial cut at reducing build times took some of the usual tacks:
1. parallel builds with better modularity
2. faster build machines
3. running fewer concurrent builds so that build machines have a lighter load
4. profiling/optimizing long-running integration tests

These all made a difference, but we eventually reached the limit of what we could achieve with these approaches and still, build times were reaching at best 35 minutes and in excess of 50 minutes on a bad day.

### A New Take

If we can't make our tests any faster, and we can't make the build machines run those tests any faster, what if we just ran fewer tests?  Common sense tells us that we need to run _all_ the tests - but what if we don't need to?  

With the premise that everything on master has _already passed all of the tests_, then it follows that we really only need to run tests for the code that's affected by the last change, that is, the change in our code review.

By taking the intersection of the last commit (i.e. the code review) with the transitive Maven POM dependencies and hierarchy, we can dynamically determine which projects should run tests.  This is relatively trivial by using the Maven Plugin APIs and JGit (for Git repositories).  

To run this experiment, I created the [maven-change-impact](https://github.com/greensopinion/maven-change-impact) Maven plug-in.  Details are on [the project page on GitHub](https://github.com/greensopinion/maven-change-impact), but to summarize: the `maven-change-impact` plug-in sets a Maven property based on  analysis of the Git change and the POMs.  This property can then be used to enable or skip execution of tests using the [Maven Surefire Plugin](http://maven.apache.org/surefire/maven-surefire-plugin/) on a per-project basis.

### Results

We've been running with this experiment now for two weeks and we've seen impressive results.

On average, our builds have come down to 12 minutes.  Some builds are running as fast as 4 or 5 minutes, while others are in the 18-20 minute range.
This is a massive improvement over our previous average which was in the range of 40-50 minutes.

### Side-Effects

With faster builds, we have the initial benefit of faster feedback cycle time which translates to keeping developers in the flow.

But, there are other benefits too:

With faster builds, we've seen an increase in the number of review verification builds that we're running.  In other words, we have better throughput.  This translates to being able to move more changes through our value stream, faster.  In other words, more value to the customer and improved ability to react to change.

Developers are no longer incentivized to create large reviews.  More but smaller changes reduces the cognitive overhead in code reviews, and makes for better separation of unrelated changes.  So developers are able to collaborate more effectively, and it's easier to understand the scope of any given change.

### Applicability

This approach will benefit teams the most in cases where build times are long primarily due to test execution times, and you have reasonably good modularity in your codebase.

For example, if you want to run integration tests in addition to unit tests to verify code reviews and your integration tests take more than a few seconds each, test execution times can really add up.  This definitely applies in our case, where we have more than 30,000 tests.

### Try It Out

I invite you to try out the [maven-change-impact](https://github.com/greensopinion/maven-change-impact) to see if it makes a difference for you and your team.  Enjoy, and don't hesitate to let me know if it works out for you!
