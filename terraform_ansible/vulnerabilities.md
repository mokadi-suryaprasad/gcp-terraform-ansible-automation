
# 🔐 Finding Vulnerabilities in GCP 

## What Are Vulnerabilities?
Vulnerabilities are **security weaknesses** in:
- Servers
- Containers
- Applications
- Cloud configuration

Attackers can use these weaknesses to hack systems, so we must find and fix them.

---

## ✅ How to Find Vulnerabilities in GCP

### 1) **Security Command Center (SCC)**
This is a **dashboard** in GCP that shows:
- Misconfigurations
- Exposed resources
- Weak IAM permissions
- Storage bucket exposures

**Path:** GCP Console → Security → Security Command Center

---

### 2) **VM Vulnerability Reports**
Used to find OS/package vulnerabilities inside **GCE VM instances**.

Path:
```
Compute Engine → VM Manager → Vulnerability Reports
```

Make sure the agent is installed:
```
sudo apt-get install google-osconfig-agent
sudo systemctl restart google-osconfig-agent
```

---

### 3) **Container Image Scanning (Artifact Registry)**
If you use Docker containers:
- GCP automatically scans your images for vulnerabilities

Path:
```
Artifact Registry → Images → Vulnerabilities Tab
```

---

### 4) **Web Security Scanner**
This finds vulnerabilities in **websites and applications**, such as:
- XSS
- SQL Injection
- Broken Authentication

Path:
```
Security → Web Security Scanner
```

---

### 5) **IaC Security Scanning (Terraform / Kubernetes / YAML)**

Use tools like:
```
tfsec
checkov
trivy config
```

Example (scan Terraform):
```
tfsec .
```

---

## 🧠 One-Line Summary
| Cloud Layer | Tool Used |
|------------|-----------|
| Cloud Config & IAM | **Security Command Center** |
| VM OS Level | **VM Vulnerability Reports** |
| Container Images | **Artifact Registry Scanning** |
| Web Apps | **Web Security Scanner** |
| Terraform / K8s Files | **tfsec / checkov** |

---

## ✅ Interview Answer (Short & Clear)
**"We find vulnerabilities in GCP using Security Command Center for cloud configuration issues, VM Manager for OS-level vulnerabilities, Artifact Registry for container image CVEs, Web Security Scanner for application checks, and tools like tfsec/checkov for IaC scanning."**

