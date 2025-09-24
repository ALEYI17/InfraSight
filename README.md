# InfraSight

## 📖 Project Overview

> **“Kernel-level observability made simple with eBPF — for Linux and Kubernetes.”**

**InfraSight** is a modular, open-source observability and auditing platform built on top of **eBPF** (Extended Berkeley Packet Filter). It is designed to extract fine-grained, low-level events from Linux systems in real time to provide deep visibility into system and application behavior.

InfraSight provides deep visibility into system and container activity, helping operators, developers, and security teams understand what is happening on their infrastructure.

At its core, InfraSight traces key system calls (such as `execve`, `open`, `connect`, and more) at the kernel level using safe and efficient eBPF programs. These probes operate directly within the Linux kernel without modifying application code or requiring sidecars.

The collected data is streamed through a gRPC pipeline, where it is enriched and then stored in a ClickHouse database for high-performance querying and analysis.

The platform is suitable for both **standalone Linux systems** and **Kubernetes clusters**. In Kubernetes environments, InfraSight includes components to simplify agent deployment and lifecycle management through custom resources and a dedicated controller.

InfraSight is composed of four main components:

* A **Kubernetes controller** to manage and deploy eBPF agents across the cluster.
* A user-space **agent** that runs eBPF programs and streams structured events.
* A **server** that receives, enriches, and stores telemetry data in a ClickHouse database.
* A **Helm chart** to deploy the system in Kubernetes environments.
* A **Machine Learning** anomaly detection (resource + syscall frequency)
* A **Rules engine** for predefined threats.

InfraSight provides the foundation for building advanced observability, auditing, and security tools with a low-overhead, event-driven architecture.

## 🔍 Supported Syscalls & Their Purpose

InfraSight currently supports tracing the following system calls using eBPF:

| Syscall   | Purpose                                                                                                                             |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `execve`  | Captures process execution events, including command-line arguments. Useful for auditing what commands are being run on the system. |
| `open`    | Monitors when files are opened. Helps track access to sensitive files or unexpected file usage.                         |
| `chmod`   | Detects permission changes on files. Useful to monitor unauthorized attempts to alter access rights.                         |
| `connect` | Tracks outbound network connections made by processes. Essential for detecting unexpected or malicious network behavior.            |
| `accept`  | Captures inbound connections to servers. Important to understand what is listening and who is connecting.                                  |
| `ptrace`  | Monitors process tracing and injection attempts. Useful for detecting debugging or code injection behavior.                                  |
| `mmap`  | Tracks memory mappings. Can reveal suspicious allocations often used in exploits.                                  |
| `mount`  | Observes filesystem mounting. Helps detect container escapes or persistence mechanisms.                                  |
| `umount`  | Observes filesystem mounting. Helps detect container escapes or persistence mechanisms.                                  |
| `resource`  | Monitors low-level resource usage and memory management (context switches, page faults, mmap/munmap, brk, read/write, and process exit). Useful for detecting anomalous resource consumption or crashes.                                  |
| `syscall frequency`  | Counts syscall invocations and aggregates frequency metrics per process until exit. Useful for anomaly detection based on unusual syscall usage patterns.                                  |


These syscalls were selected for their importance in understanding:

* Process activity (`execve`)
* File system access (`open`, `chmod`)
* Network behavior (`connect`, `accept`)
* etc

By tracing these operations at the kernel level, InfraSight provides visibility into both user and system behavior whether it's detecting a rogue shell command, a file access violation, or unexpected network traffic.

InfraSight is extensible, and support for additional syscalls (such as `unlink`, `bind`, or `setuid`) can be added in future iterations.


## 🗺️ Architecture Diagram

The diagram below illustrates the high-level architecture of the **InfraSight** platform:

