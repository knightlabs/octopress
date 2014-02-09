---
layout: post
title: "HOWTO Get Crucible to have a sane base URL when proxied through nginx"
date: 2012-11-16 11:57
comments: true
categories: 
---
Just don't pay any attention to the Atlassian documentation. Its SHIT. Go to crucible admin > global settings > server. Edit settings. Under Common Configuration, under the Proxy Settings, change 'Site URL' to 'http://crucible.cloud.local'.

YMMV and hopefully you'll never have to do this.
