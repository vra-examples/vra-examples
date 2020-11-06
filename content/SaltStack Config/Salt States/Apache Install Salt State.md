---
title: "Apache Install Salt State"
date: 2020/11/04 14:21
anchor: "apacheinstallsaltstate"
---
This Salt State can be used to install Apache and ensure that the service is running:

```yaml
apache:
    pkg.installed: []
    service.running:
        - require:
        - pkg: apache
```