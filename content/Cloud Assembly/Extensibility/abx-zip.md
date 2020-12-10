
---
title: "Create a ZIP Bundle to include libraries for ABX"
date: 2020/12/10 10:36
anchor: "customvmname"
---

You can create a ZIP package that contains the Python script and dependencies used by your vRealize Automation Cloud Assembly extensibility actions.

There are two methods of building the script for your extensibility actions:

Writing your script directly in the ABX editor in vRealize Automation or creating your script on your local environment, then bundling it with any relevant dependencies in a ZIP package for ABX ingestion. By using a ZIP package, you can create a custom preconfigured template of action scripts and dependencies that you can import to vRealize Automation for use in ABX extensibility actions.

Furthermore, you can use a ZIP package in scenarios where modules associated with dependencies in your action script cannot be resolved by the vRealize Automation Cloud ABX, e.g. whenever your environment lacks Internet access.

This example uses Python but the same approach applies to other Scripting Engines.

+ Create and activate a new Python environment:

Let's assume you had Python enviroment: 

```js
---
root@ubuntu_server: mkdir environments
root@ubuntu_server: python3 -m venv vraDNSDev 
```

Create and move to the project root folder:

```js
---
(vraDNSDev) root@ubuntu_server: mkdir vraDNS-action    
(vraDNSDev) root@ubuntu_server: cd /root/enviroments/vraDNSDev/vraDNS-action
```
+ Define your library requirements and install them with PIP at your action root folder:

Copy or Create then place the requirements.txt inside your ABX action root folder For our example only the _"dnspython==1.16.0"_  proprietary propietary library is needed, which it is not included in the standard Python or FaaS Engines

```js
---
(vraDNSDev) root@ubuntu_server: cat requirements.txt 
	dnspython==1.16.0 
```

Now install _"dnspython==1.16.0"_ in your Python virtual enviroment:

```js
---
(vraDNSDev) root@ubuntu_server:  pip install -r requirements.txt --target=/root/enviroments/vraDNSDev/vraDNS-action/lib   
```

I create _"main.py"_  Python Script, it is a basic sample for translating a MSISDN number into ENUM format calling dnspython's dns.e164.from_e164() it also resolves and lists the MX records for a given domain via dnspython's dns.resolver.query() and finally resolve A record with prebuilt python's socket library.

```python
---
import socket
import dns.resolver
import dns.e164

def handler(context, inputs):
    print('Action started.')
    msAddr = inputs["msisdn"]
    dnsMX = inputs["dnsMX"]

    # Converts E164 Address to ENUM with propietary library dnspython
    n = dns.e164.from_e164(msAddr)
    print ('My MSISDN:', dns.e164.to_e164(n), 'ENUM NAME Conversion:', n)

    # Resolve DNS MX Query with propietary library dnspython
    answers = dns.resolver.query(dnsMX, 'MX')
    print ('Resolving MX Records for:', dnsMX)
    for rdata in answers:
	print ('Host', rdata.exchange, 'has preference', rdata.preference)

    # Resolve AAA with prebuilt socket library
    addr1 = socket.gethostbyname(dnsMX)
    print('Resolving AAA Record:', addr1)

    return addr1  
```

Now let's package the main Script with the customized installed libraries, both your script and dependency elements must be stored at the root level of the ZIP package.
When creating the ZIP package in a Linux environment (or libraries installed with arch Linux parameter), you might encounter a problem where the package content is not stored at the root level. If you encounter this problem, create the package by running the zip -r command in your command-line shell.

```js
---
(vraDNSDev) root@ubuntu_server: cd ~/enviroments/vraDNSDev/vraDNS-action
(vraDNSDev) root@ubuntu_server: zip -r9 vraDNS-actionR08.zip *
```

At this point, the ZIP package could be used for creating an ABX extensibility action script by importing it at vRealize Automation and following the menu to : [ Cloud Assembly ]--> [ Extensibility ] --> [ Actions ] --> [ Create a New Action ] then switch to _IMPORT PACKAGE_

Please note that same rules applies for defining the Handler Function, for example, my main script file is titled _"main.py"_ and my entry point is _"handler (context, inputs)"_, then name of the main function must be _"main.handler"_.


