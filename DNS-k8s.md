<p> In Kubernetes, DNS resolution is a critical component for service discovery. When a Pod is deleted and recreated automatically (e.g., by a Deployment or StatefulSet), its IP address may change. However, Kubernetes provides mechanisms to ensure that DNS resolution continues to work seamlessly in such scenarios. Here's how forward and reverse DNS lookups work in Kubernetes and how communication is maintained: </p>
<h2> 1. Forward Lookup (Service to Pod)</h2>
<p> Forward lookup is the process of resolving a service name to its IP address. Kubernetes uses CoreDNS (or kube-dns) as the default DNS service to handle this.</p> 
<ui><li> <h3>  Service DNS Resolution: </h3></li>
  <ul>
<li>When a Service is created, Kubernetes automatically creates a DNS record for it in the format: <code style="color : name_color"> <service-name>.<namespace>.svc.cluster.local.</code></li>
<li>Pods can resolve the Service name to its ClusterIP using this DNS record.</li>
<li>Example: If you have a Service named my-service in the default namespace, Pods can resolve it using <code style="color : name_color"> my-service.default.svc.cluster.local. </code></li></ul></ui>

  <ui><li> <h3> Pod DNS Resolution: </h3></li>
  <ul>
<li>Pods are assigned a DNS name in the format: <pod-ip>.<namespace>.pod.cluster.local.</li>
<li>However, this is rarely used directly because Pod IPs are ephemeral and change when Pods are recreated.</li>
<li>How Forward Lookup Works When Pods Are Recreated:</li>
<li>When a Pod is deleted and recreated, its IP changes, but the Service's ClusterIP remains the same.</li>
<li>The Service acts as a stable endpoint, and it routes traffic to the new Pod IPs using its underlying Endpoints or EndpointSlice objects.</li>
<li>CoreDNS automatically updates its records to reflect the new Pod IPs associated with the Service. </li></ul></ui>
