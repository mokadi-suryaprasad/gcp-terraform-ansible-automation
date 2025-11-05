
# 📝 Ansible Interview Questions

## 1) Why do we use Ansible?
We use Ansible to **automate tasks** so we don't have to do them manually on every server.  
Example: installing software or configuring settings on many servers at once.

---

## 2) Is Ansible Push or Pull based?
Ansible is **Push-Based**.  
The control machine **pushes** changes to other servers using **SSH**.

---

## 3) What is a Playbook?
A Playbook is a **YAML file** that tells Ansible what tasks to perform.

Example:
```yaml
- name: Install nginx
  hosts: webservers
  tasks:
    - name: Install package
      apt:
        name: nginx
        state: present
```

---

## 4) What is an Inventory file?
The inventory file contains the **list of servers** Ansible will manage.

Example:
```
[webservers]
10.10.1.11
10.10.1.12
```

---

## 5) What are Ansible Modules?
Modules are **pre-built commands** that Ansible uses to perform actions.

| Module name | Purpose |
|------------|---------|
| apt / yum | Install packages |
| service | Start/stop services |
| copy | Copy files to server |

---

## 6) What is Idempotency?
Idempotency means:
Even if we **run the playbook many times**, the server state **remains correct** and does not get duplicated or broken.

---

## 7) What are Roles in Ansible?
Roles help **organize playbooks** into folders.  
They make the configuration **clean and reusable**.

---

## 8) How does Ansible connect to servers?
Ansible uses:
- **SSH** for Linux servers
- **WinRM** for Windows servers

No agent required.

---

## 9) How do you test an Ansible Playbook?
- Run it in **Dev**, then **Stage**, then **Prod**
- Use `ansible-playbook --check` for a **dry run**
- Use **ansible-lint** to check for mistakes

---

## 10) Difference Between Ansible, Puppet, and Chef?

| Feature | Ansible | Puppet | Chef |
|--------|---------|--------|------|
| Mode | **Push-Based** | Pull-Based | Pull-Based |
| Language | YAML | DSL | Ruby |
| Agent Needed? | **No** | Yes | Yes |
| Learning Curve | Very Easy | Medium | Hard |

---

## ⭐ One-Line Interview Answer
**Ansible is a simple, agentless, and push-based automation tool that configures and manages multiple servers using YAML playbooks.**
