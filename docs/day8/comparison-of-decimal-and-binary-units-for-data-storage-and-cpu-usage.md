### **Comparison of Decimal (SI) and Binary (IEC) Units for Data Storage and CPU Usage**

#### **Data Storage Units**

##### Decimal (SI) Units for Data Storage:
- **1 Gigabyte (1 GB)** = 1,000,000,000 bytes
- **1 Megabyte (1 MB)** = 1,000,000 bytes
- **1 Kilobyte (1 KB)** = 1,000 bytes

##### Binary (IEC) Units for Data Storage:
- **1 Gibibyte (1 GiB)** = 1,073,741,824 bytes (2^30)
- **1 Mebibyte (1 MiB)** = 1,048,576 bytes (2^20)
- **1 Kibibyte (1 KiB)** = 1,024 bytes (2^10)

---

#### **CPU Usage Units**

CPU usage is generally measured in terms of processing power allocated or consumed by a process. The most common unit for CPU usage is **millicores**, which is derived from the CPU cores of the machine.

##### Units for CPU Usage:
- **1 CPU (core)** = 1 CPU core or 1,000 millicores
- **1 millicore** = 1/1000 of a CPU core

These units are particularly used in Kubernetes or container orchestration systems where resources are allocated as a fraction of CPU cores.

---

### **Combined Summary:**
- **Data Storage**:
  - **1 GB** = 1,000,000,000 bytes (SI)
  - **1 GiB** = 1,073,741,824 bytes (IEC)
  
- **CPU Usage**:
  - **1 CPU core** = 1,000 millicores
  - **1 millicore** = 1/1000 of a CPU core

Both data storage and CPU usage units help manage and allocate resources precisely, whether you're tracking storage in gigabytes or CPU usage in millicores.
