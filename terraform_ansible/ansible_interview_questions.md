
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

# 📂 Ansible Roles 

## What is a Role in Ansible?
A **Role** in Ansible is a way to **organize playbooks** into separate folders.  
Roles help you keep your automation **clean, reusable, and easy to manage**.

Instead of writing everything inside one playbook, Roles split configuration into parts.

---

## Why Use Roles?

| Benefit | Description |
|--------|-------------|
| Clean Structure | Files are organized properly |
| Reusable | The same role can be used in many projects |
| Easy to Maintain | Updates are done in one place |
| Best Practice | Used in real company DevOps projects |

---

## Structure of a Role

A typical Role looks like this:

```
roles/
  nginx/
    tasks/
      main.yml
    vars/
      main.yml
    files/
      index.html
    templates/
      site.conf.j2
    handlers/
      main.yml
```

### Purpose of Each Folder
| Folder | Purpose |
|--------|---------|
| `tasks/` | Contains the main tasks to run |
| `vars/` | Variables used in the role |
| `files/` | Files to copy to remote servers |
| `templates/` | Jinja2 templates for configuration files |
| `handlers/` | Restart/Reload services when needed |

---

## Example Usage in Playbook

```yaml
- hosts: webservers
  roles:
    - nginx
```

This tells Ansible to **run the nginx role** on all servers in the `webservers` group.

---

## One-Line Interview Answer
**Roles in Ansible are used to organize playbooks into structured folders, making configuration clean, reusable, and easy to manage.**


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
