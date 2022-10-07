---
layout: default
title: "Jinja"
description: "Develop Jinja extensions to extend the SCS templating system"
permalink: "/extensions/jinja"
nav_order: 4
parent: Extensions
---
# Jinja Extensions
Due to the design of Jinja extensions (See [docs](https://jinja.palletsprojects.com/en/3.0.x/extensions/#writing-extensions)), it's not possible to pass options to the
init method of an extension. However, by using the `templates.rendering_options`
property in scs-configuration.yaml, and/or `template.rendering_options` in
scs-env files, users can extend the environment attributes, which can then be
used inside the extension. A basic example:

_mymodule.py_
```python
from jinja2.ext import Extension
import functools

def add_suffix(str_: str, *, suffix: str) -> str:
    """
    Add a suffix the end of a string
    """
    return str_ + suffix


class AddSuffixExtension(Extension):
    """
    Simple Extension that adds SCS to the end of a phrase
    """
    def __init__(self, environment):
        super().__init__(environment)
        suffix = environment.suffix_for_string
        environment.filters['add_suffix'] = functools.partial(
          add_suffix, suffix=suffix
        )
```

This is configured from scs-configuration.yaml like:
```yaml
templates:
  rendering_options:
    suffix_for_string: ' is the best!'

extensions:
  jinja2:
    - name: mymodule.AddSuffixExtension
```
