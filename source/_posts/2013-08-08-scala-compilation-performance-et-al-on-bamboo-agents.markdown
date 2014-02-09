---
layout: post
title: "scala compilation performance et al. on bamboo agents"
date: 2013-08-08 07:29
comments: true
categories: bbbb cloud scala bamboo performance 
---
Most of this was initially captured in http://jira.cloud.local/browse/DEVOPS-685, but http://jira.cloud.local/browse/DEVOPS-697 now also exists to continue tracking improvements.

I'll start with the basics, or technically, the basic, singular:

*sbt compilation performance on the bamboo agents has, to date and to be blunt, sucked donkey cojones. using profile as my experiment, i'm able to sbt compile locally on my laptop in about 40 seconds; sbt test:compile takes another 30-35 seconds. On an agent, compile takes around 180 seconds, and test:compile another 150-160 seconds. This is basically linear with respect to the number of scala classes being compiled.*

We've made incremental improvements over time, but nothing has ever really dug into the heart of why the bamboo agents are so ass-slow. And so here we go.

The first steps were to copy the existing profile plan to http://bamboo.cloud.local/browse/SOCLSVCEXP-DO685, and break out each step individually to be able to effectively measure time taken including bamboo overhead.

In the end, this resulted in eight serial stages:

* SETUP
* COMPILE
* TEST
* TESTCOMPILE
* BUILDTEST
* STAGE
* BUILDSTAGE
* PACKAGE

Many of these stages overlap; for reference, the standard profile build in Bamboo involves two stages: a compile/test/package stage, and a deploy-to-CI stage. For the purpose of this experiment, the deploy stage became irrelevant from a timing perspective, and was disabled.

In the standard build, a single call is made to sbt: SBT_OPTS=... ; sbt compile test stage. This has the benefit of executing the entire build lifecycle in a single sbt execution, so the process is aware of the overall flow, and only executes compile once, resulting in an overall build lifecycle time of about 13 minutes (prior to any tuning).

In the exploded model, as evidenced in its final incarnation as http://jira.cloud.local/secure/attachment/12837/Screen%20Shot%202013-08-04%20at%2011.52.50%20AM.png, each stage may re-execute chunks of the sbt lifecycle: for instance, while COMPILE (labeled BUILD in the image) takes a mere 4.2 minutes, TEST not only executes the test:compile and test lifecycle phases, but also the compile phase; by executing them separately, and subtracting out the known COMPILE time, we're able to determine the actual TEST phase time. This is all just to explain why it seems that certain phases may feel overly long.

So, having said all that, and referring back to the image showing build times for the final exploded view of the profile build:

The SETUP and PACKAGE steps are trivial: these steps involve git clone and tar -cz calls, and run in consistent time regardless of tuning. More to the point, at under a minute total for the both of them combined, they're low on the tuning question.

COMPILE consistently took about 6.3 minutes prior to tuning, as evidenced at http://jira.cloud.local/secure/attachment/12833/Screen%20Shot%202013-08-04%20at%207.56.18%20AM.png

Also prior to initial tuning, the rest of the stages, when compared to post-tuning, were a constant time multiple of the tuned bamboo agent. As such, I'm simply going to state that the initial tuning, as described below, sped up CPU bound compilation by about 25-35% on both the compile and test:compile phases of the lifecycle.

To finish describing the first image: post-tuning, compile takes about 4 minutes overall, including overhead. Subtracting that time from the TEST step indicates that TEST takes about 5 minutes overall; Comparing the TESTCOMPILE and TEST stages indicates that TESTCOMPILE takes about 3 minutes and actual test execution takes about 2 minutes. STAGE, the actual artifact generation can also be shown to be non-considerable by comparing BUILD v. STAGE v. BUILDSTAGE and noting that since sbt stage and sbt compile stage execute the same code path, and we know the time for sbt compile, we can easily notice that the stage phase is trivial.

OK, on to the fun:

If compile and test:compile are the chunks taking up the majority of the build time (~65-70%), then why?

And here's the first demonstration of improvement:

The VMs provided by DIRE are running on a cluster of hypervisors managed by VCenter; to date, and unbeknownst to us, said VMs have been running in a resource pool for all the cloud VMs. So of the vast CPU and memory space available to the cluster, we've been sitting in a little wading pool of CPU and memory space, designed to prevent our VMs from impacting the performance of the overall cluster. But better than that, not only are we resource limited across the 64 VMs in our pool, but if the bigger pool needs more resources, we're not guaranteed our pool, we're just limited by it. Our pool is as big as we'll ever get, but it can get smaller if there's a drought elsewhere.

So we tested this by moving five agents out of the wading pool into the big kids pool; and there, immediately, was our 25-35% improvement. Both compile and test:compile times dropped by 30%. At this point (after confirming this a few times), I disabled the test compile, test, stage, etc. phases, since it seemed that comparing compile to compile was good enough, and shaved 10 minutes off my testing.

So, so far, so good: we're 30% faster on scala builds, down from 260s to 180s, but its still nowhere near the 40s locally.

So in talking with Matt and Kohn, we devised a new plan: get a VM as close to having dedicated cores as possible: each agent is setup to have 2 virtual CPUs; if we can map those two virtual CPUs to physical cores, we should be able to figure out just how fast the VMs could run when not in contention for CPU.

And fortunately, there's a couple of 810s just setup that have no other VMs in the cluster.

So, we moved agent10 over, forced profile to build on it, and got the following:

BEFORE: http://jira.cloud.local/secure/attachment/12863/Screen%20Shot%202013-08-08%20at%205.04.04%20PM.png - 10.5 minutes to compile test and stage

AFTER: http://jira.cloud.local/secure/attachment/12864/Screen%20Shot%202013-08-08%20at%205.04.13%20PM.png - 4.1 minutes to compile test and stage.

And so there you have it: in a couple days, we went from 16-20 minutes of time to compile, test, package and deploy profile to 6 minutes, and all thanks to unlimited CPU.

Now, anyone have massive spare CPU time lying around?

