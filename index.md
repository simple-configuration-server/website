---
layout: default
title: "Features"
description: "A server for hosting configuration files and variables that's designed to be simple to deploy and use"
permalink: "/"
sort_order: 1
---
# Simple Configuration Server (SCS)
With SCS, you can host directories with configuration files and variables
directly over HTTP(S), like any other file-server. Unlike simple file-servers
however, SCS offers the following:

* **Jinja2 Templates**: In addition to hosting simple files, templates allow
  you to dynamically render configuration files/variables. Thereby adding the
  ability to define variables used accross multiple endpoints centrally, and
  reference them in multiple configuration file templates.
* **Seperation of Secrets**: Using the `!scs-secret` YAML-tag you can reference
  secrets stored in a seperate location. In combination with the templating
  system you can completely seperate your configuration from secrets. This
  allows you to securely version-control your configuration using Git.
* **POST requests with template variables**: Clients can send one or more
  template variables via POST requests, to include server-specific configuration
  parameters in returned configuraiton files, such as path to specific
  directories on machines.
* **Simple User Authentication**: The built-in user authentication system
  uses a simple file-based configuration, but allows fine-grained control over
  the urls users can access, and the IP-addresses/Subnets they can access them
  from.
* **Extendability**: At it's core, SCS is already a powerfull system, but
  inside the 'scs-configuration.yaml' file you can further configure:
    1. A custom Flask blueprint to use for authentication, instead of the
       system mentioned above
    2. Additional YAML tag constructors
    3. Additional Flask blueprints
    4. Additional template extensions for Jinja2