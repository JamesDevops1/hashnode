---
title: "Networking in Kubernetes: The Part Everyone Pretends to Understand"
datePublished: 2026-04-25T00:38:15.626Z
cuid: cmodm23ba003a2blnb4yjf5s1
slug: networking-in-kubernetes
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/f7eb2717-dc81-4136-904f-0366778be4b7.jpg

---

There’s a point in every Kubernetes journey where things stop being “just YAML” and start becoming… weird.

Your pods are running. Your deployments look healthy. But something feels off.

*   A service can’t reach another service
    
*   DNS randomly fails
    
*   Traffic doesn’t go where you expect
    
*   Everything works locally, but breaks in the cluster
    

And suddenly, you’re staring at Kubernetes networking — the part everyone nods at… but quietly avoids explaining.

Let’s fix that.

### **First, Reset Your Mental Model**

Forget VMs. Forget traditional networking for a second.

Kubernetes networking is built on **a few non-negotiable rules**:

1.  **Every Pod gets its own IP**
    
2.  **Pods can talk to each other directly (no NAT)**
    
3.  **Nodes can talk to Pods, and vice versa**
    
4.  **The system assumes a flat network**
    

That’s it. Everything else builds on top of this.

This is very different from traditional setups where containers are hidden behind host IPs or NAT layers.

In Kubernetes, **Pods are first-class citizens on the network.**

* * *

### **The Pod Network (Where Everything Starts)**

Every Pod gets:

*   A unique IP address
    
*   Its own network namespace
    
*   A virtual interface
    

From the cluster’s perspective, **a Pod is like a mini machine on the network**.

So if you have:

*   Pod A → `10.244.1.5`
    
*   Pod B → `10.244.2.8`
    

Pod A can talk to Pod B directly.

No port mapping. No tricks.

Just:

```plaintext
curl http://10.244.2.8
```

But here’s the catch:

👉 Pods are **ephemeral**.

They die. They get rescheduled. Their IPs change.

So relying on Pod IPs directly is a bad idea.

## Services: The Stable Front Door

This is where **Services** come in.

A Service gives you:

*   A **stable IP**
    
*   A **DNS name**
    
*   Built-in **load balancing**
    

Instead of calling Pods directly, you call the Service:

```plaintext
curl http://user-service
```

Behind the scenes:

*   Kubernetes finds all matching Pods
    
*   Distributes traffic across them
    

Think of it like a smart internal load balancer.

### **Types of Services (Quickly)**

*   **ClusterIP** → internal-only communication
    
*   **NodePort** → exposes service on node IP + port
    
*   **LoadBalancer** → integrates with cloud providers
    

If you’re inside the cluster, **ClusterIP is your default friend**.

### **DNS: The Invisible Glue**

When you write:

```plaintext
curl http://user-service
```

You’re relying on Kubernetes DNS.

Under the hood (via **CoreDNS**):

```plaintext
user-service.default.svc.cluster.local
```

gets resolved to the Service IP.

That’s why:

*   Pods don’t need to know IPs
    
*   Everything is discoverable by name
    

But when DNS breaks… everything breaks.

### **The CNI Plugin (The Part People Skip)**

Here’s where things get real.

Kubernetes **does NOT implement networking itself**.

It delegates that responsibility to something called a **CNI (Container Network Interface)** plugin.

Popular ones include:

*   Flannel
    
*   Calico
    
*   Cilium
    

These plugins are responsible for:

*   Assigning Pod IPs
    
*   Routing traffic between nodes
    
*   Enforcing network policies
    

So when networking fails, it’s often not “Kubernetes” — it’s your CNI.

That’s an important distinction.

## Ingress: Getting Traffic Into the Cluster

Services handle internal traffic.

But what about **external users**?

That’s where **Ingress** comes in.

Ingress lets you define rules like:

*   `/api` → backend service
    
*   `/auth` → auth service
    

Instead of exposing multiple NodePorts, you get:

*   One entry point
    
*   Clean routing rules
    
*   TLS termination
    

Usually powered by something like:

*   NGINX Ingress Controller
    
*   Traefik
    

## Network Policies: The Missing Security Layer

By default, Kubernetes networking is wide open.

Any Pod can talk to any Pod.

That’s… not great.

Network Policies let you define rules like:

Only allow frontend → backend Block everything else Restrict external access

Example idea:

allow: from: - podSelector: frontend to: - podSelector: backend

But here’s the catch:

👉 Network policies only work if your CNI supports them (e.g., Calico, Cilium).

## Where Things Usually Go Wrong

This is the part no one tells you upfront.

Most Kubernetes networking issues fall into these buckets:

1.  DNS Issues Service name not resolving CoreDNS misconfigured
    
2.  CNI Problems Pods can’t communicate across nodes IP ranges overlapping
    
3.  Service Misconfigurations Wrong selectors No endpoints behind the service
    
4.  Network Policies Blocking Traffic Silent failures No logs by default
    
5.  Ingress Confusion Routing rules not matching TLS misconfigured
    

## A Simple Way to Debug Networking

When things break, don’t panic. Follow a path:

1.  **Can the Pod reach itself?**
    
2.  **Can it reach another Pod (via IP)?**
    
3.  **Can it reach the Service (via DNS)?**
    
4.  **Is DNS resolving correctly?**
    
5.  **Are endpoints attached to the Service?**
    
6.  **Is a Network Policy blocking traffic?**
    

Most issues reveal themselves if you follow the chain.

## The Truth Most People Won’t Say

Kubernetes networking feels complex not because it’s inherently complicated…

…but because:

*   It’s **layered**
    
*   It’s **abstracted**
    
*   And when it breaks, the error messages are terrible
    

Once you understand the flow:

**Pod → Service → DNS → CNI → Ingress**

…it stops feeling like magic and starts feeling like a system you can reason about.

## Final Thought

You don’t need to memorize everything about Kubernetes networking.

But you do need to understand:

*   Why Pods have IPs
    
*   Why Services exist
    
*   How DNS ties it together
    
*   And where the CNI fits in
    

Because once your system grows beyond a single service…

Networking stops being optional knowledge.

It becomes the difference between:

> “Everything is running”  
> and  
> “Everything is actually working”