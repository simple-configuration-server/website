---
layout: default
title: "Secrets Directory"
description: "Use the secrets directory to store your secrets seperatly"
permalink: "/docs/server-configuration/secrets-directory.html"
nav_order: 4
grand_parent: Documentation
parent: Server Configuration
---
# Secrets Directory

{: .highlight }
Use the 'directories.secrets' variable inside scs-configuration.yaml to set
the location of this directory

This directory works just like the 'common' directory (ADD REFERENCE). By
defining secrets in a seperate directory, you can completely seperate your
secrets from the configuration files itself, meaning you can safely store
the rest of the configuration (everything, except the Secrets Directory
and optionally also the User Definitions) in a git repository.

Every time a user requests an endpoint that exposes 1 or more secrets, the
user and the requested secrets are logged in the audit log.

Use the '!scs-secret' YAML tag to reference secrets files and variables in
the secrets directory.
