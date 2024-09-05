https://kodekloud.com/blog/ebpf-essentials-devops/#comparing-ebpf-to-the-sidecar-approach

Here's the revised version of the document with links to the Table of Contents (TOC) and links back to the TOC after each section:

---

# Table of Contents

1. [Understanding eBPF](#understanding-ebpf)
   - [Understanding the Linux Kernel](#understanding-the-linux-kernel)
   - [Why Changing the Kernel Isn’t Easy](#why-changing-the-kernel-isnt-easy)
   - [The Challenge with Kernel Modules](#the-challenge-with-kernel-modules)
   - [Enter eBPF: A Safer, More Flexible Approach](#enter-ebpf-a-safer-more-flexible-approach)
   - [eBPF in Action](#ebpf-in-action)
   
2. [Writing eBPF Programs](#writing-ebpf-programs)
   - [What is eBPF?](#what-is-ebpf)
   - [Programming eBPF: Language Choices](#programming-ebpf-language-choices)
   - [Kernel vs. User Space Code](#kernel-vs-user-space-code)
   - [Attaching eBPF Programs to Events](#attaching-ebpf-programs-to-events)
   - [eBPF Maps: Bridging Kernel and User Space](#ebpf-maps-bridging-kernel-and-user-space)
   - [A Glimpse into Opensnoop: An eBPF Example](#a-glimpse-into-opensnoop-an-ebpf-example)

3. [The Rise of Cloud-Native and eBPF](#the-rise-of-cloud-native-and-ebpf)
   - [One Kernel Rules Them All](#one-kernel-rules-them-all)
   - [Comparing eBPF to the Sidecar Approach](#comparing-ebpf-to-the-sidecar-approach)
   - [Understanding eBPF and Process Isolation](#understanding-ebpf-and-process-isolation)
   
4. [eBPF for Enhanced Cloud-Native Tools](#ebpf-for-enhanced-cloud-native-tools)
   - [eBPF in Networking](#ebpf-in-networking)
   - [Observability: Seeing All with eBPF](#observability-seeing-all-with-ebpf)
   - [Security: eBPF at the Forefront](#security-ebpf-at-the-forefront)
   - [Wrapping Up on eBPF's Power](#wrapping-up-on-ebpf-power)

---

## I. Understanding eBPF

### Understanding the Linux Kernel
At the heart of your Linux operating system sits the kernel. It's like a mediator between the applications you use and the physical hardware of your computer. Whenever an app needs to do something with the hardware, like access memory or manage files, it talks to the kernel. We don't usually see this interaction because it happens behind the scenes, and programming languages handle it for us.

[Back to TOC](#table-of-contents)

### Why Changing the Kernel Isn’t Easy
Changing the Linux kernel is a massive task. It's not just about writing code; it's also about convincing the Linux community (especially its creator, Linus Torvalds) that your change benefits everyone. The kernel itself is enormous – about 30 million lines of code! Even if your code does make it in, it could be years before it's widely used, as Linux distributions often use older kernel versions for stability and security.

[Back to TOC](#table-of-contents)

### The Challenge with Kernel Modules
One way to change the kernel is by using kernel modules. These can be loaded and unloaded as needed. But there's a catch: kernel programming is tricky. If something goes wrong, it could crash the entire system. Plus, there are security concerns. We must be sure that a kernel module won't be exploited for malicious purposes.

[Back to TOC](#table-of-contents)

### Enter eBPF: A Safer, More Flexible Approach
This is where eBPF (extended Berkeley Packet Filter) shines. It lets us safely and dynamically inject new features into the kernel. Unlike traditional methods, eBPF programs are checked rigorously by the eBPF verifier for safety before they're allowed to run. This verification process makes sure eBPF programs won't crash the system or access unauthorized memory areas.

[Back to TOC](#table-of-contents)

### eBPF in Action
With eBPF, you can quickly and safely add new functions to the kernel. For example, you can track every time a folder is created or file is opened on your system. Using eBPF, you can write a program to monitor file-open or file-create activities, regardless of when the processes were started.

[Back to TOC](#table-of-contents)

---

## II. Writing eBPF Programs

### What is eBPF?
eBPF, which stands for extended Berkeley Packet Filter, is a Linux kernel technology that allows users to run custom programs within the kernel space without changing the kernel code. This offers an incredible advantage, enabling functionalities like performance analysis, network monitoring, and security.

[Back to TOC](#table-of-contents)

### Programming eBPF: Language Choices
While the kernel takes eBPF programs in bytecode, writing bytecode by hand is tedious. Instead, developers generally use higher-level languages like:

- **C:** The most widely adopted language, as the Linux kernel is written in C.
- **Rust:** A newer addition, gaining traction due to its safety features.

[Back to TOC](#table-of-contents)

### Kernel vs. User Space Code
For eBPF tools, there are typically two parts:

- **Kernel Space (eBPF program):** This runs within the kernel.
- **User Space Code:** This interacts with the eBPF program, passing in configurations and displaying the program's data. It can be written in languages like C, Go, Rust, or Python.

[Back to TOC](#table-of-contents)

### Attaching eBPF Programs to Events
Once an eBPF program is in the kernel, it needs an event to trigger it. Common triggers include:

- Entry/Exit from functions using kprobes and fentry/fexit.
- Tracepoints within the kernel.
- Perf Events for performance data collection.
- Linux Security Module Interface for security policies.
- Network Interfaces with eXpress Data Path (XDP) for efficient networking.

[Back to TOC](#table-of-contents)

### eBPF Maps: Bridging Kernel and User Space
eBPF Maps are key-value stores that allow data passage between eBPF programs and the user space. They're crucial for:

- Storing metrics and data about an event.
- Holding configuration details.
- Coordinating information across multiple kernel events.

[Back to TOC](#table-of-contents)

### A Glimpse into Opensnoop: An eBPF Example
Opensnoop is a tool that displays files being opened by any process. It works by attaching eBPF programs to the `open()` and `openat()` system calls, collecting data, and presenting it to the user.

[Back to TOC](#table-of-contents)

---

## III. The Rise of Cloud-Native and eBPF

### One Kernel Rules Them All
Every machine (or virtual machine) has just one kernel. All containers on that machine share this kernel. Tapping into the kernel using eBPF allows insights into every piece of application code running on that machine.

[Back to TOC](#table-of-contents)

### Comparing eBPF to the Sidecar Approach
Before eBPF, Kubernetes relied on the sidecar model, which added an instrumentation container in the same pod as the application. This model has drawbacks, such as resource consumption and incomplete coverage. In contrast, eBPF monitors all processes in the kernel, even rogue ones.

[Back to TOC](#table-of-contents)

### Understanding eBPF and Process Isolation
Though eBPF can monitor everything on a machine, it's safe due to two factors:

- **Kernel's Role:** The kernel uses cgroups and namespaces to isolate processes.
- **eBPF's Verifier:** eBPF programs go through strict checks to ensure they can’t misbehave.

[Back to TOC](#table-of-contents)

---

## IV. eBPF for Enhanced Cloud-Native Tools

### eBPF in Networking
eBPF has transformed networking. For instance:

- **Facebook's Katran**: eBPF powers a scalable layer 4 load balancer.
- **Cloudflare’s Unimog & Cilium**: These tools efficiently redirect network packets using eBPF.

[Back to TOC](#table-of-contents)

### Observability: Seeing All with eBPF
eBPF grants detailed monitoring capabilities. Tools like BCC, Pixie, Parca, and Hubble enable system performance insights without modifying applications.

[Back to TOC](#table-of-contents)

### Security: eBPF at the Forefront
eBPF enhances security in two areas:

- **Network Security:** Tools like Cloudflare and Cilium use eBPF to enforce network policies.
- **Runtime Security:** Tools like Falco and Tracee monitor application behavior for threats.

[Back to TOC](#table-of-contents)

### Wrapping Up on eBPF's Power
eBPF is revolutionizing cloud-native computing, offering tools for networking, observability, and security. For more information, visit [ebpf.io](https://ebpf.io), or explore GitHub resources and eBPF community events.

[Back to TOC](#table-of-contents)



References:
[eBPF Essentials for DevOps Professionals](https://kodekloud.com/blog/ebpf-essentials-devops/#comparing-ebpf-to-the-sidecar-approach)
