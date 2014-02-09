---
layout: post
title: "HOWTO Switch JIRA to Crowd SSL"
date: 2013-08-11 22:25
comments: true
categories: 
---
Two things: not only do you have to switch up the crowd configuration file in WEB-INF/classes, you also have to update the database for any Crowd directories you've got inside JIRA;

update cwd_directory_attribute set attribute_value='https://crowd.cloud.local/crowd' where directory_id=10000 and attribute_name='crowd.server.url';

And obviously then bounce JIRA.

