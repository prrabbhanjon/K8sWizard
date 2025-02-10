<p> In Kubernetes, DNS resolution is a critical component for service discovery. When a Pod is deleted and recreated automatically (e.g., by a Deployment or StatefulSet), its IP address may change. However, Kubernetes provides mechanisms to ensure that DNS resolution continues to work seamlessly in such scenarios. Here's how forward and reverse DNS lookups work in Kubernetes and how communication is maintained: </p>

 <!-- This is a 1st line -->
<h2> 1. Forward Lookup (Service to Pod)</h2>
<p> Forward lookup is the process of resolving a service name to its IP address. Kubernetes uses CoreDNS (or kube-dns) as the default DNS service to handle this.</p> 
<ul>
    <li> <h3>  Service DNS Resolution: </h3></li>
  <ul>
    <li>When a Service is created, Kubernetes automatically creates a DNS record for it in the format: <code style="color : name_color"> <service-name>.<namespace>.svc.cluster.local.</code></li>
    <li>Pods can resolve the Service name to its ClusterIP using this DNS record.</li>
    <li>Example: If you have a Service named my-service in the default namespace, Pods can resolve it using <code style="color : name_color"> my-service.default.svc.cluster.local. </code></li>
  </ul>
</ul>
<!-- This is a 2nd line -->      
    <ul>
      <li> <h3> Pod DNS Resolution: </h3> </li>
        <ul>
            <li>Pods are assigned a DNS name in the format: <pod-ip>.<namespace>.pod.cluster.local.</li>
             <li>However, this is rarely used directly because Pod IPs are ephemeral and change when Pods are recreated.</li> 
        </ul>
    </ul>
<!-- This is a 3rd line -->  
<ul>
  <li> <h3> How Forward Lookup Works When Pods Are Recreated:</h3> </li>
  <ul>
      <li>When a Pod is deleted and recreated, its IP changes, but the Service's ClusterIP remains the same.</li>
      <li>The Service acts as a stable endpoint, and it routes traffic to the new Pod IPs using its underlying Endpoints or EndpointSlice objects.</li>
      <li>CoreDNS automatically updates its records to reflect the new Pod IPs associated with the Service.  </li> 
  </ul>
</ul>
             
<!-- This is a 4th line -->
<h2>2. Reverse Lookup (IP to Name)</h2>
<p>Reverse lookup is the process of resolving an IP address to a hostname. In Kubernetes, reverse DNS lookup is less commonly used but can be achieved.</p>
<ul>
    <li> <h3> Pod Reverse DNS:</h3></li>
  <ul>
    <li>Kubernetes assigns a reverse DNS name to each Pod in the format: <code style="color : name_color"> <pod-ip>.<namespace>.pod.cluster.local.</code></li>
    <li>Example: If a Pod has the <code style="color : name_color"> IP 10.244.1.2</code>, its reverse DNS name would be <code style="color : name_color"> 2.1.244.10.in-addr.arpa.</code></li> 
  </ul>
</ul>
     
<!-- This is a 3rd line -->  
<ul> 
    <li> <h3>Service Reverse DNS:</h3> </li>
  <ul>
    <li> Services do not typically have reverse DNS records because they use ClusterIPs, which are virtual IPs and not tied to a specific Pod.</li> 
  </ul>
</ul>

<!-- This is a 3rd line --> 
<ul>
   <h3>How Reverse Lookup Works When Pods Are Recreated:</h3> </li>
  <ul>
      <li> When a Pod is recreated, its IP changes, and the reverse DNS record is updated accordingly. </li>
      <li> CoreDNS handles these updates automatically. </li>
  </ul>
</ul>

<!-- This is a Heading line -->
<h2> 3. Communication in a Dynamic Environment</h2>
   <p> When Pods are deleted and recreated automatically (e.g., by a Deployment), the following ensures seamless communication: </p>
<!-- This is a sube heading line -->
<ul>
   <li> <h3> Service Abstraction:</h3> </li>
 <ul>
    <li>Use Services to abstract Pod IPs. Services provide a stable endpoint (ClusterIP) that does not change even if the underlying Pods are recreated.</li>
    <li>Applications should communicate with the Service name <code style="color : name_color"> (e.g., my-service.default.svc.cluster.local) </code>rather than directly with Pod IPs.</li>
  </ul>
</ul> 
<ul>
<li> <h3>CoreDNS Updates:</h3></li>
  <ul>
    <li>CoreDNS continuously watches the Kubernetes API for changes to Services, Endpoints, and Pods. </li>
    <li> When a Pod is recreated, CoreDNS updates its DNS records to reflect the new Pod IPs.</li>
  </ul>
</ul>
<ul>
   <li> <h3>Headless Services:</h3></li>
 <ul>
    <li> If you need direct Pod-to-Pod communication, you can use a Headless Service <code style="color : name_color">(by setting clusterIP: None).</code> This creates DNS records for individual Pods, allowing direct resolution of Pod IPs.</li>
    <li> Example: A Headless Service named my-headless-service will create DNS records like <code style="color : name_color"> pod-ip.my-headless-service.default.svc.cluster.local.</code></li>
  </ul>
</ul>
