## **Memory & CPU Units**

### **SI vs IEC: Understanding the Difference**

**SI (International System of Units)** and **IEC (International Electrotechnical Commission)** refer to two different standards for measuring data and computing resources.

---

#### **SI Units (International System of Units)**
- **Definition**: SI is the modern form of the metric system, and it is the most widely used system of units. When applied to data storage, SI units follow decimal-based (base-10) measurement.
  
  **Examples**:
  - 1 Kilobyte (KB) = 1,000 bytes
  - 1 Megabyte (MB) = 1,000,000 bytes
  - 1 Gigabyte (GB) = 1,000,000,000 bytes
  
  These units are often used in marketing materials for storage devices, such as hard drives and flash drives.

---

#### **IEC Units (International Electrotechnical Commission)**
- **Definition**: IEC units are binary-based (base-2) units used to represent data more accurately in the context of computing, where memory and processing use powers of 2. These units are designed to eliminate confusion between decimal-based and binary-based measurements.

  **Examples**:
  - 1 Kibibyte (KiB) = 1,024 bytes (2^10)
  - 1 Mebibyte (MiB) = 1,048,576 bytes (2^20)
  - 1 Gibibyte (GiB) = 1,073,741,824 bytes (2^30)
  
  These units are commonly used by operating systems when displaying the actual storage space available on a disk or memory.

---

#### **Key Differences Between SI and IEC**
- **SI (Base-10)**: Uses powers of 10 (1,000 bytes in a Kilobyte, etc.)
- **IEC (Base-2)**: Uses powers of 2 (1,024 bytes in a Kibibyte, etc.)


#### **Data Storage Units**

---
##### Decimal (SI) Units for Data Storage:
- **1 Gigabyte (1 GB)** = 1,000,000,000 bytes
- **1 Megabyte (1 MB)** = 1,000,000 bytes
- **1 Kilobyte (1 KB)** = 1,000 bytes

##### Binary (IEC) Units for Data Storage:
- **1 Gibibyte (1 GiB)** = 1,073,741,824 bytes (2^30)
- **1 Mebibyte (1 MiB)** = 1,048,576 bytes (2^20)
- **1 Kibibyte (1 KiB)** = 1,024 bytes (2^10)

---

### **CPU Usage Units**

CPU usage is generally measured in terms of processing power allocated or consumed by a process. The most common unit for CPU usage is **millicores**, which is derived from the CPU cores of the machine.

##### Units for CPU Usage:
- **1 CPU (core)** = 1 CPU core or 1,000 millicores
- **1 millicore** = 1/1000 of a CPU core

These units are particularly used in Kubernetes or container orchestration systems where resources are allocated as a fraction of CPU cores.

