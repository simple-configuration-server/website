---
layout: default
title: "Server Configuration"
description: "Describes the configuration options for the server"
permalink: "/docs/server-configuration.html"
nav_order: 1
parent: Documentation
has_children: true
---
# Server Configuration
To serve your configuration files/variables using SCS you need:

1. The scs-configuration.yaml file, containing the settings of SCS
1. A config directory, containing the files/templates that should be served
2. A common directory, containing yaml files with common configuration elements
   (Can be empty)
3. A secrets directory, containing secrets, such as passwords, used in
   configuration files (Optional)
4. scs-users.yaml, containing SCS user definitions (Optional; Only if the
   built-in auth module is used)
5. scs-validate.yaml, containing the validation script settings (Optional; Only
   if validate.py is used to validate the configuration)
