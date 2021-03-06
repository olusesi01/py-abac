# py-ABAC
Attribute Based Access Control (ABAC) for python. 

[![Build Status](https://travis-ci.com/ketgo/py-abac.svg?token=oCVxhfjJAa2zDdszGjoy&branch=master)](https://travis-ci.com/ketgo/py-abac)
[![codecov.io](https://codecov.io/gh/ketgo/pyabac/coverage.svg?branch=master)](https://codecov.io/gh/ketgo/pyabac/coverage.svg?branch=master)
[![Apache 2.0 licensed](https://img.shields.io/badge/License-Apache%202.0-yellow.svg)](https://raw.githubusercontent.com/kolotaev/vakt/master/LICENSE)

---

## Introduction

Py-ABAC is an attribute-based access control ([ABAC](https://en.wikipedia.org/wiki/Attribute-based_access_control)) toolkit based on policies. ABAC gives you a fine-grained control on definition of the rules that restrict an access to resources and is generally considered a "next generation" authorization model. The design of py-ABAC stems from the [XACML](https://en.wikipedia.org/wiki/XACML) standard, and the ABAC python SDK [Vakt](https://github.com/kolotaev/vakt).

See [documentation](https://py-abac.readthedocs.io) for more details.

## Install

PyABAC runs on Python >= 3.5. PyPy implementation is supported as well.


To install basic package run the following:
```bash
pip install py-abac
```

For different policy storage backends run:
```bash
# MongoDB backend
pip install py-abac[mongo]

# SQL backend
pip install py-abac[sql]
```

## Example Usage

A quick dive-in:

```python
from pymongo import MongoClient
from py_abac import PDP, Policy, AccessRequest
from py_abac.storage import MongoStorage

# Policy definition in JSON
policy_json = {
    "uid": "1",
    "description": "Max and Nina are allowed to create, delete, get any "
                   "resources only if the client IP matches.",
    "effect": "allow",
    "rules": {
        "subject": [{"$.name": {"condition": "Equals", "value": "Max"}},
                    {"$.name": {"condition": "Equals", "value": "Nina"}}],
        "resource": {"$.name": {"condition": "RegexMatch", "value": ".*"}},
        "action": [{"$.method": {"condition": "Equals", "value": "create"}},
                   {"$.method": {"condition": "Equals", "value": "delete"}},
                   {"$.method": {"condition": "Equals", "value": "get"}}],
        "context": {"$.ip": {"condition": "CIDR", "value": "127.0.0.1/32"}}
    },
    "targets": {},
    "priority": 0
}
# Parse JSON and create policy object
policy = Policy.from_json(policy_json)

# Setup policy storage
client = MongoClient()
storage = MongoStorage(client)
# Add policy to storage
storage.add(policy)

# Create policy decision point
pdp = PDP(storage)

# A sample access request JSON
request_json = {
    "subject": {
        "id": "", 
        "attributes": {"name": "Max"}
    },
    "resource": {
        "id": "", 
        "attributes": {"name": "myrn:example.com:resource:123"}
    },
    "action": {
        "id": "", 
        "attributes": {"method": "get"}
    },
    "context": {
        "ip": "127.0.0.1"
    }
}
# Parse JSON and create access request object
request = AccessRequest.from_json(request_json)

# Check if access request is allowed. Evaluates to True since 
# Max is allowed to get any resource when client IP matches.
assert pdp.is_allowed(request)
```

## Documentation

Py-ABAC documentation can be found at [https://py-abac.readthedocs.io](https://py-abac.readthedocs.io)

You can also build the documentation by running `make html` inside the `docs` folder.

## Logging

py-ABAC follows a common logging pattern for libraries:

Its corresponding modules log all the events that happen but the log messages by default are handled by `NullHandler`. It's up to the outer code/application to provide desired log handlers, filters, levels, etc.

For example:

```python
import logging

root = logging.getLogger()
root.setLevel(logging.INFO)
root.addHandler(logging.StreamHandler())

... # here go all the py_abac calls.
```

## Milestones

Most valuable features to be implemented in the order of importance:

- [x] Sphinx Documentation
- [ ] Policy Obligations
- [ ] In-Memory Storage
- [x] SQL Storage
- [ ] Caching mechanism for Storage
- [ ] File Storage

## Acknowledgements

The conceptual and implementation design of py-ABAC stems from the [XACML](https://en.wikipedia.org/wiki/XACML) standard and the ABAC python SDK [Vakt](https://github.com/kolotaev/vakt).

*[Back to top](#py-abac)*

## Development

To hack py-ABAC locally run:

```bash
$ pip install -e .[dev]			# to install all dependencies
$ docker run --rm -d -p 27017:27017 mongo			# Run mongodb server on docker
$ pytest --cov=py_abac tests/			# to get coverage report
$ pylint py_abac			# to check code quality with PyLint
$ bandit py_abac			# to check code security with Bandit
```

Optionally you can use `make` to perform development tasks.

## License

The source code is licensed under Apache License Version 2.0

## Contributions

Pull requests and bug reports always welcomed! :)
