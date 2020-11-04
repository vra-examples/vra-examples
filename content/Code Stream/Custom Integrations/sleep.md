---
title: "Sleep"
date: 2018-01-28T22:01:36+01:00
anchor: "sleep"
---

Sleeps for a custom number of seconds set by `sleepTime`:

```yaml
---
runtime: shell
code: |
  sleep $sleepTime
inputProperties:
  - name: 'sleepTime'
    type: number
    title: 'Sleep Time (s)'
    placeHolder: 'Number input'
    minimum: 1
    maximum: 600
    defaultValue: 30
```