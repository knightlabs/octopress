---
layout: post
title: "Quick thoughts on various clouds now that we're sharding"
date: 2013-04-20 22:30
comments: true
categories: 
---
Per a conversation with Dan Rinzel about a need for documentation to have a cloud they can connect their machines to, it seems we have two sets of things to figure out: what clouds do we need, and how do we train people to self-service;

Right now, we have (or have in discussion) three (3) publicly facing social clouds: production, beta and possibly a much smaller go-to-market cloud; production is obviously always on, but beta is designed to be started up and shut down as needed during release cycles, making it suboptimal for use by external "always-on" demo and test servers, hence the theory for a much smaller and cheaper system to run an always-on demo and test cloud.

However, those systems are less than useful for Learn instances inside the firewall, since communication can't come back in; to that end, we currently have 10 internal clouds running, comprising (when available) 20 virtual machines (the 01 and 02 shards for each cloud.)

latest and production engineering
latest and production qa
latest and production product management
latest security
latest go-to-market
latest and production continuous integration
However, given a situation that came up last week, we're now discussing adding a set of staging clouds along with latest and production; more on this in another post, but we're very rapidly growing our number of systems.

To date, we've allocated clouds based on groups using them; security v. engineering v. qa, etc. with some notable reasons for some of the splits (qa and eng split because qa needs SSL but eng doesn't, and fixing all the learn instances to trust the cloud.local certificates was overkill. Nb. we may resolve this by adding cloud.bb virtual hosts to the systems allowing us to use a valid trusted certificate everywhere, which would allow us to combine eng and qa clouds easily.

With documentation requesting access to an internal cloud, there's several options: we could certainly use Lisa's pm clouds at the risk of having data bleed visible between what Lisa is using and what documentation is using (all the social data). We could set up a new pair of clouds (doc-next and doc-prod) isolated to documentation, but as noted above, the theoretical need for three doc clouds starts to get out of hand, as we're looking to minimize churn on documentation Learn servers;

So I was thinking: what if, instead of allocating clouds by group, we allocate them by purpose? The biggest differentiating factor in terms of cloud use, other than group, is the requirement for duration of data such that need for recreation of cloud data/re-connection can be a well-understood setting.

Having not really spoken to the security and g2m groups, my expectation is that we have _ types of data requirements:

continuous integration: data is completely transient, and any Learn instance or test connecting to these systems must set up its own data from scratch, always
engineering/qa: data is somewhat transient: systems should reasonably expect that registration data, et al. remains over time, but there is zero guarantee of that; eng/qa data can and at points will be destroyed and systems should be expected to be able to recover; possibly a guaranteed schedule of data destruction will help drive that point eg. "every sunday the eng/qa databases are destroyed assuming no active cross-boundary cycle, etc."
pm/docs: data is non-transient: devops will make reasonable efforts to ensure that data remains consistent across the systems to minimize impact on non-technical teams with appropriate notifications assuming a "registration" process so we know who to actually notify.
ad-hoc: now that ansible is driving most and soon all of the setup and configuration of stack cloud systems, the ability to bring up and tear down clouds for various purposes is almost one-click; if security or l10n needs a cloud to mess with, bringing these up and shutting them back down when complete are reasonably easy and getting easier
With g2m and external demos expected to move outside the firewall, the real question is whether the "non-transient data" group can share a cloud, or whether that creates enough impact on visual data bleed that this is a non-starter.

Possible players: lisa, rinzel, mcgarr, devops, eric, mdey, heather, franco/stan, g2m?