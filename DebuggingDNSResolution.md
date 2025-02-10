<h2> Scenario Setup</h2> 

Namespace: k8-dns
Components:
A Deployment with 2 Pods running a simple web server.
A Service to expose the Pods.
A Headless Service for direct Pod DNS resolution.
Objective: Validate DNS resolutions for both forward and reverse lookups.
Step 1: Create the Namespace

yaml