![InfraSight Architecture](https://github.com/ALEYI17/InfraSight/blob/main/docs/images/infrasight.png)

InfraSight follows a modular pipeline:

* **eBPF Agents** collect raw syscall events (like `execve`, `open`, `connect`, etc.) directly from the kernel using eBPF programs.
* These events are enriched at the source (e.g., resolving user names, container metadata), then streamed via **gRPC** to the central **InfraSight Server**.
* The server performs further enrichment (timestamps, formatting, latency conversion) and writes the data into **ClickHouse**, a columnar database optimized for analytical queries.
* A Machine Learning module analyzes patterns in resource usage and syscall frequency to detect anomalies.
* A Rules engine applies predefined detection logic to generate alerts on known threats
* Finally, users can analyze and visualize the collected data using tools like **Grafana**, **pytorch**, or direct SQL queries.

This architecture enables deep observability on both standalone Linux hosts and Kubernetes clusters.

### 4. **🧩 Project Components**

| Component                                                                   | Description                                                                               |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [`ebpf_loader`](https://github.com/ALEYI17/ebpf_loader)                     | Lightweight agent that runs eBPF programs on each node and sends telemetry to the server  |
| [`ebpf_server`](https://github.com/ALEYI17/ebpf_server)                     | Receives, enriches, and stores events in ClickHouse                                       |
| [`infrasight-controller`](https://github.com/ALEYI17/infrasight-controller) | Kubernetes-native controller for deploying and managing eBPF agents                       |
| [`ebpf_deploy`](https://github.com/ALEYI17/ebpf_deploy)                     | Helm-based deployment for ClickHouse and the server in Kubernetes                         |
| [`InfraSight_ml`](https://github.com/ALEYI17/InfraSight_ml)                 | Machine learning models for anomaly detection (resource + syscall frequency)              |
| [`InfraSight_sentinel`](https://github.com/ALEYI17/InfraSight_sentinel)     | Rules engine for generating alerts based on predefined detection logic                    |
| [`ClickHouse`](https://clickhouse.com/)                                       | High-performance columnar database used for storing and querying enriched eBPF event data |


### 5. **🚀 Getting Started**

InfraSight is composed of multiple modular components that can be deployed individually or together, depending on your needs. It supports both **Kubernetes** and **non-Kubernetes (bare-metal or VM)** environments.

To get started, **follow the README files in each of the individual repositories**. Each one contains specific setup and usage instructions tailored to its component:

| Component                | Repository                                                                  |
| ------------------------ | --------------------------------------------------------------------------- |
| eBPF Agent (Loader)      | [`ebpf_loader`](https://github.com/ALEYI17/ebpf_loader)                     |
| Event Server & Ingestion | [`ebpf_server`](https://github.com/ALEYI17/ebpf_server)                     |
| Kubernetes Controller    | [`infrasight-controller`](https://github.com/ALEYI17/infrasight-controller) |
| Helm-based Deployment    | [`ebpf_deploy`](https://github.com/ALEYI17/ebpf_deploy)                     |
| ML Anomaly Detection     | [`InfraSight_ml`](https://github.com/ALEYI17/InfraSight_ml)                 |
| Rules Engine             | [`InfraSight_sentinel`](https://github.com/ALEYI17/InfraSight_sentinel)     |


### 6. **✨ Features**

* **Kernel-Level Tracing with eBPF**
  Trace key system calls like `execve`, `open`, `chmod`, `connect`, and `accept` directly from the Linux kernel.

* **Real-Time Event Streaming**
  Events are streamed over gRPC for minimal latency and efficient transport.

* **Structured Storage with ClickHouse**
  Events are stored in a high-performance, columnar database for fast querying and analysis.

* **Machine Learning Anomaly Detection**
  Detect resource usage spikes and unusual syscall frequency patterns.

* **Rules Engine for Threat Detection**
  Catch predefined malicious behaviors such as reverse shells, privilege escalation, or container escapes.

* **Works in Bare Metal or Kubernetes**
  InfraSight can run on regular Linux systems or be deployed in Kubernetes using Helm and a custom controller.

* **Kubernetes Controller with CRD Support**
  Deploy and manage tracing agents with fine-grained configuration using a custom resource.

### 8. **🔮 Future Work**

InfraSight is designed to be extensible. The following enhancements are under consideration to make the platform even more powerful and user-friendly:

* [x] **Anomaly Detection & Behavior Profiling**
  Machine learning models for syscall frequency and resource usage anomaly detection.

* [x] **Resilience & Scalability**
  Retries, graceful shutdown, batching, and optional message queue integration for scaling the server.
* [x] **Threat Detection Capabilities**
  Add rule-based detection for attack patterns such as privilege escalation, reverse shells, or unauthorized access attempts.
* [x] **Sentinel Integration**
  Static analysis and correlation engine for combining runtime events with code-level insights.

* [ ] **Standard Dashboards (Grafana / Metabase)**
  Ready-made dashboards for visualizing telemetry data.

* [ ] **Alerting System**
  Integrate with email, Slack, or webhook notifications when anomalies or threats are detected.

* [ ] **Web Interface or CLI**
  User-friendly interface to explore and interact with traced events.

> Have an idea or suggestion to improve InfraSight?
Feel free to open an issue or reach out — contributions and feedback are always welcome!

