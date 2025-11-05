
# 🏗️ Terraform + 🏡 Ansible — Real-Time, Real-World Scenario

## 🌍 Scenario 1: Deploying a Web Application on GCP VM

### 🔹 Step 1 — Terraform Work (Infrastructure Provisioning)

A company decides to host an **internal HR portal** on **Google Cloud**.  
They need the following infrastructure:

| Resource | Purpose |
|---------|---------|
| VPC | Secure network boundary |
| Subnets | Separate environments / workloads |
| VM Instance | Server to host the web app |
| Firewall Rule | Allow HTTP (port 80) traffic |

**Terraform creates:**
- VPC
- Subnets
- Compute Engine VM instance
- Public IP assigned to the VM
- Firewall rule to allow inbound HTTP (80)

After running:
```bash
terraform apply
```
✅ The VM exists  
❌ But it has **no software installed yet**

---

### 🔹 Step 2 — Ansible Work (Configuration & Application Deployment)

Now we need to configure the VM and deploy the application.

**Tasks Ansible will perform:**
- Install **NGINX**
- Copy website/application files
- Start the NGINX service
- Enable service on reboot

#### ✅ Ansible Playbook Example (`install-nginx.yml`)

```yaml
- name: Configure Web Server on GCP VM
  hosts: webserver
  become: yes

  tasks:
    - name: Install NGINX
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy Website Files
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
        mode: "0644"

    - name: Start and Enable NGINX
      service:
        name: nginx
        state: started
        enabled: yes
```

#### Run the playbook:
```bash
ansible-playbook install-nginx.yml
```

---

## 🎯 Summary

| Tool | Responsibility |
|------|---------------|
| **Terraform** | Creates the VM + Network + Firewall |
| **Ansible** | Installs software + Deploys web application |

> **Terraform builds the server, Ansible makes it usable.**

---

