---
layout: default
title: "(Auth) Blueprints"
description: "Develop blueprint extensions manipulate requests and responses"
permalink: "/extensions/blueprints"
nav_order: 3
parent: Extensions
---
# Blueprints
You can use any Flask Blueprint as an extension for SCS. This page provides
general guidance on developing blueprint extensions, and considerations for
developing custom authentication blueprints.

## 1 Extensions
When developing a Flask Blueprint extension, note that 'options' can be passed
via the 'scs-configuration.yaml' file. You can capture these, by creating
an initialization function in your blueprint, with the `bp.record` decorator.
Below is a simple example:

_mypackage/mymodule.py_
```python
from flask import Blueprint

bp = Blueprint('custom-blueprint')

@bp.record
def init(setup_state: BlueprintSetupState):
    global blueprint_name
    opts = setup_state.options
    blueprint_name = opts['name']

@bp.after_app_request
def print_something(response: Response) -> Response:
    print(f'{blueprint_name}: This is executed after the app request')
    return response
```

This is configured from scs-configuration.yaml like:
```yaml
extensions:
  blueprints:
    - name: mypackage.mymodule.bp
      options:
        name: 'Great Blueprint!'
```

## 2 Authentication Extension
The same considerations as above apply to developing an authentication
extension for SCS. You can use the source-code of the built-in
[scs.auth](https://github.com/Tom-Brouwer/simple-configuration-server/blob/master/scs/auth.py)
module for inspiration. Make sure to integrate your auth module with the
logging module, so important authentication events are logged to the audit log.
