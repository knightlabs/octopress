---
layout: post
title: "jetty on the colorstacks"
date: 2013-08-26 04:44
comments: true
categories: bbbb cloud colorstack jetty ssl ops
---
Over the weekend, I finished the work to replace Tomcat on the colorstacks with Jetty. This has a couple of primary benefits:

* Rebuilding connect doesn't hose tests running against productinfo, or partners, or CIS, and so on;
* Jetty is significantly lighter weight and faster to start up, leading to improved build times;
Secondarily, it gave me much better awareness of just how insane Jetty and Jetty documentation are, which is a story for a different time.

Summarizing:

* Tomcat on 8080, previously responsible for Connect, PIS, Partners and CIS, is now disabled and will soon be deleted from the stacks;
* For services requiring a Java web container, each starts its own, on the standard ports defined for Jetty via maven on developer machines (9570 for Connect, 9580 for PIS, 9590 for Partners and 9620 for CIS)
* SSL is still offloaded at the nginx terminators on stack-lb01 and stack-lb02
* Each Jetty instance is directly installed under the cloud.bb/<service> path on the stacks.

# Details:

Jetty is installed via the spectacularly less than pretty manner in which Ansible overrides role specifics:

In ansible-stack-playbooks, looking at something like https://github.com/blackboard/ansible-stack-playbooks/blob/develop/connect_install.yml, you'll see:

    - { role: jetty, jetty__root: /usr/local/cloud.bb/connect/shared, jetty__home: /usr/local/cloud.bb/connect/jetty, jetty__owner: $connect__service, jetty__port: $connect__port, jetty__service: $connect__service, jetty__service_id: $connect__service_id }

Basically, that's overriding the default variables in the Jetty role; connect-base, the role https://github.com/blackboard/ansible-stack-playbooks/blob/develop/roles/connect-base/vars/main.yml, defines:

    connect__service: connect
    connect__service_id: 2042
    
    connect__port: 9570
    connect__jmx_port: 17570

    connect__home: ${stack__cloud_home}/${connect__service}
    
In the end, after running through the connect-base and jetty roles, you end up with a jetty instance installed at /usr/local/cloud.bb/connect/shared with a symlink at the top level as:

    /usr/local/cloud.bb/connect/jetty -> /usr/local/cloud.bb/connect/shared/jetty-distribution-<version>

The critical piece of the puzzle is found in what's by default in $JETTY_HOME/etc/jetty.xml (@ https://github.com/blackboard/ansible-stack-playbooks/blob/develop/roles/jetty/templates/usr_local_jetty_etc_jetty.xml):

If a customizer to handle X-Forwarded-For headers isn't included, everything breaks

    <Call name="addCustomizer">
      <Arg><New class="org.eclipse.jetty.server.ForwardedRequestCustomizer"/></Arg>
    </Call>

This is required to allow Jetty to properly deal with the nginx reverse proxy offloading SSL.

# More Fun

The included-with-Jetty init.d-style startup script attempts to be really helpful, automatically searching various paths decided upon by the Jetty developers. If you're a complete n00b, this is useful. In our case, its a disaster.

Final changes were made in https://github.com/blackboard/ansible-stack-playbooks/blob/develop/roles/jetty/templates/etc_init.d_jetty to include upstart-style headers, define JAVA_HOME properly, set the JETTY_HOME and JETTY_RUN directories to avoid overlapping PIDs, and finally fix the fact that Jetty attempts to define a local variable '$JAVA' with JAVA=$(which java).

h1. Three Conclusions

1. The init.d script needs to be stripped bare and rebuilt for our needs; its a 609 line startup script attempting to make things simple on non-admins; it sucks beyond that.
2. I'm stripping out a lot of the default Jetty configurations; reworking the entire configuration tree seems like a valid task at this point
3. Given that we're basically only using the lib directory at that point, the idea of using Jetty from 'source' in the model we do with other stack technologies is getting ponderous. Options range from committing the libs as binaries to Github, installing as source and removing everything except the libs, inverting the execution back into the services themselves (a la Maven-style), etc.