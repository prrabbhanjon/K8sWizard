<h2> Scenario Setup</h2> 

Namespace: k8-dns
Components:
A Deployment with 2 Pods running a simple web server.
A Service to expose the Pods.
A Headless Service for direct Pod DNS resolution.
Objective: Validate DNS resolutions for both forward and reverse lookups.
Step 1: Create the Namespace

yaml
yaml
Copy
apiVersion: v1
kind: Namespace
metadata:
  name: k8-dns
Apply the manifest:

bash
Copy
kubectl apply -f namespace.yaml
Step 2: Create a Deployment

Create a Deployment with 2 Pods running an nginx web server:

yaml
Copy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: k8-dns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
Apply the manifest:

bash
kubectl apply -f deployment.yaml
Step 3: Create a Service

Create a Service to expose the Pods:

yaml
Copy
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: k8-dns
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
Apply the manifest:

bash
Copy
kubectl apply -f service.yaml
Step 4: Create a Headless Service

Create a Headless Service for direct Pod DNS resolution:

yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
  namespace: k8-dns
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
Apply the manifest:

bash
Copy
kubectl apply -f headless-service.yaml
Step 5: Validate DNS Resolutions

1. Forward Lookup for Service

Resolve the Service name (web-service) to its ClusterIP:

bash
Copy
kubectl run -it --rm --image=busybox:1.28 --restart=Never --namespace=k8-dns dns-test -- nslookup web-service.k8-dns.svc.cluster.local
Expected Output:

erver:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-service.k8-dns.svc.cluster.local
Address 1: 10.96.123.45 web-service.k8-dns.svc.cluster.local
2. Forward Lookup for Headless Service

Resolve the Headless Service name (web-headless) to individual Pod IPs:

bash
Copy
kubectl run -it --rm --image=busybox:1.28 --restart=Never --namespace=k8-dns dns-test -- nslookup web-headless.k8-dns.svc.cluster.local
Expected Output:

Copy
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-headless.k8-dns.svc.cluster.local
Address 1: 10.244.1.2 web-app-12345-abcde.k8-dns.pod.cluster.local
Address 2: 10.244.1.3 web-app-67890-fghij.k8-dns.pod.cluster.local
3. Reverse Lookup for Pod IPs

Resolve a Pod IP to its DNS name:

bash
Copy
kubectl run -it --rm --image=busybox:1.28 --restart=Never --namespace=k8-dns dns-test -- nslookup <pod-ip>
Replace <pod-ip> with one of the Pod IPs from the Headless Service output (e.g., 10.244.1.2).

Expected Output:

Copy
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      2.1.244.10.in-addr.arpa
Address 1: 10.244.1.2 web-app-12345-abcde.k8-dns.pod.cluster.local
Step 6: Simulate Pod Deletion and Validate DNS

Delete one of the Pods:
bash
Copy
kubectl delete pod <pod-name> -n k8-dns
The Deployment will automatically recreate the Pod with a new IP.
Validate DNS resolution again:
Check the Headless Service resolution to ensure the new Pod IP is included.
Perform a reverse lookup for the new Pod IP.
Key Takeaways

Service DNS: Provides a stable endpoint for accessing Pods, even when Pod IPs change.
Headless Service DNS: Allows direct resolution of individual Pod IPs.
Reverse DNS: Maps Pod IPs back to their DNS names.
CoreDNS: Automatically updates DNS records when Pods are recreated.
