## ⚙️ eBPF Programs and Attach Points

InfraSight uses a suite of eBPF programs to trace specific kernel-level events. Each program is attached to carefully chosen **tracepoints** or **kprobes** to monitor relevant syscall or kernel function activity.

### 🔍 Overview

| Program     | Attach Points                                         | Attach Type        | Description                                                                       |
| ----------- | ----------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------- |
| **execve**  | `sys_enter_execve`, `sys_exit_execve`                 | Tracepoint         | Traces process execution events, including command-line arguments and exit status |
| **open**    | `sys_enter_openat`, `sys_exit_openat`                 | Tracepoint         | Captures file open attempts, including accessed filename                          |
| **chmod**   | `sys_enter_fchmodat`, `sys_exit_fchmodat`             | Tracepoint         | Monitors changes to file permissions                                              |
| **accept**  | `inet_csk_accept` (entry and return)                  | kprobe / kretprobe | Captures accepted network connections (i.e., incoming TCP connections)            |
| **connect** | `tcp_v4_connect`, `tcp_v6_connect` (entry and return) | kprobe / kretprobe | Monitors outbound TCP connection attempts for both IPv4 and IPv6                  |
| **ptrace**  | `sys_enter_ptrace`, `sys_exit_ptrace`                 | Tracepoint         | Observes process tracing actions like `attach`, `peek`, `poke`, and `continue`; useful for detecting debuggers, tampering, or reverse engineering |
| **mmap**    | `sys_enter_mmap`, `sys_exit_mmap`                     | Tracepoint         | Tracks memory mapping requests, including suspicious RWX regions used in shellcode or injection attacks                                           |
| **mount**   | `sys_enter_mount`, `sys_exit_mount`                   | Tracepoint         | Monitors mount system calls useful for detecting container mount propagation, overlay mounts, or filesystem tampering |
| **umount** | `sys_enter_umount`, `sys_exit_umount` | Tracepoint | Tracks unmount operations, which may indicate container teardown, cleanup, or attempts to hide malicious filesystems |
| **Resource Tracer**   | - `kprobe/finish_task_switch`  <br> - `tracepoint/exceptions/page_fault_user`  <br> - `tracepoint/exceptions/page_fault_kernel` <br> - `tracepoint/syscall/sys_enter_mmap` <br> - `tracepoint/syscalls/sys_enter_munmap` <br> - `tracepoint/syscalls/sys_exit_munmap` <br> - `tracepoint/syscalls/sys_exit_brk` <br> - `tracepoint/syscalls/sys_exit_read` <br> - `tracepoint/syscalls/sys_exit_write` <br> - `tracepoint/sched/sched_process_exit` | Kprobe + Tracepoints | Monitors low-level resource usage and memory management (context switches, page faults, mmap/munmap, brk, read/write, and process exit). Useful for detecting anomalous resource consumption or crashes. |
| **Syscall Freq Tracer** | `tracepoint/raw_syscalls/sys_enter`, `tracepoint/sched/sched_process_exit` | Tracepoint         | Counts syscall invocations and aggregates frequency metrics per process until exit. Useful for anomaly detection based on unusual syscall usage patterns. |






### 🧩 Attach Types Explained

* **Tracepoint**: Static instrumentation points in the kernel. Safer and more stable across kernel versions. Used for syscalls like `execve`, `open`, `chmod`.
* **kprobe / kretprobe**: Dynamic probes on kernel functions. Used for networking-related functions like `inet_csk_accept` and `tcp_v*_connect`.

### 📁 Program Location

Each eBPF program is written in C and located under the [`bpf/`](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf) directory of the `ebpf_loader` repository. Here are the links to each specific tracer:

* **[execve\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/execve_tracer)** – Monitors process execution
* **[open\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/open_tracer)** – Tracks file open activity
* **[chmod\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/chmod_tracer)** – Observes permission changes
* **[accept\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/accept_tracer)** – Hooks accepted TCP connections
* **[connect\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/connect_tracer)** – Tracks outbound TCP connection attempts
* **[ptrace\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/ptrace_tracer)** – Detects process tracing and debugging behavior such as `ptrace` attach, memory read/write, and syscall control
* **[mmap\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/mmap_tracer)** – Observes memory allocation via `mmap` useful for detecting RWX mappings, shellcode injection, or memory-based exploits
* **[mount\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/mount_tracer)** – Monitors filesystem mount operations useful for observing overlay mounts or suspicious remounts
* **[umount\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/umount_tracer)** – Monitors filesystem unmount operations, useful for detecting container shutdowns, cleanup routines, or attempts to hide activity by unmounting evidence
* **[resource\_tracer](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/resource_tracer)** – Provides visibility into low-level resource and memory usage. It hooks context switches, page faults, memory mappings/unmappings, `brk`, read/write syscalls, and process exits. This tracer is useful for detecting anomalous resource consumption, memory pressure, or potential exploitation attempts.
* **[syscall_freq](https://github.com/ALEYI17/ebpf_loader/tree/main/bpf/syscall_freq)** – Tracks the frequency of syscalls per process by hooking into raw syscall entries and process exits. The collected counts are useful for anomaly detection and behavioral baselining.









These programs are compiled using **[`bpf2go`](https://pkg.go.dev/github.com/cilium/ebpf/cmd/bpf2go)**, a tool from the Cilium/ebpf project, which generates Go bindings for eBPF C code. The resulting artifacts are then loaded dynamically by the `ebpf_loader` agent at runtime.
