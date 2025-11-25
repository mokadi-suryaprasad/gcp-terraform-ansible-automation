

# ⚙️ Configuration Management in Ansible

## 🧩 What is Configuration Management?

Configuration Management means **making sure all servers have the correct software, settings, and configuration — automatically and consistently.**

Instead of logging into each server manually, **Ansible** applies the same configuration to many servers at once using **Playbooks (YAML files)**.

| Section           | Explanation                                  |
| ----------------- | -------------------------------------------- |
| **Setup**         | Prepare control node, SSH keys, Python.      |
| **Installation**  | Install Ansible using apt/dnf/pip.           |
| **Settings**      | Configure inventory, ansible.cfg, groups.    |
| **Configuration** | Use playbooks and modules to manage servers. |


---

## 🟢 Why We Use Ansible for Configuration Management

| Benefit | Explanation |
|--------|-------------|
| **Agentless** | No need to install software on servers. Uses SSH. |
| **Idempotent** | Running the same playbook again will not break anything. |
| **Automated & Consistent** | All servers get the exact same configuration. |
| **Scalable** | Works for 1 server or thousands. |

---

## 🏡 Real-Time Example Scenario

Your company has **3 web servers**, and you need to:

- Install **NGINX**
- Deploy the company website
- Start the web service and keep it running

Doing this manually on each server can cause mistakes.  
So we use **Ansible Playbooks** to automate it.

---

## ✅ Example Playbook (Configuration Management)

**File: `setup-webserver.yml`**

```yaml
- name: Configure Web Server
  hosts: webservers
  become: yes

  tasks:
    - name: Install NGINX
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy Website Home Page
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
        mode: "0644"

    - name: Start NGINX Service
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## 📌 Inventory File (Server List)

**File: `inventory.ini`**
```
[webservers]
10.10.1.11
10.10.1.12
10.10.1.13
```

---

## ▶️ Run the Playbook

```bash
ansible-playbook -i inventory.ini setup-webserver.yml
```

---

## 🎯 What This Did

| Step | Action |
|------|--------|
| Ansible connected to all servers | via SSH |
| Installed **NGINX** | automatically |
| Copied website files | same on all servers |
| Ensured service runs every reboot | reliable setup |

---

## 🧠 One Line Interview Answer

> **Configuration Management in Ansible means automating the setup, software installation, and configuration of servers to ensure they remain consistent and reliable across all environments.**

