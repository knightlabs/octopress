---
layout: post
title: "Roles in Ansible 1.2"
date: 2013-04-27 22:27
comments: true
categories: 
---
With 1.2, Ansible is making a push into concepts pulled from Puppet and cfengine by creating a mechanism for a configuration by convention that leverages all the strengths of Ansible: simple, readable, idempotent sets of actions.

The original goals of ansible were to bring the best of both the prescriptive CM tools (Puppet, Chef, cfengine) and descriptive deployment tools (capistrano, ant, shell scripts) in a single DSL. To date, most of what we've used ansible for is to replace a mash of shell and capistrano scripts with a set of straightforward playbooks, indicating, for any particular service, the exact steps to bring an Ubuntu 12 system up to speed with that service.

This has applied regardless of whether we're deploying Cloud services (graph_install.yml) or backend services (mongodb_install.yml). The playbooks have guaranteed that for each particular environment, the services are correctly installed. A typical playbook is run something like this: {{ansible-playbook aws/production/graph_install.yml --extra-vars="version=<build_version>" --tags=<some_tag> --limit="<particular_hosts>"}}. Nb. This is pseudo-code, first, and second, production deploys aren't run this way at all, there's a script that does everything to simplify things, but the theory is sound.

But in addition to being able to install _our_ services, the aws/production playbooks include ones for installing mongo, redis, cassandra, et al. This has worked spectacularly to date in that various mongo versions, configurations, and so on, that vary between, for instance, production and beta and internal servers, have been entirely encapsulated within a single directory structure.

However, it has led to a significant amount of duplication of effort: There are, by quick estimation, at least seven playbooks that install and configure mongodb across AWS production, AWS beta, AWS performance, internal CI, internal test and AWS xplor production, beta and internal servers. Some of this is due to variations in required versions, some due to variations in the configurations, some due to the different environments.

The nice thing about Ansible is it makes keeping things separate manageable, because the playbooks themselves are simply a straight list of commands to execute, rather than a complicated mess of include or require statements, with various recipes in various versions, etc.

But, its suboptimal.

Roles may be a way to incrementally improve the overall structure of our ansible playbooks, by providing a common set of plays for common infrastructure.

At their core, a role in ansible is simply a configured-by-convention exploded playbook: rather than including vars and tasks in a single playbook, roles break out as follows:

{noformat}
role
 |-- vars
 |-- tasks
 |-- files
 |-- templates
 |-- handlers
{noformat}

Then, to have a playbook use the role, its as simple as:

{noformat}
# ansible-playbooks/test-playbook.yml
---
- user: cloud
  sudo: true
  hosts: <ansible-hosts group>
  roles: 
  - role1
  - role2
{noformat}

By cross-cutting role1 and role2 into the hosts group through the playbook, when {{ansible-playbook test-playbook.yml}} is executed, role1 and role2 tasks will be executed with the variable set in the role vars file, notify handlers in the role handlers files, using templates and files in the role templates and files directories, but still allow additional tasks to execute in the playbook as needed.

See ansible-playbooks/stack-playbooks/roles/nginx for an example of a role built for the stack servers.

In the end, and having used this week-old code for under 24 hours at this point, my initial recommendation is that we continue to leverage the straight-up playbooks for all new development, and once stable, make determinations on a case-by-case basis as to what makes sense to Role-ize.

Obviously, there's open questions about various configurations (answerable, but not here, at least not yet), versioning (git branches v. node08/node10 roles, etc), extending roles to maintain _install, _uninstall and _service.yml playbooks, how to leverage tags in the playbooks into the roles, and so on. More to come.