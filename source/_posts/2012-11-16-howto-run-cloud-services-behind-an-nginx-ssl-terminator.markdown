---
layout: post
title: "HOWTO run cloud services behind an nginx SSL terminator"
date: 2012-11-16 11:58
comments: true
categories: 
---
This entire experiment was a result of the firewall configurations on stack-extdemo01, which only allows traffic through the VIP on 8080, 9540 and 9530. While we could have certainly worked with Managed Hosting to further open the firewall, or terminate SSL at the load balancer, there was a great opportunity to experiment with using nginx to terminate the SSL connections and proxy requests through to a separate port for the big three services. Not only did this work around the infrastructure restrictions for stack-extdemo01.blackboard.com/stack-extdemo01.mhint, but it provided a model for allowing developers to locally terminate SSL traffic for cloud services.

As always, and especially with respect to local development environments, SSL is a bit of a pain in the ass. This post aims to describe steps required to use nginx as the SSL terminator for both 'demo-class' stack servers as well as local development environments.

Since there are only four services (and three application server instances in development, as Connect and PIS typically run in the same Tomcat container) that are "public", this only focuses on configuring those services under SSL. However, certain non-public services (Profile, notably) require configuration to communicate with the secure services.

Architecture
Service Configuration
Router Service
Profile Service
Graph & Wall Services
Spaces
UI Service
Connect Service
Webapp Configuration
Software Updates Service (PIS)
Webapp Configuration
Tomcat
Nginx & SSL Termination
tl;dr
Nginx on Ubuntu
Nginx on OS X
Nginx Configuration
Using port 80 with multiple virtual hosts
SSL Certificates
Java keystore
Local System Configuration
/etc/hosts
Simulating non-stacked cloud services environments with /etc/hosts
Learn
Java cacerts and the *.cloud.local certificate
Architecture
Since there's only three/four services that expose public endpoints (router, UI, connect, PIS), I've simplified the reconfiguration by simply moving the default ports up one, leaving the original ports for nginx to listen on. So, where by default Tomcat runs on 8080, Router on 9540 and UI on 9530, I've moved them to 8081, 9541 and 9531 respectively. More information on the port forwarding below, but references to 8081, 9541 and 9531 reflect the new non-secure ports that nginx forwards to.

Service Configuration
I'm assuming someone who uses ./configure more often than I can weigh in on how much of this is easily configurable.

Router Service
Critically, application.conf must be updated to set bb.useSSL=true, and router-paths.properties needs to be updated to point all the Tomcat paths to https:

...
v1/sites/[^/]+/reports=https://public.cloud.local:8080/connect
...
connect=https://public.cloud.local:8080
reports=https://public.cloud.local:8080/connect
Everything else remains pointed to http:// calls. Not certain about router-paths-test.properties, as its unused in production.

Profile Service
The reference to UI in Profile configuration should be reset to use the secure link:

# Bb Cloud URLs
bb.cloud.graph="http://localhost:9500"
bb.cloud.wall="http://localhost:9510"
bb.cloud.ui="https://public.cloud.local:9530"
Graph & Wall Services
Any configuration files remain as non-secure http://...

Spaces
tbd

UI Service
The "external hosts" section of config.json should point to the public secure URLs:

...
"externalHosts":{ "assets":"https://public.cloud.local:9530",    
                  "ui":"https://public.cloud.local:9530",    
                  "restApi":"https://public.cloud.local:9540" },
"internalHosts": { "restApi":"http://localhost:9541"  },
...
Connect Service
Connect configuration should point the 'routerUri' parameter to the public, secure URL; profile and space URLs can remain non-secure:

routerUri=https://public.cloud.local:9540
profileUri=http://localhost:9520
spaceUri=http://localhost:9550
Webapp Configuration
The useSSL parameter in config-connect-service-default.properties must be set to true, or overridden.

Software Updates Service (PIS)
I didn't actually do anything with this, but I'm assuming that the connectServiceUri should move from localhost over http:// to https://public.cloud.local.

Webapp Configuration
The useSSL parameter in config-product-info-service-default must be set to true, or overridden.

Tomcat
Tomcat configuration needs to be updated to reflect SSL proxying:

