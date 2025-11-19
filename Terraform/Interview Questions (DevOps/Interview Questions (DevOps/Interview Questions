# Interview Questions (DevOps/GCP)

## 1️⃣ What do you understand by VPC Service Controls?
VPC Service Controls add an extra security boundary around Google Cloud services (like Cloud Storage, BigQuery, Pub/Sub).  
They prevent **data exfiltration** by blocking access from outside the defined security perimeter.  
This helps protect data even if credentials are leaked.

---

## 2️⃣ Difference between Pod, Deployment, and Service in Kubernetes

### **Pod**
- Smallest unit in Kubernetes  
- Contains one or more containers  
- Ephemeral (dies when failed unless controlled by a higher object)

### **Deployment**
- Manages Pods  
- Ensures desired number of replicas  
- Handles rolling updates and rollbacks  
- Provides self-healing by recreating failed pods

### **Service**
- Provides stable networking to pods  
- Types: ClusterIP, NodePort, LoadBalancer  
- Decouples network access from pod lifecycle

---

## 3️⃣ What is a Dynamic Block in Terraform?
A **dynamic block** is used when you need to create multiple nested blocks inside a resource using loops.  
Useful when the number of rules, settings, or configurations is not fixed.

### Example:
```hcl
dynamic "port" {
  for_each = var.ports
  content {
    number = port.value
  }
}
```

---

## 4️⃣ What does agentless mean in Ansible?
Ansible is **agentless**, meaning:
- No software/agent needs to be installed on target machines  
- Uses **SSH** for Linux and **WinRM** for Windows  
- Very easy to manage and operate

---

## 5️⃣ Will you use external IP for an instance in GCP as best practice?
**No**, it is not recommended.

### Best Practices:
- Use **internal IPs**  
- Use **Identity-Aware Proxy (IAP)**  
- Use **Cloud NAT** for outbound access  
- Use **Bastion Host** for controlled SSH/RDP access

This reduces exposure and improves security.

---

## 6️⃣ Difference between External HTTPS Load Balancer and External TCP Proxy Load Balancer

### External HTTPS Load Balancer (L7)
- Layer 7 (HTTP/HTTPS)  
- SSL termination  
- URL/path-based routing  
- Global load balancing  
- Best for web applications

### External TCP Proxy Load Balancer (L4)
- Layer 4 (TCP)  
- Handles encrypted TCP traffic that is not HTTP  
- Best for SSL/TLS-based non-HTTP apps

---

## 7️⃣ What is Workload Identity in GCP?
Workload Identity allows Kubernetes workloads (GKE pods) to authenticate to Google Cloud **without service account keys**.

It maps:
```
Kubernetes Service Account → Google Cloud Service Account
```

Benefits:
- No key files  
- More secure  
- Automatic identity binding

---

## 8️⃣ How will you host an app running on GCP to the internet?

### Common Ways:
- External HTTPS Load Balancer  
- Cloud Run (automatically gives HTTPS URL)  
- GKE Service type LoadBalancer  
- App Engine  
- VM behind External Load Balancer  

Best practice → Avoid exposing VM directly using external IP.

---
