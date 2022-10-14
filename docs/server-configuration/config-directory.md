---
layout: default
title: "Config Directory"
description: "Describes the layout of the Config Directory"
permalink: "/docs/server-configuration/config-directory"
nav_order: 2
grand_parent: Documentation
parent: Server Configuration
---
# Config Directory

{: .highlight }
Use the 'directories.config' variable inside scs-configuration.yaml to set
the location of this directory

The contents of this directory are served by the SCS, with the exception of
files with names that end with 'scs-env.yaml'. These files contain
configuration variables for specific endpoints, or entire directories.
The URL structure of the SCS is derived from the folder structure inside your
config directory.

## 1 *scs-env.yaml Files
Using scs-env.yaml files is optional, but allows you to configure the
variables for the templating system, configure the validation of requests and
responses of each endpoint. Below is a full example of all options that can be
set in a *scs-env.yaml file:
```yaml
template:
  context:  # Variables available inside templates
    variable: value
    common_variable: !scs-common 'global.yaml#var_array.[5]'
    secret: !scs-secret 'system-users.yaml#user.password'
  rendering_options:  # Override the default jinja rendering options from scs-configuration.yaml
    lstrip_blocks: false
  enabled: true  # To host simple files, templating can also be disabled

request:
  methods:  # Allowed methods
    - GET
    - POST
  schema:   # Schema used to validate POST request JSON body
    type: object
    additionalProperties: false
    required:
      - memory_size
    properties:
      memory_size:
        type: integer

response:
  status: 200  # Set the default status of the response
  headers:  # Set headers on the response
    Content-Type: text/plain
```
This files should adhere to the schema found [here](https://github.com/simple-configuration-server/simple-configuration-server/blob/master/scs/schemas/scs-env.yaml),
which also includes more extensive descriptions of each property. Note that
this schema also defines the defaults for each property, used when properties
are not defined. All properties in scs-env files are optional, but empty
scs-env files should be removed.

In case you simply want to host the files in your config directory,
without using any of the templating features, you can place the
following scs-env.yaml file in the root of the config dir:
```yaml
template:
  enabled: false
```

As mentioned, *scs-env.yaml files can apply to entire directories or to
specific endpoints (files) only. Multiple scs-env files can apply to same
endpoint, in which case properties defined in the more specific scs-env.yaml
file, override properties in the less specific one. For example, consider the
following example:
```
.
├── root_endpoint.json
├── root_endpoint.json.scs-env.yaml
├── scs-env.yaml
└── subdirectory
    ├── scs-env.yaml
    ├── sub_endpoint2
    ├── sub_endpoint2.scs-env.yaml
    └── sub_enpoint
```
The './scs-env.yaml' applies to all endpoints and the './subdirectory/scs-env.yaml'
applies to all endpoints in the subdirectory. For example for
'./root_endpoint.json' the following scs-env files are used:
1. './scs-env.yaml'
2. './root_endpoint.json.scs-env.yaml'

Similairly, the following apply to './subdirectory/sub_endpoint2':
1. './scs-env.yaml'
2. './subdirectory/scs-env.yaml'
3. './subdirectory/sub_endpoint2.scs-env.yaml'

Values of type Object (dict) of more specific scs-env files update the contents
of less specific ones. Values with other data types are replaced, in case a
more specific scs-env file defines them also. For example, considering the
scs-env files for './root_endpoint.json' have the following contents:
1. **./scs-env.yaml**
   ```yaml
   template:
     context:
       key1: general
       key2: general
   response:
     status: 200
     headers:
       Content-Type: text/plain
   ```
2. **./root_endpoint.json.scs-env.yaml**
   ```yaml
   template:
     context:
       key2: specific
   response:
     status: 418
     headers:
      X-TeaType: Rooibos
   ```

Then the configuration for the endpoint would look like (excl. default values):
```yaml
template:
  context:
    key1: general
    key2: specific
response:
  status: 418
  headers:
    Content-Type: text/plain
    X-TeaFlavor: Rooibos
```

## 2 URL structure
All files inside the config directory of which the names do not end with
'scs-env.yaml', are used as server endpoints. These files are served under the
/configs/ urls. The example file structure above would expose
the following endpoints:
```
/configs/root_endpoint.json
/configs/subdirectory/sub_endpoint
/configs/subdirectory/sub_endpoint2
```
