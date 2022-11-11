---
layout: default
title: Exceptions
description: "During server startup you may encounter these exceptions"
permalink: "/docs/troubleshooting/exceptions"
nav_order: 1
grand_parent: Documentation
parent: Troubleshooting
---
# Exceptions
If exceptions occur during server start-up, for example when loading the
configuration files or endpoint configurations, the server will fail to start.
This page describes the most common exceptions that can occur on server
startup, and how to fix them.

{: .note}
This list describes possible exceptions in the SCS core functionality. If you
have configured [extensions](../../extensions) these may raise additional
exceptions to the ones described below. Note that exceptions that are
raised during request handling, will trigger a 500 response code as described
[here](./error-responses), and the exception will be logged in the application
log.

| Class                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ValueError             | These occur in case values are wrongly configured, such as Environment Variables. Please see the error message for a more thorough description.                                                                                                                                                                                                                                                                                                    |
| YAMLError              | One of the YAML configuration files (e.g. scs-configuration.yaml, *scs-env.yaml files or scs-users.yaml) contain syntax errors. Check the error traceback to determine what file failed to load.                                                                                                                                                                                                                                                   |
| TemplateError          | One of the endpoint templates failed to render during caching. This can be because of problems with the template syntax or because the required context variables for the template are not available. To troubleshoot, disable [templates.cache](../server-configuration/config-file#2-templates-configuration) to temporarily suppress the issue and [validate your configuration](../configuration-validation) to test each endpoint seperately. |
| EnvFileFormatException | One or more *scs-env.yaml files do not conform to the [scs-env.yaml schema](https://github.com/simple-configuration-server/simple-configuration-server/blob/main/scs/schemas/scs-env.yaml).                                                                                                                                                                                                                                                        |

Note that you should be able to prevent most, if not all exceptions, by
[validating your configuration](../configuration-validation) prior to deployment.