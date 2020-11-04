---
title: "Sleep"
date: 2020/11/04 14:21
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