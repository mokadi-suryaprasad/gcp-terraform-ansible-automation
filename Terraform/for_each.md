### 9) for_each Loop (Important for Scaling)

The `for_each` meta-argument is used to **dynamically create multiple resources** from a single resource block.

---

### **📝 Real-Time Example: Creating Multiple VM Instances**

#### **variables.tf**
```hcl
variable "vm_instances" {
  description = "List of VM names with machine types"
  type = map(string)
  default = {
    app-server   = "e2-medium"
    db-server    = "e2-small"
    cache-server = "e2-micro"
  }
}
```

#### **main.tf**
```hcl
resource "google_compute_instance" "vm" {
  for_each = var.vm_instances

  name         = each.key
  machine_type = each.value
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }
}
```

---

### **💡 Explanation**
| Element | Meaning |
|--------|---------|
| `for_each = var.vm_instances` | Loops through each key-value pair of the map |
| `each.key` | VM Name (e.g., app-server, db-server) |
| `each.value` | Machine type (e.g., e2-medium, e2-small) |

Terraform will generate resources like:
```
google_compute_instance.vm["app-server"]
google_compute_instance.vm["db-server"]
google_compute_instance.vm["cache-server"]
```

---

### **✅ Optional Output**
```hcl
output "created_vm_names" {
  value = keys(google_compute_instance.vm)
}
```

---

### **🎯 Result**
This single resource block **dynamically creates multiple virtual machines** without duplicating code — making your Terraform **clean, scalable, and easy to maintain**.