<Connector port="8081" protocol="HTTP/1.1"
           connectionTimeout="20000"
           scheme="https" 
           secure="true" 
           proxyPort="8080" />
Nginx & SSL Termination
Here's where the shit starts to hit the fan, and configuration starts to vary widely between Ubuntu and OS X.

tl;dr
Nginx doesn't support chunked transfer requests natively, and requires a module to be compiled in; Ubuntu has a precompiled version of nginx with the HttpChunkinModule, and I'm working on a brew recipe for the same. Additionally, our pseudo-production systems that are doing this are using (for now) real addresses (blackboard.com, cloud.bb); using cloud.local addresses (whether for stack-qa-next.cloud.local or a locally-defined /etc/hosts entry) will require add'l steps to get the JVM to trust the *.cloud.local certificates. Other than that, this works great.

Nginx on Ubuntu
The nginx-extras package happens to have the HttpChunkinModule precompiled in with the server; apt-get install nginx-extras is all that's needed to replace the standard .deb package with the one containing the chunked transfer encoding module.

Nginx on OS X
Until I finish the brew recipe, you can download the source for nginx and the chunkin-module, and compile with:

./configure --prefix=/usr/local/nginx-with-chunkin --add-module=../chunkin-nginx-module-0.23rc2 --with-http_ssl_module --with-pcre --with-ipv6 --with-cc-opt=-I/usr/local/include --with-ld-opt=-L/usr/local/lib --conf-path=/usr/local/nginx-with-chunkin/etc/nginx/nginx.conf --pid-path=/usr/local/nginx-with-chunkin/var/run/nginx.pid --lock-path=/usr/local/nginx-with-chunkin/var/run/nginx.lock --http-client-body-temp-path=/usr/local/nginx-with-chunkin/var/run/nginx/client_body_temp --http-proxy-temp-path=/usr/local/nginx-with-chunkin/var/run/nginx/proxy_temp --http-fastcgi-temp-path=/usr/local/nginx-with-chunkin/var/run/nginx/fastcgi_temp --http-uwsgi-temp-path=/usr/local/nginx-with-chunkin/var/run/nginx/uwsgi_temp --http-scgi-temp-path=/usr/local/nginx-with-chunkin/var/run/nginx/scgi_temp

Nginx Configuration
In either case, the standard nginx configuration (/etc/nginx on Ubuntu, or /usr/local/etc/nginx for brew-installed Nginx on OS X) should automatically reference configuration files in $NGINX_CONF/sites-available. Here's an example of the stack configuration file for nginx that listens on the three appropriate ports, terminates SSL and forwards the requests to the internal ports:

server
{
  listen                      8080;
  server_name                 public.cloud.local;
  
  access_log                  /var/log/nginx/access.log;
  
  ssl                         on;
  ssl_certificate             /etc/ssl/certs/wildcard.cloud.local.cer;
  ssl_certificate_key         /etc/ssl/private/wildcard.cloud.local.key;
  
  client_max_body_size        128M;

  chunkin on;
  error_page 411 = @my_411_error;
  location @my_411_error 
  {
    chunkin_resume;
  }
  
  location / 
  {
     # needed to forward user's IP address
     proxy_set_header X-Real-IP $remote_addr;
     
     # needed for HTTPS
     proxy_set_header X-Forwarded-Proto https;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Host $http_host;
     proxy_redirect off;
     proxy_max_temp_file_size 0;
     
     proxy_pass http://localhost:8081;
   }
}

server
{
  listen                      9540;
  server_name                 public.cloud.local;
  
  access_log                  /var/log/nginx/access.log;
  
  ssl                         on;
  ssl_certificate             /etc/ssl/certs/wildcard.cloud.local.cer;
  ssl_certificate_key         /etc/ssl/private/wildcard.cloud.local.key;
  
  client_max_body_size        128M;

  chunkin on;
  error_page 411 = @my_411_error;
  location @my_411_error 
  {
    chunkin_resume;
  }
  
  location / 
  {
     # needed to forward user's IP address
     proxy_set_header X-Real-IP $remote_addr;
     
     # needed for HTTPS
     proxy_set_header X-Forwarded-Proto https;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Host $http_host;
     proxy_redirect off;
     proxy_max_temp_file_size 0;
     
     proxy_pass http://localhost:9541;
   }
}

