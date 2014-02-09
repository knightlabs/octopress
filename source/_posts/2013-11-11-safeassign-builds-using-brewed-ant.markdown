---
layout: post
title: "Safeassign builds using brewed Ant"
date: 2013-11-11 06:09
comments: true
categories: bbbb safeassign ant homebrew
---
Two immediate pre-requisites: `brew install ant` doesn't install ant-contrib or testng, both of which are apparently required in $ANT_LIB (for the taskdefs) as well as in Safeassign Antfiles at this point. So really simply, get ant-contrib-1.0b3.jar from the Internet or the Safeassign common/lib tree. Do the same with [testng](http://testng.org/doc/download.html). Put them in `/usr/local/Cellar/ant/1.9.2/libexec/lib`. Profit.