---
layout: default
title: "Features"
description: "A server for hosting configuration files and variables that's designed to be simple to deploy and use"
permalink: "/"
nav_order: 1
---
# Simple Configuration Server (SCS)

{: .fs-6 .fw-300 }
A Server for hosting configuration files and variables that's designed to
be simple do deploy and use

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
  allows you to securely _version-control your configuration using Git_.
* **POST requests with template variables**: Clients can send one or more
  template variables via POST requests, to include server-specific configuration
  parameters in returned configuration files, such as paths to specific
  directories on machines.
* **Simple User Authentication**: The built-in user authentication system
  uses a simple file-based configuration, but allows fine-grained control over
  the urls users can access, and the IP-addresses/subnets they can access them
  from.
* **Extendability**: The SCS core functionality can be further extended at
* runtime, by configuring any of the following in the configuration file:
    1. A custom Flask blueprint for authentication, to replace the
       default SCS user authentication system mentioned above
    2. Additional YAML tag constructors, with functions you can use to
       extend the YAML language used in scs-env files.
    3. Additional Flask blueprints that can pre- or post-process the requests
       and responses of the server.
    4. Additional Jinja2 template extensions, allowing you to use custom
       functions and tags inside your configuration file templates.
