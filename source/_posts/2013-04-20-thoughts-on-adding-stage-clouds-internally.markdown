---
layout: post
title: "Thoughts on adding "stage" clouds internally"
date: 2013-04-20 22:28
comments: true
categories: 
---
During the april release cycle, there was a need to test a hotfix to the march services; given our CI, we have two CI clouds: -next and -prod; the current model is to always have develop pushing to ci-next, and have ci-prod running the CI for the latest-and-greatest (LAG) line.

In this case, when we branched april (04), the march (03) builds were shut down to prevent accidental bleed-through and ci-prod was updated with all the requirements for 04.

The need to test 03 again meant unwinding changes made to the systems for 04, executing the CI builds for 03, confirming them, and then rewinding the changes to allow 04 to run again.

Situationally, this blew, although its not really a hard process, just time-consuming.

Latest thinking is to add, at minimum, a ci-stage cloud consisting of a pair of VMs for sharding, allowing the following model:

ci-next: LAG;
ci-stage: LAG, until branch for release, then pre-release branch, then back to LAG on branch release;
ci-prod: What's actually at AWS for production.
Adding the add'l two stage servers for ci is pretty easy, but it introduces a wrinkle: non-ci servers now violate the naming model. That is, the qa servers follow the original model:

qa-next: LAG
qa-prod: Production, until branch for release, then pre-release branch which becomes Production
In conversation with Heather, one thought is to consider the actual usage of these clouds and simply fix the naming:

qa-next: LAG, but not usually in use, or even on;
qa-stage: LAG, until branch for release, then pre-release branch, then back to LAG on branch release;
qa-prod: What's actually at AWS for production
In this model, we could simply change the name of qa-next to qa-stage at this point; other than affecting deployment scripts (and ditto for eng builds, etc.) nothing significant would occur; the trick here is that while CI needs to be able to differentiate -next and -stage, most people in the organization don't.

With our ability to bring up ad-hoc clouds, we could, if needed, turn on or bring up eng-next or qa-next in those situations where QA is simultaneously testing against the release about to occur and the develop lines due to some test cycle overlap, significant changes in the develop cloud deployed in the middle of a release time period, etc.

But at least the naming would be consistent.