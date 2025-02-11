<!-- This is a heading line -->    
<div class="container">
    <h1>Scenario Setup</h1>
    <p><strong>1. Namespace:</strong> <code style="color : name_color"> k8-dns </code></p>
    <p><strong>2. Components:</strong> </p>
    <ul>
        <ul>
        <li>A <strong>Deployment </strong>with 2 Pods running a simple web server. </li>
        <li>A <strong>Service </strong>to expose the Pods.</li>
        <li>A <strong>Headless Service</strong> for direct Pod DNS resolution. </li>
        </ul>
    </ul>
    <p><strong>Objective:</strong> Validate DNS resolutions for both forward and reverse lookups.</p
</div>
<!-- This is a heading line -->  
    <div class="step">
        <h3>Step 1: Create the Namespace</h3>
        <pre lang="java">
apiVersion: v1
kind: Namespace
metadata:
  name: k8-dns </pre>
        <p>Apply the manifest:</p>
        <pre lang="java" class="command">kubectl apply -f namespace.yaml</pre>
    </div>
<!-- This is a heading line -->  
<div class="step">
        <h3> Step 2: Create a Deployment </h3>
        <p>Create a Deployment with 2 Pods running an <code style="color : name_color"> nginx </code> web server: </p>
 </div>
 <div class="step>
       <pre lang="java">
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
        image: nginx</pre>
        <p>Apply the manifest:</p>
        <pre lang="java" class="command">kubectl apply -f deployment.yaml</pre>
    </div>
<!-- This is a heading line -->  
    <div class="step">
        <h3>Step 3: Create a Service</h3>
        <p>Create a Service to expose the Pods:</p>
        <pre lang="java">
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
    targetPort: 80</pre>
        <p>Apply the manifest:</p>
        <pre lang="java" class="command">kubectl apply -f service.yaml</pre>
    </div>
<!-- This is a heading line --> 
    <div class="step">
        <h3>Step 4: Create a Headless Service </h3>
        <p>Create a Headless Service for direct Pod DNS resolution:</p> 
        <pre lang="java">
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
    targetPort: 80</pre>
        <p>Apply the manifest:</p>
        <pre lang="java" class="command">kubectl apply -f headless-service.yaml</pre>
    </div>
<!-- This is a heading line -->  
<div>
<h3> Applying Kubernetes YAML Files in a Loop</h3>
<p> Run the following command to apply all .yaml files in the current directory sequentially, with a 5-second pause between each:</p>
    <pre># ls -ltr
total 32
-rw-r--r--@ 1 prrabbhanjon  staff  151 Feb  9 16:29 4-Service.yaml
-rw-r--r--@ 1 prrabbhanjon  staff  273 Feb  9 16:29 3-deployment.yaml
-rw-r--r--@ 1 prrabbhanjon  staff  170 Feb  9 16:29 2-headless-service.yaml
-rw-r--r--@ 1 prrabbhanjon  staff   56 Feb  9 16:29 1-namespace.yaml</pre>
<pre class="command">
#for file in *.yaml; do kubectl apply -f $file; sleep 5; done  
namespace/k8-dns created
service/web-service created
deployment.apps/web-app created
service/web-headless created</pre></div>

<!-- This is a heading line --> 

<div class="step">     
        <pre class="command">kubectl get all -n k8-dns 
NAME                           READY   STATUS    RESTARTS   AGE
pod/web-app-65d846d465-pt2sb   1/1     Running   0          53m
pod/web-app-65d846d465-xqclv   1/1     Running   0          53m

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/web-headless   ClusterIP   None           <none>        80/TCP    53m
service/web-service    ClusterIP   10.96.144.65   <none>        80/TCP    53m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-app   2/2     2            2           53m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/web-app-65d846d465   2         2         2       53m
        </pre>
