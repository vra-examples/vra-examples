---
title: "Custom Integrations"
date: 2020/11/04 14:21
anchor: "customintegrations"
weight: 70
---
To create a Custom Integration:

1. Create a new Custom Integration
2. Select the Runtime to match the example below
3. Replace the placeholder code with the example from below
4. Save and version the Custom Integration, ensuring you eanble the “Release Version” toggle

To use a Custom Integration in a pipeline:

1. Configure the Host (you’ll need a Docker endpoint configured for this) and Builder image URL settings in the Workspace tab of your pipeline (e.g. python:latest will work for a Python3 example)
2. Add a new Custom task to your Stage, select the name of your integration and configure any inputs
3. Use the task outputs to refer to the output values in your later tasks - e.g. `stageName.taskName.output.properties.<outputPropertyName>`