---
layout: default
title: "Server Configuration File"
description: "Describes the available options for scs-configuration.yaml"
permalink: "/docs/server-configuration/config-file"
nav_order: 1
grand_parent: Documentation
parent: Server Configuration
---
# Server Configuration File
Before starting the server, you must create a 'scs-configuration.yaml' file
containing the server configuration, and set the `SCS_CONFIG_DIR` environment
variable to point to the directory that contains the file. This page describes
the format of the 'scs-configuration.yaml' file, and the available
configuration options therein.

The following is an example of a complete configuration:
```yaml
directories:  # See the pages under 'Server Configuration' for each directory
  common: !scs-expand-env ${SCS_CONFIG_DIR}/common
  config: !scs-expand-env ${SCS_CONFIG_DIR}/config
  secrets: &secrets-dir !scs-expand-env ${SCS_CONFIG_DIR}/secrets

environments:  # See section 1 below
  cache: true
  reject_keys_containing_dots: true

templates:  # See section 2 below
  cache: true
  validate_on_startup: true
  rendering_options: {}

logs:  # See section 3 below
  audit:
    file:
      path: /var/log/scs/audit.log.jsonl
      max_size_mb: 1
      backup_count: 5
      level: INFO
  application:
    stdout:
      level: INFO
  # Optionally, define an alternative string to use for the 'source' field in
  # the logs (default: scs)
  # source_name: scs

auth:  # See section 4 below
  # By default the scs.auth.bp flask blueprint is used for authentication
  # blueprint: scs.auth.bp
  options:
    users_file: !scs-expand-env ${SCS_CONFIG_DIR}/scs-users.yaml
    directories:
      secrets: *secrets-dir
    networks:
      private_only: true
      whitelist:
      - 127.0.0.1/32
      - 172.16.134.0/24
    max_auth_fails_per_15_min: 10

extensions:  # See section 5 below
  constructors: []
    # - name: scs.dummy_constructors.GeneratePhraseConstructor
    #   options:
    #     startswith: 'This is great because:'
  blueprints: []
    # - name: scs.dummy_blueprint.bp
    #   options:
    #     print_all_requests: true
  jinja2: []
    # - name: scs.jinja_extensions.GreatExtension
```
Note that only the 'directories', 'logs' and 'auth' top-level properties are
required. A full description of the format of this file, including the defaults
that are used if properties are not defined, can be found in the
[scs-configuration.yaml schema](https://github.com/simple-configuration-server/simple-configuration-server/blob/main/scs/schemas/scs-configuration.yaml).

## 1 Environment Configuration
By default `environments.cache` is true, meaning scs-env.yaml files are only
loaded once at startup. If you make changes to these files, with this set to
true, you'll have to restart the server.

The `environments.reject_keys_containing_dots` is discussed on
[this page](./common-directory).

## 2 Templates Configuration
By default, `templates.cache` is true, meaning that if you change endpoint
files for which templating is enabled (default), you need to restart the server
for the changes to take effect.

If `templates.validate_on_startup` is enabled (default), all environment files
and templates are loaded once, to verify that the syntax of these files are
correct. Please note that if you have endpoints which depend on variables
being passed via POST requests, without a backup value defined in the
`template.context`, loading of these templates, and therefore server startup,
may fail.

`templates.rendering_options` sets the default rendering options for jinja2
templates (See options [here](https://jinja.palletsprojects.com/en/3.0.x/api/#jinja2.Environment.overlay)).
Note that the [configuration file schema](https://github.com/simple-configuration-server/simple-configuration-server/blob/main/scs/schemas/scs-configuration.yaml)
already defines global defaults. If you want to override the defaults, you need
to specify new values for these. Setting an empty object for this will still
cause the default settings to be applied. If you need to use alternative
rendering_options for specific endpoints, you can also define
`template.rendering_options` in scs-env.yaml files, which will update any
globally defined rendering options for specific endpoints ([More info](/config-directory.html#1-scs-envyaml-files)).

## 3 Logs Configuration
The SCS creates 2 types of logs:
1. `audit` logs: These contain audit events of the following types (stored
   in the event.type field):
   * `config-loaded`: An authorized user loaded a config file
   * `secrets-loaded`: An authorized user loaded secrets
   * `unauthenticated`: An unauthenticated client tried to access an endpoint
   * `rate-limited`: An IP address was rate-limited because of too many
     authentication attempts
   * `unauthorized-ip`: An authenticated user tried to access the SCS from an
     unauthorized ip address
   * `unauthorized-path`: An authenticated user tried to access an unautorized
     path
2. `application` logs: These contain general application logs, such as internal
   errors of the SCS

{: .note }
The set of audit event types listed under (1) can be extended by
third party modules, as described [here](/extensions/integration#2-logging)

The SCS produces logs in the JSON-lines format by default. Please see the
AppLogFormatter and AuditLogFormatter classes in the
[logging module](https://github.com/simple-configuration-server/simple-configuration-server/blob/main/scs/logging.py)
for the exact format of these.

Using the configuration file you can set the logs to output to a log
file, which is auto-rotated by SCS. Use this in combination with a log
centralization agent, like Elastic Filebeat, to centralize your logs. An example
of how to do this can be found [here](/docs/deployment/log-centralization)

Alternatively you can simply output the logs to the console, by setting the
stdout option. In this case you can for example see the logs via the
`docker logs` command.

## 4 Auth configuration
Use the `auth.blueprint` setting to define the flask blueprint you want to
use for user-authentication. When omitted, the default 'scs.auth.bp' will be
used. The `auth.options` object is passed directly to the blueprint as the
BluePrintSetupState.options attribute.

{: .note }
The below paragraphs refer to the options of the built-in auth module. If
you've set a different 'scs.auth.bp', these do not apply

`auth.options.users_file` defines the location of [scs-users.yaml](./defining-users).
The `directories.secrets` defines the directory to be referenced using the
`!scs-secret` YAML tag inside the scs-users.yaml file. If you want to use the
same folder when other secrets are kept, you can simply use a YAML anchor, as
illustrated in the [example](#server-configuration-file).

`auth.options.networks.private_only` is 'true' by default, and ensures data
on the server can only be accessed from internal subnets.
`auth.options.networks.whitelist` must be set, to indicate what IPs or subnets
are allowed to access the server. It's advised to define this as restrictive as
possible. Please note that regardless of these settings, it's a good idea to
use a firewall to controll access to the server. Although unauthorized IPs
cannot be used to access any data, their requests can still flood the server,
possibly causing denial of service.

The `auth.options.max_auth_fails_per_15_min` defines the maximum number of
requests allowed per IP address with false authentication credentials every 15
minutes (10 by default). In combination with the network whitelist, this should
reduce the chances of successfully brute-forcing the user authentication system.
Even if attackers can spoof all IP addresses, this will limit
the authentication attempts per 15 minutes to the value of this property
multiplied by the size of your whitelisted network(s).

## 5 Extensions Configuration
You can extend the core-functionality of SCS at runtime, by defining extensions.
Extensions are third-party Python packages that add more functionality to SCS.
For a description of the different types of extensions that are supported,
including considerations for development of these, please see the
[extensions documentation](/extensions).
