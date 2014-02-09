---
layout: post
title: "99 problems of which having 99 copies of jira is one"
date: 2013-11-07 19:38
comments: true
categories: bbbb atlassian capex tools
---
# tl;dr
We spend a lot of money on duplicate licenses for our internal engineering tools; we can do better.

# A Brief Review
BBBB PD has now absorbed the PDs formerly known as Learn, Mobile, Cloud, Engage, Connect, Collaborate and Analytics. With the notable exception of Connect, due to platform, all of these teams use Atlassian products in one form or another as engineering tools. 

The Big Four (Learn, Mobile, Collaborate, Cloud, or LMCC) collectively have the collective license capacity in JIRA and Confluence et al. to handle:

* 30,000+ JIRA/JIRA Agile users;
* 20,000+ Confluence users;
* 20,000+ Crowd users, and;
* 10,000+ Fisheye/Crucible users, among other things. 

This, as will be shown, costs us about $40,000 more a year in opex maintenance costs than we need to actually pay, predicated on the assumption that we can all share like we learned in kindergarten.

Additionally, we currently pay just around $55,000 a year in opex maintenance costs for our 350-user Perforce license and our pair of platinum-level Github organizations.

Furthermore, stretching out beyond LMCC and even beyond PD, there's at least another 100-user Perforce license for Global Services (@ $15,000 a year opex), another unlimited JIRA license at Transact (@ $8,000 a year opex), another couple unlimited Confluence licenses at Transact and Managed Hosting (@ $16,000 a year aggregate opex), and various smaller Atlassian licenses like the Atlassian On Demand usage at Analytics, cost TBD.

