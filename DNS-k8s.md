In Kubernetes, DNS resolution is a critical component for service discovery. When a Pod is deleted and recreated automatically (e.g., by a Deployment or StatefulSet), its IP address may change. However, Kubernetes provides mechanisms to ensure that DNS resolution continues to work seamlessly in such scenarios. Here's how forward and reverse DNS lookups work in Kubernetes and how communication is maintained:

1. Forward Lookup (Service to Pod)

Forward lookup is the process of resolving a service name to its IP address. Kubernetes uses CoreDNS (or kube-dns) as the default DNS service to handle this.

Service DNS Resolution:
When a Service is created, Kubernetes automatically creates a DNS record for it in the format: <service-name>.<namespace>.svc.cluster.local.
Pods can resolve the Service name to its ClusterIP using this DNS record.
Example: If you have a Service named my-service in the default namespace, Pods can resolve it using my-service.default.svc.cluster.local.
Pod DNS Resolution:
Pods are assigned a DNS name in the format: <pod-ip>.<namespace>.pod.cluster.local.
However, this is rarely used directly because Pod IPs are ephemeral and change when Pods are recreated.
How Forward Lookup Works When Pods Are Recreated:
When a Pod is deleted and recreated, its IP changes, but the Service's ClusterIP remains the same.
The Service acts as a stable endpoint, and it routes traffic to the new Pod IPs using its underlying Endpoints or EndpointSlice objects.
CoreDNS automatically updates its records to reflect the new Pod IPs associated with the Service.
2. Reverse Lookup (IP to Name)

Reverse lookup is the process of resolving an IP address to a hostname. In Kubernetes, reverse DNS lookup is less commonly used but can be achieved.

Pod Reverse DNS:
Kubernetes assigns a reverse DNS name to each Pod in the format: <pod-ip>.<namespace>.pod.cluster.local.
Example: If a Pod has the IP 10.244.1.2, its reverse DNS name would be 2.1.244.10.in-addr.arpa.
Service Reverse DNS:
Services do not typically have reverse DNS records because they use ClusterIPs, which are virtual IPs and not tied to a specific Pod.
How Reverse Lookup Works When Pods Are Recreated:
When a Pod is recreated, its IP changes, and the reverse DNS record is updated accordingly.
CoreDNS handles these updates automatically.
3. Communication in a Dynamic Environment

When Pods are deleted and recreated automatically (e.g., by a Deployment), the following ensures seamless communication:

Service Abstraction:
Use Services to abstract Pod IPs. Services provide a stable endpoint (ClusterIP) that does not change even if the underlying Pods are recreated.
Applications should communicate with the Service name (e.g., my-service.default.svc.cluster.local) rather than directly with Pod IPs.
CoreDNS Updates:
CoreDNS continuously watches the Kubernetes API for changes to Services, Endpoints, and Pods.
When a Pod is recreated, CoreDNS updates its DNS records to reflect the new Pod IPs.
Headless Services:
If you need direct Pod-to-Pod communication, you can use a Headless Service (by setting clusterIP: None). This creates DNS records for individual Pods, allowing direct resolution of Pod IPs.
Example: A Headless Service named my-headless-service will create DNS records like pod-ip.my-headless-service.default.svc.cluster.local.
4. Example Scenario

Let’s say you have a Deployment with 3 Pods and a Service named my-service.

Initial State:
Pods have IPs: 10.244.1.2, 10.244.1.3, 10.244.1.4.
Service my-service resolves to these IPs.
Pod Deletion and Recreation:
One Pod (10.244.1.2) is deleted and recreated with a new IP: 10.244.1.5.
CoreDNS updates the DNS records for my-service to include the new IP (10.244.1.5) and removes the old IP (10.244.1.2).
Communication:
Applications continue to use my-service.default.svc.cluster.local to communicate.
The Service routes traffic to the new Pod IP (10.244.1.5) seamlessly.
