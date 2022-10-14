---
layout: default
title: "Example"
description: "An illustration of SCS features"
permalink: "/example"
nav_order: 2
---
# SCS Basic Example
This page gives a quick demonstration of how the Simple Configuration Server
can be used to host configuration variables and configuration file templates
from a set of files.

## File Structure
The URL structure for the '/configs/' endpoints of a SCS deployment is
derived from the directory containing configuration variables and configation
file(s) (templates).

As an example, you can create the file-structure below:
```
.
├── scs-configuration.yaml (Main SCS configuration file)
├── scs-users.yaml (Contains the list of users of the SCS)
├── common
|   | # Contains global variables referenced using the !scs-common YAML tag
│   └── global.yaml
├── secrets
|   | # Resources in the secrets/ directory can be referenced using the
|   | # !scs-secret yaml tag
|   ├── database-users.yaml (Database user data)
|   └── scs-tokens.yaml (Access tokens for the SCS users)
└── config
    | # All files or templates under the config/ folder, except the ones ending
    | # in scs-env.yaml, are hosted under the configs/ url path of the server
    ├── database
    │   ├── create_users.json (Uses templating to build a JSON list from variables)
    │   └── create_users.json.scs-env.yaml (Variables and settings for the above)
    └── server1
        ├── db-config.json (Uses templating to build a JSON object)
        ├── db-config.json.scs-env.yaml (Variables and settings above endpoint)
        ├── db-config.json.template (Illustrates how to host the raw template)
        ├── db-config.json.template.scs-env.yaml (Disables templating for above
        |                                         endpoint)
        ├── environment  (Example of simple plain-text variable)
        ├── ip-address (Example of using calculations in a template)
        └── scs-env.yaml (Variables that apply to  all server1/* endpoints) 
```
The scs-configuration.yaml file references the locations of all relevant folders
(common, config, secrets) as well as the location of the scs-users.yaml file,
relative to the folder that's configured for the *SCS_CONFIG_DIR* environment
variable (See scs-configuration.yaml contents below). In the above example,
the root folder of the file structure should be set as the *SCS_CONFIG_DIR*.

The 'common/' and 'secrets/' folders contain Yaml files with common configuration
variables, and secrets.

The 'config/' folders contains all endpoints of the server. All files
ending with 'scs-env.yaml' contain configuration variables and settings related
to one endpoint, or a folder with endpoints. From inside the 'scs-env.yaml' file,
you can reference the contents of the 'common/' and 'secrets/' folders using
YAML tags. This way, you can put your configurations in Git, without including
any secrets. All files under the 'config/' folder, that do not end with
'scs-env.yaml', are hosted as endpoints of the server.

### File Contents
For reference, the contents of each file in the above structure, are provided
below. The files used in this example illustrate how the templating system
can be used for configuration files, and how 'secrets' and 'common'
configuration variables can be referenced.

<details markdown="1"><summary>scs-configuration.yaml</summary>
```yaml
directories:
  common: !scs-expand-env ${SCS_CONFIG_DIR}/common
  config: !scs-expand-env ${SCS_CONFIG_DIR}/config
  secrets: &secrets-dir !scs-expand-env ${SCS_CONFIG_DIR}/secrets
logs:
  audit:
    stdout:
      level: INFO
  application:
    stdout:
      level: INFO
auth:
  options:
    users_file: !scs-expand-env ${SCS_CONFIG_DIR}/scs-users.yaml
    directories:
      secrets: *secrets-dir
    networks:
      whitelist:
      - 127.0.0.1/32
```
</details>

<details markdown="1"><summary>scs-users.yaml</summary>
```yaml
- id: example-user
  token: !scs-secret 'scs-tokens.yaml#example-user'
  has_access:
    to_paths:
      - /configs/*
    from_networks:
      - 127.0.0.1/32
```
</details>

<details markdown="1"><summary>common/global.yaml</summary>
```yaml
database:
  host: 172.16.48.55
  port: 1234
```
</details>

<details markdown="1"><summary>secrets/database-users.yaml</summary>
```yaml
- username: server-1
  password: password1
- username: server-2
  password: password2
```
</details>

<details markdown="1"><summary>secrets/scs-tokens.yaml</summary>
```yaml
example-user: example-user-token
```
</details>

<details markdown="1"><summary>config/database/create_users.json</summary>
{% raw %}
```liquid
[
{% for user in users %}
  {"username": "{{ user.username }}", "password": "{{ user.password }}"}{% if not loop.last %},{% endif %}

{% endfor %}
]
```
{% endraw %}
</details>

<details markdown="1"><summary>config/database/create_users.scs-env.yaml</summary>
```yaml
template:
  context:
    users: !scs-secret 'database-users.yaml'
response:
  headers:
    Content-Type: application/json
```
</details>

<details markdown="1"><summary>config/server1/db-config.json</summary>
{% raw %}
```liquid
{
  "ip": "{{ database.host }}",
  "port": {{ database.port }},
  "username": {{ username }},
  "password": {{ password }},
}
```
{% endraw %}
</details>

<details markdown="1"><summary>config/server1/db-config.json.scs-yaml</summary>
```yaml
template:
  context:
    database: !scs-common 'global.yaml#database'
    username: !scs-secret 'database-users.yaml#[0].username'
    password: !scs-secret 'database-users.yaml#[1].password'
  response:
    headers:
      Content-Type: application/json
```
</details>

<details markdown="1"><summary>config/server1/db-config.json.template</summary>
(Same as config/server1/db-config.json)
</details>

<details markdown="1"><summary>config/server1/db-config.json.template.scs-env.yaml</summary>
```yaml
template:
  enabled: false
```
</details>

<details markdown="1"><summary>config/server1/environment</summary>
```
production
```
</details>

<details markdown="1"><summary>config/server1/ip-address</summary>
{% raw %}
```liquid
172.16.48.{{ 10 + server_number }}
```
{% endraw %}
</details>

<details markdown="1"><summary>config/server1/scs-env.yaml</summary>
```yaml
template:
  context:
    server_number: 1
response:
  headers:
    Content-Type: text/plain
```
</details>

## Usage
After deploying the SCS with this file-structure (See [Deployment Docs](./docs/deployment)),
you can query every endpoint the token of the example user (See
'scs-users.yaml' and 'secrets/scs-tokens.yaml' above). For example,
using curl:
```bash
curl http://localhost:5000/configs/database/create_users.json --header "Authorization: Bearer example-user-token"
# Output:
# [
#     {"username": "server-1", "password": "password1"},
#     {"username": "server-2", "password": "password2"}
# ]
curl http://localhost:5000/configs/server1/environment --header "Authorization: Bearer example-user-token"
# Output:
# production
curl http://localhost:5000/configs/server1/ip-address --header "Authorization: Bearer example-user-token"
# Output:
# 172.16.48.11
curl http://localhost:5000/configs/server1/db-config.json --header "Authorization: Bearer example-user-token"
# Output: 
# {
# "ip": "172.16.48.55",
# "port": 1234,
# "username": server-1,
# "password": password2,
# }
curl http://localhost:5000/configs/server1/db-config.json.template --header "Authorization: Bearer example-user-token"
# Output: 
# {
# "ip": "{{ database.host }}",
# "port": {{ database.port }},
# "username": {{ username }},
# "password": {{ password }},
# }
```

Note that this is a simple example that illustrates the main features of SCS.
For a more in-depth example of how you can use Git to version control your
configurations, and use the CI/CD or Workflow system of common Git platforms
to build a Docker image for your configuration, please take a look at the
[example-scs-configuration repository](https://github.com/simple-configuration-server/example-configuration).