</div>
<!-- This is a heading line -->  
    <div class="step">
        <h3>Step 5: Validate DNS Resolutions</h3>
        <h4>1. Forward Lookup for Service</h4>
        <p>Resolve the Service name <code style="color : name_color"> (web-service) </code> to its ClusterIP:</p>
        <pre lang="java" class="command">kubectl run -it --rm --image=busybox:1.28 --restart=Never --namespace=k8-dns dns-test \
            -- nslookup web-service.k8-dns.svc.cluster.local</pre>
        <p><strong>Expected Output:</strong></p> 
        <pre>
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:      web-service.k8-dns.svc.cluster.local
Address 1: 10.96.123.45 web-service.k8-dns.svc.cluster.local</pre>
</div>
<!-- This is a heading line -->  
  <div class="step">     
        <h3> 2. Forward Lookup for Headless Service </h3>
  <p> Resolve the Headless Service name <code style="color : name_color"> (web-headless) </code> to individual Pod IPs: </p>
        <pre class="command">kubectl run -it --rm --image=busybox:1.28 --restart=Never --namespace=k8-dns dns-test \
        -- nslookup web-headless.k8-dns.svc.cluster.local</pre>
        <p><strong>Expected Output:</strong></p>  
        <pre>Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:      web-headless.k8-dns.svc.cluster.local
Address 1: 10.244.1.2 web-app-12345-abcde.k8-dns.pod.cluster.local
Address 2: 10.244.1.3 web-app-67890-fghij.k8-dns.pod.cluster.local </pre>
  </div> 
  <!-- This is a heading line -->
    <div class="step"> 
          <h4>3. Reverse Lookup for Pod IPs</h4>
        <pre class="command">kubectl run -it --rm --image=busybox:1.28 --restart=Never --namespace=k8-dns dns-test -- nslookup &lt;pod-ip&gt;</pre>
        <p>Replace <code>&lt;pod-ip&gt;</code> with one of the Pod IPs from the Headless Service output <code style="color : name_color"> (e.g., 10.244.1.2). </code></p>
        <p><strong>Expected Output:</strong></p>
        <pre>
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:      2.1.244.10.in-addr.arpa
Address 1: 10.244.1.2 web-app-12345-abcde.k8-dns.pod.cluster.local</pre>
    </div>
<!-- This is a heading line -->  
    <div class="step">
        <h3>Step 6: Simulate Pod Deletion and Validate DNS</h3>
        <h3>1. Delete one of the Pods:</h3>
        <pre lang="java" class="command">kubectl delete pod &lt;pod-name&gt; -n k8-dns</pre>
            <p>The Deployment will automatically recreate the Pod with a new IP.</p>
         <h3>2. Validate DNS resolution again:</h3>
        <ul> 
            <ul>
                <li>Check the Headless Service resolution to ensure the new Pod IP is included.</li>
                <li>Perform a reverse lookup for the new Pod IP.</li>
            </ul>
        </ul>
    </div>
<!-- This is a heading line -->  
<div class="step"> 
    <h2>Key Takeaways</h2>
    <ul>
        <ul>
            <li><strong>Service DNS:</strong> Provides a stable endpoint for accessing Pods, even when Pod IPs change.</li>
            <li><strong>Headless Service DNS:</strong> Allows direct resolution of individual Pod IPs.</li>
            <li><strong>Reverse DNS:</strong> Maps Pod IPs back to their DNS names.</li>
            <li><strong>CoreDNS:</strong> Automatically updates DNS records when Pods are recreated.</li>
        </ul>
     </ul>
</div>
<div>
     <a href="https://github.com/kubernetes/dns/blob/master/docs/specification.md" target="_blank">DNS Record </a> and
    <a href="https://github.com/kubernetes/dns" target="_blank">K8s DNS Definitions</a> for more details and
    <a href="https://github.com/kubernetes/examples/blob/master/staging/cluster-dns/README.md" target="_blank"> for a example </a> how to use Kubernetes DNS <button onclick="alert('You liked the article!')">👍 do you this article? </button>
</div>
