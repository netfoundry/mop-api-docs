---
title: Tools
permalink: /tools/
redirect_from:
    - /v2/tools/
toc: false
---

*Miscellaneous Resources for NetFoundry API*

## HelpAPI for BASH

[HelpAPI](https://github.com/netfoundry/mop-bash-helpapi) is an interactive terminal console.

## Python Module

[Python Package Index module](https://pypi.org/project/netfoundry/)
: Python3 interface to the NetFoundry API

```bash
pip install netfoundry
```

Looking for [the Python guide](/guides/python/)?

### Postman

* [PostMan Service Runner](https://github.com/netfoundry/developer-tools/raw/master/NetFoundryRunners.postman_collection.json)
: create Services in bulk with PostMan

## Scripts

* [bulkInviteEndpoints.py](https://raw.githubusercontent.com/netfoundry/developer-tools/master/bulkInviteEndpoints.py)
: create Endpoints and send the enrollment token to a list of email addresses

```bash
# Supply email addresses one per line on standard input or use --invitees INVITEES_FILE
python3 ./bulkInviteEndpoints.py \
            --metadata laptop \
            --network-name ZitiBastions \
            --attributes salesEndpoints \
            --credentials credentials.json  <<< "alice.faulkner@example.com"

python3 ./bulkInviteEndpoints.py \
            --metadata mobile \
            --network-name ZitiBastions \
            --attributes salesEndpoints \
            --credentials credentials.json  <<EOF
alice.faulkner@example.com
bob.criton@example.com
EOF
```

* [bulkEditRoleAttributes.py](https://raw.githubusercontent.com/netfoundry/developer-tools/master/bulkEditRoleAttributes.py)
: replace the role attributes on all Endpoints, Edge Routers, or Services; optionally limited to those that match a pattern

```bash
python3 ./bulkEditRoleAttributes.py endpoints \
            --credentials credentials.json \
            --attributes sandbox staging production \
            --network-name ZitiBastions \
            --include ".*laptop$"                                       
```

### Ansible Modules

Looking for [the Ansible Guide](/guides/ansible/)?

## Demos

Looking for [the Quickstart Demo](/guides/demo/)?

## More Control with Python

Looking for [the Python guide](/guides/python/)?