## A Giant Wall
And as if it weren't enough to just be spending too much, as of 22 Feb 2014, the grandfathered JIRA Unlimited licenses will automatically convert to 10000+ licenses under the [new Atlassian licensing models for JIRA](https://www.atlassian.com/licensing/jira#downloadlicenses-4). As of 2 Nov 2014, we'll have one more year before our grandfathered Confluence licenses will have to be renewed as Confluence 2000 or 2000+ licenses under the [new Atlassian licensing models for Confluence](https://www.atlassian.com/licensing/confluence#downloadlicenses-4). This will increase 2014 maintenance costs in opex from our current $8K or $10K to at least $12K for each Unlimited JIRA license, and in either 2014 or 2015 by $12,000 capex and then going forward by $6,000 in opex for maintenance.

# Proposal for 2014
Since Atlassian products are permanently licensed, there's no _particular_ need to rush any tool migrations or merging; rather, we can simply maintain the existing systems for as long as desired on expired licenses, and simply re-license, at appropriate license levels, any systems we want for going forward through 2014 and beyond. Since JIRA is likely to be the hardest set of systems to consolidate, I'll discuss that in a separate section.

## Crowd
We currently have two unlimited Crowd licenses (Learn, with 2700+ users in it, and Mobile, with 500+ users in it), and a 500-user license for Cloud with  247 users in it. Consolidating this to a single unlimited Crowd license switches maintenance costs from $9.1K per year to $4K per year, opex.

## Confluence
We currently have two grandfathered Confluence licenses (Learn, with 2900 users, and Mobile with ~500 users.) A single 10000 user license would suffice to cover all _possible_ permutations of Confluence moving forward, including absorbtion of AoD, etc. Noting the ability to maintain an unlimited license until 2015, this would save us at minimum the $8K of maintenance on one of the two Confluence licenses.

## Fisheye/Crucible
Mobile has already dropped Fisheye/Crucible in favor of straight up Github indexing and Pull Request reviews. Atlassian itself has [noted that Fisheye and Crucible are being maintained mostly for legacy VCS systems, and Git users should seriously consider pull reviews](https://answers.atlassian.com/questions/129486/git-stash-vs-git-fisheye-crucible) . To that end, and given the clear direction of PD moving from Perforce/SVN to Git, I recommend simply updating any Fisheye/Crucible instances to the latest available version as possible the last possible moment before our maintenance expires and then just not renew it. This should have _minimal_ impact in that Mobile currently doesn't use Crucible, Cloud is all Git already, Collaborate is virtually up to date on Fisheye/Crucible, and Learn Fisheye/Crucible is at version 2.5.6, released on 24 May 2011, so there seems to be little impetus to update. Canceling the maintenance costs of Fisheye/Crucible for Learn, Cloud and Collaborate would save $16K in opex.

## Clover
AFAIK, we have a single Clover 100+ license at $4K in maintenance. I have zero understanding if this is still in use, if we can drop the number down from 100+ to a smaller tier. 

## Bamboo
Since Learn and Collaborate are Jenkins-based, the only cost here is the combined Mobile and Cloud Bamboo licensing, although there's a separate Bamboo instance with no remote agents run by Global Services; at $1K in maintenance cost per year for each of Mobile and Cloud for our respective 10 remote agents, we could consolidate that into a single $2K a year maintenance license that would give us 25 remote agents, a free gain of 5 agents. Buy 20, get 5 free, so to speak.

## JIRA
This is the 800 thousand-pound Gorilla/Shark/Octopus hybrid that destroys Tokyo and then eats the moon for dessert in the sequel. Other than arguments about Git pull reviews, I'm not sure anything is going to drive more passionate response about consolidating systems than JIRA (and when I say JIRA, I'm including JIRA Agile neé Greenhopper): 

* Collaborate relies on an up-to-date JIRA on a bi-annual update cycle to drive its entire product development cycle across 40-ish product lines; Collaborate JIRA is deliberately kept as close to out-of-the-box as possible;
* Cloud relies on an up-to-date JIRA integrated with Github and Bamboo to drive significant chunks of its product development cycle; Cloud JIRA is deliberately kept un-integrated with major external systems, but has relied on more significant workflow changes wrt agile processes, as well as some simple internal and 3rd party plugins that are well-trusted;
* Mobile relies on an up-to-date JIRA integrated with Github and Bamboo yadda yadda cf. Cloud; with the retirement of jira.medu.com and the rebirth of issue tracking on tix.medu.com, there's less craziness, but tix.medu.com still has a _significant_ number of 3rd-party plugins available to the system, as well as a significant number of possibly outdated and unused workflows and fields; careful review should be considered before estimating an LOE on merging Mobile JIRA anywhere;
* Learn, on the other hand, relies on a _lot_ of plugins, workflows, historical process, processional history, integrations with Salesforce and previously Peoplesoft, and so on. See [crying unicorn](http://urustar.net/wp-content/uploads/2013/07/unicornCRY.png) for my thoughts on all that.

The point, I believe, is that while _theoretically_ we should be able to consolidate all our JIRA's down into one, and move from paying $28K+ in maintenance costs annually for three unlimited JIRA + a 500 user license, we may have to think much harder about this. There are 2900 users in Learn JIRA currently, which is an order of magnitude larger than any of the others.

Moving to a single 10K user license, and downgrading the other licenses to a couple of 500 user licenses would end up saving us about $12K in opex maintenance (currently $36K for 3 unlimited plus 1 500 license, including JIRA Agile.) This would drop to $24K (1 10K license, two 500 user licenses, including JIRA Agile.)

## Perforce and Version Control
This is a much harder reach, but we currently spend about $52,000 in maintenance for our 350 user Perforce license in PD, and another $15,000 for the GSVCS Perforce server for Global Services; on top of that, we've got about $4800 a year in our pair of Platinum Github organizations (blackboard and blackboardmobile); There's then the additional costs of Bitbucket, but these are minor in the grand scheme of things.

Given the push to move to Git as our VCS of choice, any decrease in user maintenance costs in Perforce is a net win, which leads to Stash:

## Stash

Github user management for an organization of our size is painful at best; Git, however, makes it simple to move remote repositories around. Since Stash would integrate easily with our current Git workflows as well as our existing tool stack, I'm recommending moving our internal DVCS repositories to a Blackboard ES:O run Stash instance.

A 1000-user license would be a $28,000 hit on capex in 2014. A 2000-user license would be a $48,000 hit, but probably unneccesary given that we have no more than 450 users between our two Perforce instances, and _maybe_ another couple hundred in our various Github and Bitbucket licenses.

The $28K capex hit is _easily_ absorbed by the consolidation of existing Atlassian licenses, which I'll discuss below in my summary of costs.

# Show Me The Money
Given all the points made above, my numbers show the following:

* JIRA+Agile costs in 2014 would drop from $36K to $24K, giving us $12K of opex;
* Confluence costs in 2014 would drop from $16K to $8K, giving us $8K of opex;
* Bamboo costs in 2014 would remain constant at $4K, but give us 20% more agent capacity;
* Crowd costs in 2014 would drop from $9.1K to $4K, giving us $5K of opex;
* Fisheye/Crucible costs in 2014 would drop from $16K to $0, giving us $16K of opex;

This frees up $41K of opex in 2014. Stash then adds $28K of capex to the 2014 budget, which at 1-1 still frees up $13K, and assuming we can convert the $28K of opex to capex at 3-1 for the Stash purchase, still leaves  the $13K opex, while opening up $18K of add'l capex for 2014.

The faster we can then migrate and shrink user requirements to legacy code in Perforce, the faster that we might be able to reduce the opex hit on the 2014 budget for Perforce, but regardless, the 2015 Perforce budget should be able to be significantly reduced.

# Bonus
This has all been predicated on the Big 4. Once we start looking at the AoD costs, the smaller JIRA/Confluence/et al. instances, the numbers should get even better, however small the added bonus is.
