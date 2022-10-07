---
layout: default
title: "Secrets Directory"
description: "Use the secrets directory to store your secrets seperatly"
permalink: "/docs/server-configuration/secrets-directory"
nav_order: 4
grand_parent: Documentation
parent: Server Configuration
---
# Secrets Directory

{: .highlight }
Use the 'directories.secrets' variable inside scs-configuration.yaml to set
the location of this directory

This directory works just like the ['common' directory](./common-directory). By
defining secrets in a seperate directory, you can completely seperate your
secrets from the configuration files itself, meaning you can safely store
the rest of the configuration (everything, except the Secrets Directory
and optionally also the User Definitions) in a git repository.

In secrets files, you can additionally use the `!scs-gen-secret` YAML tag to
let SCS auto-generate a secret using the [secrets.token_urlsafe](https://docs.python.org/3/library/secrets.html#secrets.token_urlsafe)
function, with a random number of bytes between 32 and 64 characters. The
first time a file with this tag is read, the token will be generated, and
saved back to the YAML file.

{: .note}
Files with the !scs-gen-secret YAML tag are parsed and re-saved. Any comments
or additional whitespace will be lost

Every time a user requests an endpoint that exposes 1 or more secrets, the
user and the requested secrets are logged in the audit log.

Use the `!scs-secret` YAML tag to reference secrets files and variables in
the secrets directory.
