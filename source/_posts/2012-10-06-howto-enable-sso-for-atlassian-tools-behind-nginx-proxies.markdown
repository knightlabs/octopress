---
layout: post
title: "HOWTO Enable SSO for Atlassian Tools Behind Nginx Proxies"
date: 2012-10-06 22:23
comments: true
categories: 
---
Since we use a lightweight nginx proxy in front of all our enterprise applications to allow simple port forwarding via proxy (and eventually, SSL termination at the proxy rather than dealing with Java keystores), there's a couple additional steps that have to go on in order for SSO to work.

This assumes you've already configured the application to speak to Crowd, and are merely looking to transparently handle SSO.

*JIRA*: 

Modify {{/usr/local/jira/atlassian-jira/WEB-INF/classes/seraph-config.xml}} to replace the Seraph authenticator from {{<authenticator class="com.atlassian.jira.security.login.JiraSeraphAuthenticator"/>}} to {{<authenticator class="com.atlassian.jira.security.login.SSOSeraphAuthenticator"/>}}, and bounce JIRA.

Additionally, to ensure that Crowd token cookies don't conflict with tokens from, say, the .pd.local domain, JIRA needs to change its crowd.properties configuration to create a unique-to-cloud cookie name by adding {{cookie.tokenkey=crowd_cloud_local.tokenkey}} to the crowd.properties file in {{/usr/local/jira/atlassian-jira/WEB-INF/classes/crowd.properties}}. Additionally, change the {{session.validationinterval}} parameter to 5 from 0, to only check against Crowd every 5 minutes.

Next, ensure that the nginx configuration handles passing the proxy headers correctly:

{noformat}
  location / 
  {
     # needed to forward user's IP address
     proxy_set_header X-Real-IP $remote_addr;
     
     # needed for HTTPS
     proxy_set_header X-Forwarded-Proto http;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Host $http_host;
     proxy_redirect off;
     proxy_max_temp_file_size 0;
     
     proxy_pass http://jira.cloud.local:8080;
   }
{noformat}

An aside about nginx configuration: we follow a fairly standard model in which {{/etc/nginx/nginx.conf}} loads configuration files from {{/etc/nginx/sites-enabled}} which are simply symlinks to {{/etc/nginx/sites-available}}, allowing distribution of multiple configuration files in sites-available, but only turning on the vhosts for the symlinked configurations.

In the event that SSL termination is enabled, the {{proxy_set_header X-Forwarded-Proto http;}} line would become 'https' to ensure that the forwarded protocol is correct.

Now bounce nginx.

Finally, Crowd needs to be made aware of the fact that JIRA is coming through a proxy: login to Crowd as an administrator and add the IP of the JIRA server (in this case "10.103.37.101" to the list of Trusted Proxies in Crowd.

Profit.

*Bamboo*:

Almost entirely the same set of steps: Update Bamboo's nginx configuration, alter the seraph-config.xml in {{/usr/local/bamboo/webapp/WEB-INF/classes}} to use the Crowd SSO authenticator, change {{/var/atlasssian/application-data/bamboo/xml-data/configuration/crowd.properties}} to reflect the {{cookie.tokenkey=crowd_cloud_local.tokenkey}} and {{session.validationinterval}} changes, ensure nginx configuration is updated as above, and add Bamboo's IP to Crowd's trusted proxy list.

*Crowd*

In order to ensure SSO to Crowd itself, same steps: Update crowd.properties in {{/var/atlassian/application-data/crowd}}, update nginx configuration, and to be safe, add the Crowd IP and localhost to Trusted Proxies. The only thing you _don't_ have to do for Crowd is deal with Seraph, as its set up by default to SSO.

*Fisheye/Crucible*

See [~jknight:/2012/11/19/HOWTO Enable SSO for Fisheye or Crucible when using a non-standard Crowd cookie token name] and don't forget the damn Crowd proxy IP.