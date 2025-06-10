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

These syscalls were selected for their importance in understanding:

* Process activity (`execve`)
* File system access (`open`, `chmod`)
* Network behavior (`connect`, `accept`)

By tracing these operations at the kernel level, InfraSight provides visibility into both user and system behavior whether it's detecting a rogue shell command, a file access violation, or unexpected network traffic.

InfraSight is extensible, and support for additional syscalls (such as `unlink`, `bind`, or `setuid`) can be added in future iterations.


## 🗺️ Architecture Diagram

The diagram below illustrates the high-level architecture of the **InfraSight** platform:

![InfraSight Architecture](https://github.com/ALEYI17/InfraSight/blob/main/infrasight.drawio.png)

InfraSight follows a modular pipeline:

* **eBPF Agents** collect raw syscall events (like `execve`, `open`, `connect`, etc.) directly from the kernel using eBPF programs.
* These events are enriched at the source (e.g., resolving user names, container metadata), then streamed via **gRPC** to the central **InfraSight Server**.
* The server performs further enrichment (timestamps, formatting, latency conversion) and writes the data into **ClickHouse**, a columnar database optimized for analytical queries.
* Finally, users can analyze and visualize the collected data using tools like **Grafana**, **pytorch**, or direct SQL queries.

This architecture enables deep observability on both standalone Linux hosts and Kubernetes clusters, offering flexibility and high performance.
