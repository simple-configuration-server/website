---
layout: default
title: "Integration"
description: "When developing an extension, make use of the SCS Error Handling and logging facilities"
permalink: "/extensions/integration"
nav_order: 1
parent: Extensions
---
# Integration
SCS extends the Flask app object with several functions for defining
standardized responses in case errors or exceptions occur and functions to
register audit events from an extension, such as a blueprint. Application logs
can be generated from any part of the code using the built-in python
logging package.

{: .note}
The functions for registering errors, exceptions and audit events are linked
to the Flask app object. As illustrated in the examples, these are mainly
designed to be used from inside Flask Blueprints. It's possible to use these in
other extension types also, but you'd have to use the flask.current_app object
that's available during a request and add code to register your errors,
exceptions or audit events when your extension is used the first time.

## 1 Custom Error Responses
Use the `app.scs.register_error()` function to define custom JSON error
responses that are returned in case a specific HTTP status code is generated.
An example implementation:
1. Register the status code, error id and error message combination
   when the blueprint is initialized:
    
   ```python
   from flask import BluePrint

   bp = Blueprint('test', 'test')

   @bp.record
   def init(setup_state):
      setup_state.app.scs.register_error(
          code=429,
          id_='auth-rate-limited',
          message='Rate limited due to too many false auth attempts from this ip',
      )
   ```

2. Use the flask.abort function somewhere in your code:
   
   ```python
   from flask import abort

   abort(429, description={'id': 'auth-rate-limited'})
   ```

Additionally, use the `app.scs.register_exception()` function to ensure
informational error messages are returned, in case specific exceptions are
raised during request processing. An example implementation:
1. Create a custom error class (optional) and register it (or alternatively,
   register existing exceptions):
    
   ```python
   from flask import BluePrint
   from scs import errors

   class CustomException(Exception):
       pass

   bp = Blueprint('test', 'test')

   @bp.record
   def init(setup_state):
      setup_state.app.scs.register_exception(
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
It's good practice to prefix the error-id with your package name, since
your ids may otherwise overlap with errors registered by other modules.

For further examples on how to use the above error handling, also take a look
at the [SCS source code](https://github.com/simple-configuration-server/simple-configuration-server/tree/main/scs),
since errors and exceptions are registered by multiple built-in SCS modules.

## 2 Logging
Use the `app.scs.register_audit_event()` function to register audit events at
initialization, after which you can log them for each request. An example:

1. Add a custom audit event:

   ```python
   from flask import BluePrint

   bp = Blueprint('test', 'test')

   @bp.record
   def init(setup_state):
      setup_state.app.scs.register_audit_event(
          type_="custom-audit-event",
          level='INFO',
          message_template="User '{user}' has done {terrible_thing}"
      )
   ```

2. Use the 'add_audit_event' function added to the Flask.g object while
   handling a request:

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
As with error ids, it's advised to prefix the 'event_type' with your package
name to prevent overlap with other extensions.

To log to the application log, simply use the python logging module:

```python
import logging

logger = logging.getLogger(__name__)

logger.info('INFO message')
```
