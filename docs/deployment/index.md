---
layout: default
title: "Deployment"
description: "You can deploy SCS using as a Docker container"
permalink: "/docs/deployment"
nav_order: 3
parent: Documentation
has_children: true
---
# Deployment
You can use the SCS the in the following ways: 
1. By deploying Docker containers using the official SCS Docker Image
2. By cloning this repository and running SCS locally

Because of it's simplicitly and ease of updating, deploying SCS using a Docker
image is the preferred method. Local installation is not recommended for
anything other than running tests and debugging.

In addition to deploying SCS, you can also use logging agents to stream the
logs to a central logging database.

{: .note}
Prior to deploying to production, please consider the
[security recommendations in the security policy](https://github.com/simple-configuration-server/simple-configuration-server/security/policy#3-securing-scs-deployments)
first
