---
layout: default
title: "YAML Constructors"
description: "Develop YAML constructors to be able to use custom YAML tags"
permalink: "/extensions/yaml-constructors"
nav_order: 2
parent: Extensions
---
# YAML Constructors
All YAML constructors must inherit from the SCSYamlTagConstructor class. SCS
adds the the following attributes to a loader that's passed to construct
method:
* `.resave` (bool): If during parsing you edit some part of the JSON, set this to
  `True` inside your constructor, to indicate the parsed data should be saved
  back to the file after loading.
* `.filepath` (pathlib.Path): Use this inside your constructor to for example
  parse relative paths.

Below is a simple example of a custom YAML constructor:

_mymodule.py_
```python
from scs.yaml import SCSYamlTagConstructor

class AddOwnPathConstructor(SCSYamlTagConstructor):
    """
    A YAML tag that is replaced by the path of the file itself

    Attributes:
      resave: Whether to save the generated value back to the file
    """
    # Note that self.tag can also be made dynamic, by setting it as input to
    # the init function
    tag = '!set-own-path'

    def __init__(self, *args, resave: bool, **kwargs):
        self.resave = resave
        super().__init__(*args, **kwargs)

    def construct(self, loader: Loader, node: Node) -> Any:
        # If resave is true, replace tag with value on first load.
        # Note that the 'resave' option removes any comments and additional
        # whitespace from the YAML file
        if self.resave:
          # Prevent setting it to false when self.resave=False, because another
          # constructor may have set it to True already...
          loader.resave = True
        return loader.filepath.as_posix()
```

To use this YAML constructor, add the following to your scs-configuration.yaml
file:
```yaml
extensions:
  constructors:
    - name: mymodule.AddOwnPathConstructor
      options:
        resave: false
```