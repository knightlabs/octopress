---
layout: post
title: "HOWTO Enable SSO for Fisheye or Crucible when using a non-standard Crowd cookie token name"
date: 2012-11-19 11:49
comments: true
categories: bbbb cloud fisheye crucible ssl nginx crowd
---
Since there's no crowd.properties file for FECRU, you'll have to modify the config.xml file directly: in our case this is in /var/atlassian/application-data/fecru/config.xml, where /var/atlassian/application-data/fecru is specified by $FISHEYE_INST, as per best practice.

There's a `<crowd>` section in the configuration file that simulates the normal crowd.properties file for most other Atlassian applications.

    <crowd auto-add="true" resync="true" refreshExistingUsers="true" sso-enabled="true" resyncPeriod="1 hour" jiraInstance="false">
      <crowd-properties>
        #Sat Nov 03 14:08:22 EDT 2012
        application.password=*************
        application.name=fecru.cloud.local
        crowd.server.url=http\://crowd.cloud.local/crowd/services/
        cookie.tokenkey=crowd_cloud_local.tokenkey
      </crowd-properties>
      <resyncGroupsList/>
    </crowd>

The critical addition that can't be handled from within the Fisheye or Crucible applications is the addition of the cookie.tokenkey setting; to avoid conflicts with other Crowd systems in the Blackboard ecosystem, the Cloud team has separated our tokenkeys out into their own namespace.

