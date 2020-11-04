
---
title: "Custom VM Name"
date: 2020/11/04 14:21
anchor: "customvmname"
---

This action will rename your VM to the value specified inthe custom property `userDefinedName` - it need to be called at the Compute allocation stage as a blocking task.

Python:
```python
def handler(context, inputs):
    old_name = inputs["resourceNames"][0]
    new_name = inputs["customProperties"]["userDefinedName"]
    outputs = {}
    outputs["resourceNames"] = inputs["resourceNames"]
    outputs["resourceNames"][0] = new_name
    print("Setting machine name from {0} to {1}".format(old_name, new_name))
    return outputs
```
PowerShell:
```powershell
function handler($context, $payload) {
    $oldVMName = $payload.resourceNames[0]
    $newVMName = $payload.customProperties.userDefinedName
    $returnObj = [PSCustomObject]@{
        resourceNames       = $payload.resourceNames
    }
    $returnObj.resourceNames[0] = $newVMName
    Write-Host "Setting machine name from $($oldVMName) to $($newVMName)"
    return $returnObj
}
```