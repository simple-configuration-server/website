---
layout: default
title: "Defining users"
description: "Use the scs-users.yaml file to define user accounts your deployment"
permalink: "/docs/server-configuration/defining-users"
nav_order: 5
grand_parent: Documentation
parent: Server Configuration
---
### Defining Users

{: .note }
This section only applies if you're using the default user authentication
system (More info [here](./config-file#4-auth-configuration))

The 'scs-users.yaml' file contains the list of users of your SCS deployment.
Set the 'auth.options.users_file' property in your 'scs-configuration.yaml' to
point to the location of this file.

A simple scs-users.yaml, containg only 1 user, could look like:
```yaml
- id: example-user
  token: !scs-secret 'scs-tokens.yaml#example-user'
  has_access:
      to_paths:
        - /configs/*
      from_networks:
        - 127.0.0.1/32
```
All properties are required, as defined in the [schema](https://github.com/simple-configuration-server/simple-configuration-server/blob/master/scs/schemas/scs-users.yaml).

You always have to whitelist both the 'paths' as well as the
'from_networks' for each user. You can use '*' as a wildcard character in the
paths. The 'from_networks' have to be a subset of the globally allowed
IPs that are defined at `auth.options.networks.whitelist` in
scs-configuration.yaml. If you want to re-use parts like 'from_networks' for
multiple users, use [YAML anchors](https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html#anchors).

If you want to put your scs-users.yaml in a git repository, seperate the
secrets as in the example above. Note that the
!scs-secret YAML tag refers to the `auth.options.directories.secrets` rather
than `directories.secrets` (though you can set these to the same value, as
described [here](./config-file#4-auth-configuration)).