server
{
  listen                      9530;
  server_name                 public.cloud.local;
  
  access_log                  /var/log/nginx/access.log;
  
  ssl                         on;
  ssl_certificate             /etc/ssl/certs/wildcard.cloud.local.cer;
  ssl_certificate_key         /etc/ssl/private/wildcard.cloud.local.key;
  
  client_max_body_size        128M;
  
  chunkin on;
  error_page 411 = @my_411_error;
  location @my_411_error 
  {
    chunkin_resume;
  }

  location / 
  {
     # needed to forward user's IP address
     proxy_set_header X-Real-IP $remote_addr;
     
     # needed for HTTPS
     proxy_set_header X-Forwarded-Proto https;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Host $http_host;
     proxy_redirect off;
     proxy_max_temp_file_size 0;
     
     proxy_pass http://localhost:9531;
   }
}
Using port 80 with multiple virtual hosts
Conversely, since nginx can listen on port 80 if desired, you could set up three or four or more virtual hostnames in /etc/hosts (see below), allowing nginx to listen simultaneously for all the services desired on 80 and proxy through to their backend ports. In this way, you could more closely simulate a non-stacked environment, with https://ui.cloud.local, https://api.cloud.local, https://connect.cloud.local, https://softwareupdates.cloud.local, http://profile.cloud.local, etc., where each virtualhost in nginx forwards to the appropriate localhost:port for the running service.

SSL Certificates
As indicated in the nginx configuration above, both the public and private keys for the *.cloud.local certificate need to be made available to nginx; these keys are available on demand. TODO: Link to keyfiles

Java keystore
In addition to providing the certificate and key to nginx, since the *.cloud.local certificates don't have a valid chain, they'll have to be added to the JRE keystore. TODO: link to, or provide documentation on, converting and adding the certificate to the Java keystore.

Local System Configuration
/etc/hosts
Probably only applicable to local OS X or Ubuntu development environments, as production-class systems will already have valid DNS records matching the *.cloud.bb or *.blackboard.com certificates and virtual hosts in nginx.

Since nginx is using virtual hosts, and the *.cloud.local certificates will only be valid for a *.cloud.local hostname, you'll have to add an /etc/hosts entry for localhost as a .cloud.local name; the above configuration uses public.cloud.local for the virtual host names – you can choose whatever name you want provided you update the virtual host names in the nginx configuration, and be sure that all references to your local services (both in the services themselves, as above, and in Learn) use the .cloud.local address if SSL is desired.

Simulating non-stacked cloud services environments with /etc/hosts
As noted above, you could easily define ui.cloud.local, api.cloud.local, profile.cloud.local, graph.cloud.local, etc. in /etc/hosts allowing you to specify in the nginx configuration virtualhosts that listen on port 80, eliminating the need to pass a port into configuration files and URLs.

Learn
Once you have the SSL certificates in the JRE keystore, several additional options become readily available for Learn configuration. You can directly enable SSL on Learn, if desired, or you can use a similar model and terminate SSL for Learn at the nginx layer by including another nginx configuration, similar to:

server 
{
  listen                      80;
  server_name                 learn.local;
  # rewrite                     ^(.*) https://$server_name$1 permanent;

  ## Everything here disappears on SSL termination.
  access_log                  /usr/local/var/log/nginx/access.learn-local.log;

  client_max_body_size        128M;

  location / 
  {
    # needed to forward user's IP address
    # 	proxy_set_header X-Real-IP $remote_addr;

    # needed for HTTPS
    # proxy_set_header X-Forwarded-Proto https;
    # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # proxy_set_header Host $http_host;
    # proxy_redirect off;
    # proxy_max_temp_file_size 0;

    proxy_pass http://localhost:8000;
  }
}
Java cacerts and the *.cloud.local certificate
See HOWTO configure Learn to trust the *.cloud.local certificate for SSL cloud services

TODO: document the 301/302 rewrite rule, including options for using it for cloud services
TODO: document Learn bb-config.properties changes, and PushConfigUpdates.sh requirement