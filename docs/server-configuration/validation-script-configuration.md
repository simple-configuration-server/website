---
layout: default
title: "Validation Script Config"
description: "Use the scs-users.yaml file to define user accounts your deployment"
permalink: "/docs/server-configuration/validation-script-configuration.html"
nav_order: 5
grand_parent: Documentation
parent: Server Configuration
---
# Validation Script Configuration

{: .note }
The 'scs-validate.yaml' file described on this page is only needed if you're
planning to run the [validate.py script](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/docker/validate.py) from the docker image.


If you're planning to use the validation script in your CI/CD pipeline or
inside githooks, you can create the (optional) 'scs-validate.yaml' file, of
which an example is provided below:
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
You need to put this file in the folder used for the SCS_CONFIG_DIR environment
variable, just like with 'scs-configuration.yaml'.

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

# 1 Endpoints
The `endpoints` property allows you to define test/validation
configurations for one or more endpoints. Just like with the 'from_paths' of
scs-users.yaml, wildcards can be used to define endpoints. Note that if
'endpoints' is omitted from the configuration, or if some endpoints are not
defined under 'endpoints', these are validated using the default configuration
(Request Method GET; Response status code < 400). If you don't want to validate
an endpoint, you need to specifically disable it, by setting 'false' instead of
an object.

In case an endpoint both matches a specific url in this configuration, as
well as a pattern with a wildcard, both configurations are used, but settings
in the longer pattern/url (more specfic) override settings in the shorter
pattern/url(less specific). In the above example, `/configs/test.yaml` both
matches the pattern as the specific configuration. This means (1) the yaml
format is validated, (2) the Content-Type response header is validated and (3)
the contents are compared to the object given for the 'yaml:' key.

## 2 SCS Configuration
The `scs_configuration` property allows you to override parts of
'scs-configuration.yaml' during validation. The following changes are
applied by default: (1) logs are directed to stdout and  (2) the auth blueprint
is disabled. You can override the default logging configuration for the
validation script by defining a custom `scs_configuration.logs` property.

You can also use this property, as illustrated in the example, to override
custom YAML tag constructors, for example when you're using a cloud-dependent
constructor in your configuration. Note that the built-in `!scs-secret`
tag is overriden by default during validation (it generates random strings) so
secrets are not required for validation.
