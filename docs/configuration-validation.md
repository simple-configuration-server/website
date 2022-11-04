---
layout: default
title: "Configuration Validation"
description: "Use the validate.py script and scs-validate.yaml to validate SCS configurations"
permalink: "/docs/configuration-validation"
nav_order: 2
parent: Documentation
---
# Configuration Validation
The official Docker images include the 'validate.py' script, that let's you
validate your SCS configuration (for example inside a GIT repository) without
secrets present.

## 1 Running validate.py
Like with the normal server, you only need to set the `SCS_CONFIG_DIR` variable
to the directory containing your 'scs-configuration.yaml'. Optionally, you can
add a 'scs-validate.yaml' configuration file to this directory, that defines
configuration options for the validation script (See [section 2](#2-scs-validateyaml-configuration-file)
below)

The script is meant to be run from inside the official SCS docker container,
to which you've either mounted your configuration using Docker volumes or
bind-mounts, or from which you've build an image that includes your
configuration.

An example of running the validation script in case of the latter, can be
found in the [GitHub Workflow](https://github.com/simple-configuration-server/example-configuration/blob/main/.github/workflows/main.yml)
for the example-configuration repository.

## 2 scs-validate.yaml Configuration File
An example of the contents of a 'scs-validate.yaml' file is provided below:
```yaml
endpoints:  # See section 1 below
  /configs/cluster_name: false
  /configs/custer_name_redirect:
    request:
      method: GET
    response:
      status: 301
      headers:
        Location: /configs/cluster_name
      text: 'validate against this'
  /configs/test.json:
    response:
      json:
        validate: true
        against: true
        this: true
        object: true
  /configs/test.yaml:
    response:
      yaml:
        validate: true
        against: true
        this: true
        object: true
  /configs/*.yml:
    response:
      format: yaml
      headers:
        Content-Type: application/x-yaml

scs_configuration:  # See section 2 below
  extensions:
    constructors:
      - name: scs.yaml.SCSSimpleValueConstructor
        options:
            tag: '!cloud-dependent-tag'
            value: A default value to return for this tag

handle_errors: false

allow_secrets: false
```
The full schema for this file can be found [here](docker/scs-validate.SCHEMA.yaml).

{: .note }
Since all properties in this file are optional, validate.py can
also be run if this file is omitted. It will then use the default settings
from the schema.

The `handle_errors` property defines if internal errors should be handled or
raised. In case you want to test if specific endpoints returns a 500 status
code , set this to 'true'.

Set `allow_secrets` to true, in case the configuration you're testing includes
secrets. Note that this is not advised, and therefore this is set to 'false' by
default.

### 2.1 Endpoints
The `endpoints` property allows you to define test/validation
configurations for one or more endpoints. Just like with the 'from_paths' of
scs-users.yaml, wildcards can be used to define endpoints. Note that if
'endpoints' is omitted from the configuration, or if some endpoints are not
defined under 'endpoints', these are validated using the default configuration
(GET request; Response status code < 400). If you don't want to validate
an endpoint, you need to specifically disable it, by setting 'false' instead of
an object.

In case an endpoint both matches a specific url in this configuration, as
well as a pattern with a wildcard, both configurations are used, but settings
in the longer pattern/url (more specfic) override settings in the shorter
pattern/url(less specific). In the above example, `/configs/test.yaml` matches
both the general pattern `/configs/*.yml` as well as the specific pattern
`/configs/test.yaml`. This means (1) the yaml format is validated (general
pattern), (2) the Content-Type response header is validated (general pattern)
and (3) the contents are compared to the object given for
the 'yaml:' key (specific pattern).

### 2.2 SCS Configuration
The `scs_configuration` property allows you to override parts of
'scs-configuration.yaml' during validation. The following changes are
applied by default:

1. Logs are directed to stdout
2. The auth blueprint is disabled.
3. The built-in `!scs-secret` constructor is replaced, to generate random
   strings (So secrets are not required at validation time)

You can also use this property, as illustrated in the example, to override
custom YAML tag constructors, for example when you're using a cloud-dependent
constructor in your configuration.
