---
layout: default
title: Error Responses
description: "The Simple Configuration Server uses error identifiers to indicate issues during request processing"
permalink: "/docs/troubleshooting/error-responses"
nav_order: 2
grand_parent: Documentation
parent: Troubleshooting
---
# Error Responses
When issues or errors occur during request processing, the SCS is designed to
return a JSON response and a status code >= 400. The JSON-data that's
returned includes a unique identifier of the error, and a message describing
what went wrong. The below table describes the response status codes that can
be returned by SCS, and how to fix them.

{: .note}
The set of error codes in the below table can be extended by [extensions](../../extensions)
at run-time. The below table only contains the error codes that are included
in the SCS core.

| Status Code | Id                       | Description                                                                                                                                                                                                                                                         |
| :---------- | :----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 400         | request-body-invalid     | You POST request body is invalid either (1) because the JSON is malformed or (2) the JSON payload does not match the schema configured for the endpoint                                                                                                             |
| 401         | unauthenticated          | You didn't provide valid authentication credentials                                                                                                                                                                                                                 |
| 403         | unauthorized-ip          | Either (1) the client IP was not globally whitelisted or (2) the user is not authorized to be used from this IP. In case there's no trace of this access attempt in the audit log, number (1) is the case                                                           |
| 403         | unauthorized-path        | You're trying to access a server path that's not authorized for your user                                                                                                                                                                                           |
| 404         | not-found                | The path you entered does not exist on the server                                                                                                                                                                                                                   |
| 405         | method-not-allowed       | The HTTP request method you're using is not allowed for the endpoint. By default, GET and POST requests are allowed. This setting can be controlled from the *scs-env.yaml files                                                                                    |
| 500         | env-syntax-error         | One of the *scs-env.yaml files relevant for the endpoint has a malformed YAML syntax                                                                                                                                                                                |
| 500         | env-format-error         | The format of one of the *scs-env.yaml files relevant for the endpoint does not conform to the [scs-env.yaml schema](https://github.com/simple-configuration-server/simple-configuration-server/blob/main/scs/schemas/scs-env.yaml)                                 |
| 500         | template-rendering-error | An error occured while rendering the template for the endpoint. Check the syntax in the template, and if all context variables are available and of the correct type. Check the application logs for more info.                                                     |
| 500         | internal-server-error    | An unexpected error occured. Check the application logs to see what has happened. If the error occured in a core SCS component, please create a [new issue](https://github.com/simple-configuration-server/simple-configuration-server/issues/new/choose) on GitHub |

Note that you should be able to prevent most errors by
[validating your configuration](../configuration-validation) prior to deployment.