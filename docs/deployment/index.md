---
layout: default
title: "Deployment"
description: "Describes considerations for deploying the SCD"
permalink: "/docs/deployment.html"
nav_order: 2
parent: Documentation
has_children: true
---
# Deployment
You can use the SCS the in the following ways: 
1. By deploying Docker containers using the official SCS Docker Image
2. By cloning this repository and running SCS locally

Because of it's simplicitly and ease of updating, deploying SCS using Docker
is the preferred method. Local installation is not recommended for anything
other than running tests and debugging.

In addition to deploying SCS, you can also use logging agents to stream the
logs to a central logging database.

{: .note}
Prior to deploying to production, please consider the
[security recommendations in SECURITY.md ](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/SECURITY.md#3-securing-scs-deployments)
first
