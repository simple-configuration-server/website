---
layout: default
title: "Extensions"
description: "You can develop and/or use Python packages to extend SCS at runtime"
permalink: "/extensions"
nav_order: 4
has_children: true
---
# Extensions
The SCS is designed to be extended with additional functionality at runtime.
The following types of extensions are supported:

* **YAML Constructors**: YAML constructors allow you to add support for
  additional YAML tags to the SCS. Imagine for example you'd want to include
  secrets from a 3rd party secret store in your scs-env.yaml file, you can
  create a constructor that uses a tag like `!third-party-secret` to
  refer to these secrets. Users can pass initialization options to constructors
  from the configuration file, such as credentials for a third party service
  used by the contructor.
* **Flask Blueprints**: Flask Blueprints allow you to add functionality to the
  request/response handling of the system, for example to create custom
  request logs, or add CORS support to the system.
* **Jinja Extensions**: Jinja extensions allow you to add functions that can
  be used within configuration file templates.

This section provides guidance on how to develop extensions for the
SCS, and the features available to integrate them.
