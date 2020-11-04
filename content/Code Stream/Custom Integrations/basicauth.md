---
title: "Basic Authentication Header"
date: 2020/11/04 14:21
anchor: "basicauthenticationheader"
---

To generate a basic authentication header from a username and password in Code Stream you could use a CI task and execute `echo -n username:password | base64`, then export the result for use later on. A more repeatable way is to create a Custom Integration that takes the two inputs, and returns the encoded header as an output.

Using the shell runtime:
```yaml
---
runtime: shell
code: |
  export basicAuthHeader=$(echo -n $username:$password | base64)
inputProperties:
  # Username input
  - name: 'username'
    type: text
    title: 'Username'
    placeHolder: 'Enter basic authentication usename'
    required: true
    bindable: true
  # Password input
  - name: 'password'
    type: password
    title: 'Password'
    placeHolder: 'Enter basic authentication password'
    defaultValue: ''
    required: true
    bindable: true
outputProperties:
  - name: basicAuthHeader
    type: label
    title: Basic Authentication Header
```
Using the Python3 runtime:

```yaml
---
runtime: "python3"
code: |
  from base64 import b64encode
  from context import getInput, setOutput
  username = getInput('username')
  password = getInput('password')
  usernameAndPassword = b64encode(bytes(f'{username}:{password}',encoding='ascii')).decode('ascii')
  setOutput('basicAuthHeader', "Basic "+usernameAndPassword)
inputProperties:
  # Username input
  - name: 'username'
    type: text
    title: 'Username'
    placeHolder: 'Enter basic authentication usename'
    required: true
    bindable: true
  # Password input
  - name: 'password'
    type: password
    title: 'Password'
    placeHolder: 'Enter basic authentication password'
    defaultValue: ''
    required: true
    bindable: true
outputProperties:
  - name: basicAuthHeader
    type: label
    title: Basic Authentication Header
```