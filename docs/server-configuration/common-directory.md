---
layout: default
title: "Common Directory"
description: "Use the common directory to store common configuration variables"
permalink: "/docs/server-configuration/common-directory"
nav_order: 3
grand_parent: Documentation
parent: Server Configuration
---
# Common Directory

{: .highlight }
Use the 'directories.common' variable inside scs-configuration.yaml to set
the location of this directory

Use the 'common' directory to store yaml files with configuration variables
that should be used by multiple endpoints. In case you're using
git for version control, you can also checkout [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
in subfolders of this directory, which gives you the option to include YAML
configuration files from other git repositories.

You can reference common variables from scs-env.yaml files, using the
'!scs-common' YAML tag. For example, given the following common directory
layout:
```
.
└── global.yaml
    > global_object:
    >   key: value
    > global_array:
    >   - value1
    >   - value2
```
These can be referenced from scs-env files, like:
```yaml
template:
  context:
    object: !scs-common 'global.yaml#global_object'
    key: !scs-common 'global.yaml#global_object.key'
    array: !scs-common 'global.yaml#global_array'
    array_item: !scs-common 'global.yaml#global_array.[0]'
```

{: .warning }
Since dots are used as a level seperator in relative references,
it's advised not to use keys containing dots in these files. By default, the
SCS will raise an error on startup if this is the case. If you need keys with
dots, disable this check by setting 'environments.reject_keys_containing_dots'
to false in scs-configuration.yaml.
