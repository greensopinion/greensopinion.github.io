---
layout: post
title: Architecting for the Value Stream
date: '2018-01-22T12:00:00.000-08:00'
author: David Green
tags:
  - Architecture
modified_time: '2018-01-22T12:00:00.000-08:00'
original_url: https://sdtimes.com/arch/architecting-value-stream/
comments: true
listing_image: /images/blog/2018-01/software-architecture.jpeg
---

More features, faster time to market, fewer defects.  These are what organizations strive for. But an overly narrow continuous integration and continuous delivery focus on DevOps is insufficient to achieve these goals. Instead, we should be taking a whole-system point of view that considers factors beyond development and operations — including people, teams, processes and tools.  By taking a constraints perspective, we can identify and address flow-limiting aspects of our value stream.

<img src="/images/blog/2018-01/software-architecture.jpeg" class="pull-right img-responsive"/> 
The foundation of “good” architecture — principles such as modularity, separation of concerns and SOLID — deliver the promise of flexibility, maintainability and all of the other “ilities” that we value.  But these qualities, while enabling faster and sustainable delivery of business value, fail on their own to address some constraints that limit the value stream.  For example, delivery aspects such as multiple teams; new team member ramp-up; support and troubleshooting; and user feedback impose constraints on our value stream.  By taking a wider perspective than only technically driven architecture aspects, we can optimize it to remove some constraints.

### Collaboration with multiple delivery teams

Multiple delivery teams can deliver more features faster than one, but this only holds true if introducing more teams doesn’t introduce additional constraints.  But multiple teams introduce greater potential for conflicts and corresponding churn as more people work on the same codebase, build pipeline, tests and infrastructure.  So how do we minimize this problem with our architecture?

With a technology-first approach, you may architect your module or microservice boundaries in a way that optimizes separation of concerns and cohesion, e.g. loosely coupled functionality grouped by related concepts. Meanwhile, architecting value stream first considers the alignment of teams and value streams to the technology. As such, you may create API, microservice and module boundaries that provide autonomy to teams, enabling them to deliver features independently of other teams. And you will plan for evolving those APIs to avoid wait states that can lead to slower cycle times.

For example, if a team must wait four weeks for a new API that is required for a new feature, you would refine your architectural process to remove the dependency, provide an accelerated way of having that team contribute to the API, or take the architectural debt of copying the functionality while ensuring you have a path to remove that debt.

Let’s take this thinking and apply it to other aspects of our delivery: build and test infrastructure.  Is one team causing build breakages for another?  While modifying a test fixture configuration, does that team cause another’s tests to fail?  Our natural tendencies might be to share build and test environments to avoid duplication and minimize redundancy.  If you are architecting these systems value stream-first, you will design the build and test pipeline and corresponding environments in a way that insulates each team. Meanwhile, a share-nothing approach, while it may introduce redundancy, could be a great trade-off to eliminate dependencies and friction between teams.

One way of achieving share-nothing builds is to have tests provision their own environments for the required duration.  For example, a test suite that verifies services running over MySQL could bring up a containerized MySQL instance for the duration of the test, and destroy it when the test is complete.  Effort is required to enable tests to accomplish this level of provisioning; however once complete the burden of creating, maintaining and administering long-running test fixtures is eliminated.

### Feedback loop

A good feedback loop is an essential part of enabling the value stream.  Tests provide feedback to engineers so that they know whether the system works.  Fast tests, running earlier in the process (for example in the IDE) create a shorter feedback loop, enabling engineers to react more easily and more quickly.  A great feedback loop, however, goes beyond test speed and the trade-off between unit tests and integration tests per the Test Pyramid.  Test speed is important, but only covers a very small part of the engineering delivery cycle.

The biggest constraint on receiving user feedback is typically the user. To remove this constraint, you must remove the user, which is a paradox since user feedback is what we seek.  While it may not be possible to remove the user for all user feedback, we can for a class of it.  By architecting a product that automatically provides feedback without the user having to instigate it, we can approximate user feedback and eliminate the constraint.

For example, you should build metrics into the application so that you know before the user if a new feature is too slow.  Monitoring, instrumentation, telemetry and analytics are essential feedback tools that should be integrated into every application, not just mobile apps.

### Cycle time and sustainability

In many organizations, focusing on the value stream means delivering new user features. Often, it’s seen as a business decision as to whether to deliver a new feature and possibly incur additional technical debt or pay off existing technical debt. It is indeed a business decision, but making any decision should include understanding the consequences of that choice.

The true cost of technical debt is often misunderstood. By definition it slows down you, your team or the organization. While one small technical debt may not impact you significantly, pile up a few of them and your velocity will suffer. Pile up more than a few of them, and beyond slower delivery speeds, team morale and team culture will suffer.

The choice of whether to accumulate or to address new technical debt is often made without understanding the scope and quantity of existing technical debt. Take great care to avoid a pile-up. Instead err toward having little or no technical debt at all. While it may sound like an unattainable goal, it is worth pursuing, since it enables teams to perform at their highest level.

### New team member ramp-up

Staff turnover is a normal part of software delivery. Consider the time required to make new staff productive, the mistakes they make while ramping up and their consequences. These strategies can help on-boarding issues:

1. Avoid documenting, instead automate: Automated things don’t require learning to get them right, and if done well are self-documenting. Automated things run the same way every time, don’t require learning how and tend to run very fast. Automating everything is a great strategy for reducing the on-boarding burden.
2. Modular systems are easier to learn than monolithic systems because of their decoupled nature. Components have fewer dependencies, and are grouped by functionality. This means that new team members have less to learn in specific areas, since they don’t need to understand the whole system. This translates to faster time-to-value.

Thinking holistically about the value stream as it relates to the whole organization can provide insights into improved software architectures. A traditional architecture view is limited to the technical merit of a solution; however you may need a broader view to address the competitive needs of modern organizations by delivering a steady stream of value to customers.  Seek architectural strategies to address issues that involve more than just the technical aspects of software delivery. Consider too, the teams and people that are involved from idea to delivery and operations.
