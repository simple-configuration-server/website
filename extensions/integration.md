---
layout: default
title: "Integration"
description: "When developing an extension, make use of the SCS Error Handling and logging facilities"
permalink: "/extensions/integration.html"
nav_order: 1
parent: Extensions
---
# Integration
From you extension, you can use various core-features of the SCS, to make sure
your extension fully integrates with existing SCS deployments. Use the
core [scs.errors](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/scs/errors.py)
module to ensure any errors can be understood by the users and use the
[scs.logging](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/scs/logging.py)
module to ensure all relevant events are logged inside the SCS logs.

## 1 Custom Error Responses
Use the [scs.errors](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/scs/errors.py)
module inside your extension to add
custom error responses to SCS. There are 2 types of errors that can be
registered with the scs.errors module:
1. Errors based on response codes and error IDs, triggered using abort()
2. Errors based on Exceptions raised in the code

You can use the first type to explicitly return errors for certain conditions,
by:
1. Registering the status code, error id and error message combination
   with the scs.errors module:
    
   ```python
   from scs import errors

   errors.register(
       code=429,
       id_='auth-rate-limited',
       message='Rate limited due to too many false auth attempts from this ip',
   )
   ```

2. Using the flask.abort function somewhere in your code:
   
   ```python
   from flask import abort

   abort(429, description={'id': 'auth-rate-limited'})
   ```

The second option is to capture exceptions that are raised inside the code, and
return a 500 error message with an explaination:
1. Create a custom error class (optional) and register it (or alternatively,
   register existing exceptions):
    
   ```python
   from scs import errors

   class CustomException(Exception):
       pass

   errors.register_exception(
       exception_class=CustomException,
       id_='custom-exception-occurred',
       message='A custom exception occured'
   )
   ```

2. Raise the exception:

   ```python
   raise CustomException('Exception occured')
   ```

This creates a HTTP response with a 500 status code, showing the id and
message you provided.

{: .note}
It's good practice to include your package name inside the error id,
since otherwise these may overlap with errors registered by other modules.

For further examples on how to use the above error handling, also take a look
at the [SCS source code](https://github.com/Tom-Brouwer/simple-configuration-server/tree/master/scs),
since errors and exceptions are registered by multiple built-in SCS modules.

## 2 Logging
Use the [scs.logging](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/scs/logging.py) inside your extension to:
1. Log custom audit events
2. Log to the SCS application log

To log custom audit events:
1. Add a custom audit event:

   ```python
   from scs import logging

   logging.register_audit_event(
       type_="custom-audit-event",
       level='INFO',
       message_template="User '{user}' has done {terrible_thing}"
   )
   ```

2. Use the 'add_audit_event' function added to the Flask.g object:

   ```python
   from Flask import g

   g.add_audit_event(
       event_type='custom-audit-event',
       # Any keyword arguments other the event_type can be used inside the
       # message template
       terrible_thing="something aweful",
   )
   ```

{: .note}
As with error ids, it's advised to include your package name into
the 'event_type' to prevent overlap with other extensions.

To log to the application log, simply use the python logging module:

```python
import logging

logger = logging.getLogger(__name__)

logger.info('INFO message')
```
