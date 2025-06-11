
## 🚀 Getting Started

**InfraSight** is designed as a modular and flexible observability platform. Each component operates independently and can be deployed on its own or as part of the full stack. You can run InfraSight in two main environments:

### Standalone Linux Environments

You can deploy InfraSight components directly on a Linux host using:

* A **prebuilt executable binary** (ideal for simple or resource-limited setups)
* A **Docker container**, which encapsulates dependencies and makes deployment easier across systems

This mode is well-suited for single-node monitoring, embedded systems, or edge devices.

### Kubernetes Clusters

InfraSight can also be deployed in a Kubernetes cluster using Helm charts and custom controllers. This mode enables scalable, multi-node observability and is ideal for modern infrastructure.

## 📦 Component Overview

Each core module of InfraSight lives in its own GitHub repository and includes setup and configuration details in its respective `README.md`. Follow the links below to get started with each part:

| Component                                                                   | Description                                                                              |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| [`ebpf_loader`](https://github.com/ALEYI17/ebpf_loader)                     | Lightweight agent that runs eBPF programs and sends enriched telemetry via gRPC          |
| [`ebpf_server`](https://github.com/ALEYI17/ebpf_server)                     | Central gRPC server that enriches and stores eBPF data into ClickHouse                   |
| [`infrasight-controller`](https://github.com/ALEYI17/infrasight-controller) | Kubernetes operator that automates the lifecycle of eBPF agents                          |
| [`ebpf_deploy`](https://github.com/ALEYI17/ebpf_deploy)                     | Helm charts to deploy the complete stack, including ClickHouse and InfraSight components |


Each repository provides:

* Clear **installation instructions**
* Example **configuration files**
* Guidance on **running in different environments**

Whether you want to monitor a single host or your entire cluster, you can start with just one module or deploy the entire InfraSight stack.

>  **Tip**: Start by reviewing the README in each repository to see how the pieces fit together in your environment.

