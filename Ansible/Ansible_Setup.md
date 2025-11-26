## Ansible Setup – Easy Steps

Ansible has two types of machines:

- **Control Node** → The machine where Ansible is installed  
- **Managed Nodes** → The machines we want to configure using Ansible

Ansible works using **SSH**.  
No agent is required on managed nodes.

---

## 🟦 1. Requirements

### Control Node
- Linux machine  
- Install Ansible  
- SSH key generated  

### Managed Node
- SSH enabled  
- Python installed  
- Allow login for Ansible user  

---

## 🟩 2. Install Ansible on Control Node

### For Ubuntu / Debian
```bash
sudo apt update
sudo apt install ansible -y
```

### For RHEL / CentOS
```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

Check version:
```bash
ansible --version
```

---

## 🟧 3. Create “ansible” User on Managed Nodes

Do this on **every managed node**:

```bash
sudo useradd ansible
sudo passwd ansible
```

Give sudo access:

```bash
sudo visudo -f /etc/sudoers.d/ansible
```

Add line:

```
ansible ALL=(ALL) NOPASSWD: ALL
```

---

## 🟨 4. Enable Passwordless SSH

### Step 1: Generate SSH Key on Control Node

```bash
ssh-keygen -t rsa
```

Press **Enter** for all prompts.

### Step 2: Copy key to managed nodes

```bash
ssh-copy-id ansible@<managed-node-ip>
```

Example:
```bash
ssh-copy-id ansible@192.168.1.20
ssh-copy-id ansible@192.168.1.21
```

Test login:
```bash
ssh ansible@192.168.1.20
```

You should get login **without password**.

---

## 🟫 5. Check Python on Managed Nodes

Ansible needs Python.

```bash
python3 --version
```

If missing:

**Ubuntu:**
```bash
sudo apt install python3 -y
```

**CentOS:**
```bash
sudo yum install python3 -y
```

---

## 🟪 6. Create Inventory File (Control Node)

Make a folder:
```bash
mkdir ~/ansible-project
cd ~/ansible-project
```

Create inventory:
```bash
nano inventory.ini
```

Add:

```ini
[webservers]
192.168.1.20
192.168.1.21

[all:vars]
ansible_user=ansible
ansible_become=yes
```

---

## 🟥 7. Create ansible.cfg File

```bash
nano ansible.cfg
```

Add:

```ini
[defaults]
inventory = ./inventory.ini
host_key_checking = False
retry_files_enabled = False
```

---

## 🟦 8. Test Connection

```bash
ansible all -m ping
```

Expected output:

```
SUCCESS
"ping": "pong"
```

---

## 🟩 9. Run a Simple Playbook

Create file:
```bash
nano test.yml
```

Add:

```yaml
- name: Test Ansible Setup
  hosts: all
  become: yes

  tasks:
    - name: Ping all nodes
      ping:
```

Run:
```bash
ansible-playbook test.yml
```

---

## Setup Complete

Your Ansible environment is fully ready!
