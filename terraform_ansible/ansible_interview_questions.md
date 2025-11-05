
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

## What are Facts in Ansible?
**Facts** are **system information** that Ansible collects automatically from managed servers before running any tasks.

These details help playbooks make **smart decisions** based on the system.

---

## Examples of Information (Facts) Collected
- Hostname
- Operating System type
- IP Address
- CPU, RAM, Disk details
- Network interfaces
- Kernel version
- Timezone

---

## How Ansible Collects Facts
Ansible collects facts using the **`setup` module**.

### View Facts of a Server
```bash
ansible all -m setup
```

This will display **all details** available about the server.

---

## Why Use Facts?

| Benefit | Explanation |
|--------|-------------|
| Dynamic Playbooks | Playbooks adapt to different servers automatically |
| Conditional Tasks | Run tasks only if system matches condition |
| Better Automation | No need to hard-code server info |

---

## Example: Using a Fact in a Playbook

```yaml
- name: Print OS type
  hosts: all
  tasks:
    - name: Show OS
      debug:
        msg: "This server is running {{ ansible_os_family }}"
```

Output Example:
```
This server is running Debian
```

---

## One-Line Interview Answer
**Facts in Ansible are automatically collected system details from servers, used to make playbooks dynamic and smart.**


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

## 🛡️What is Ansible Vault?

## What is Ansible Vault?
**Ansible Vault** is a feature in Ansible that is used to **secure and encrypt sensitive data**, such as:
- Passwords
- API Keys
- Database credentials
- Cloud service account keys

This keeps secrets **safe** even if your files are stored in GitHub or shared with others.

---

## Why Do We Use Ansible Vault?

| Reason | Explanation |
|-------|-------------|
| Security | Prevents others from reading sensitive information |
| Safe Version Control | Encrypted data can be pushed to Git safely |
| Compliance | Helps follow security and audit policies |

---

## Basic Commands

| Action | Command |
|-------|---------|
| Create a new encrypted file | `ansible-vault create secrets.yml` |
| Encrypt an existing file | `ansible-vault encrypt secrets.yml` |
| View encrypted file | `ansible-vault view secrets.yml` |
| Edit encrypted file | `ansible-vault edit secrets.yml` |
| Decrypt a file | `ansible-vault decrypt secrets.yml` |

---

## Example: Storing a Password

### Step 1 — Create and encrypt a secrets file
```bash
ansible-vault create secrets.yml
```

### Step 2 — Add a variable inside the file
```yaml
db_password: MyStrongPassword123
```

### Step 3 — Use the variable in Playbook
```yaml
- name: Example using vault variable
  hosts: dbserver
  vars_files:
    - secrets.yml
  tasks:
    - name: Show password
      debug:
        msg: "Database password is {{ db_password }}"
```

### Step 4 — Run the playbook with vault password
```bash
ansible-playbook play.yml --ask-vault-pass
```

---



## One-Line Interview Answer
**Ansible Vault is used to encrypt and protect sensitive information such as passwords and keys inside Ansible automation files.**


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
