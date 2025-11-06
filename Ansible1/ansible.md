
# 🐧 What is Ansible?

**Ansible** is an open-source **IT automation and configuration management tool**.  
It helps in automating tasks such as:

- Installing software
- Configuring servers
- Deploying applications
- Managing multiple servers at once

---

## ✅ Key Features of Ansible

| Feature | Description |
|--------|-------------|
| **Agentless** | No need to install any agent on servers. Uses SSH to connect. |
| **Easy to Learn** | Uses simple `YAML` files called *playbooks*. |
| **Scalable** | Can manage 1 or thousands of servers the same way. |
| **Idempotent** | Running the same playbook again does not break anything. |
| **Cross-platform** | Works on Linux, Windows, Cloud, On-prem servers. |

---

## 🧠 How Ansible Works

1. You write automation steps in a **Playbook** (`.yml` file).
2. Ansible connects to servers using **SSH**.
3. It executes tasks on those servers.
4. All servers receive the same configuration—**automatically and consistently**.

---

## 📁 Example Inventory (Server List)

```
[webservers]
192.168.1.10
192.168.1.11
```

---

## 📝 Example Playbook

```yaml
- name: Install and Start NGINX
  hosts: webservers
  become: yes

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

Run:
```
ansible-playbook -i inventory.ini webserver.yml
```

---

## 🎯 One-Line Summary

> **Ansible is a tool used to automate server setup and configuration, ensuring consistency and speed across multiple systems.**

🔹 **Is Ansible Push or Pull Based?**

Ansible is **Push-Based** by default — the control node pushes configurations to the target servers over SSH.

> It also supports **Pull-Based mode** using `ansible-pull`, but this is used less often.

**One-Line Interview Answer:**  
**Ansible is primarily a Push-based configuration management tool.**
