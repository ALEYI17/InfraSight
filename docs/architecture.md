## 🗺️ Architecture Diagram

The diagram below illustrates the high-level architecture of the **InfraSight** platform:

![InfraSight Architecture](./images/infrasight.png)

**InfraSight** is designed to provide detailed monitoring and auditing of Linux systems (both standalone and within Kubernetes clusters) by collecting low-level system events using **eBPF**. It operates through a modular pipeline that integrates multiple components for high-performance data collection, enrichment, and storage:

### **1. eBPF Agents**

At the core of InfraSight are **eBPF Agents**, which are lightweight, kernel-level agents running on each monitored node. These agents use **eBPF programs** attached to various **tracepoints** and **kprobes** to capture critical system call events such as:

* **`execve`**: Monitoring process execution (command launches).
* **`open`**: Tracking file access and modification.
* **`chmod`**: Observing file permission changes.
* **`connect`** and **`accept`**: Monitoring network connection and acceptance events.

These eBPF programs operate directly within the Linux kernel, providing efficient and real-time event tracing with minimal overhead. The **eBPF Agents** collect raw data, including process information, network events, and file system interactions.

### **2. Data Enrichment at the Source**

Once raw data is collected by the eBPF agents, it is enriched on the source node. This initial enrichment may involve:

* Resolving **user IDs (UID)** to human-readable **usernames**.
* Extracting container metadata using cgroup information to understand which container the process belongs to (when applicable).
* Converting **latency** from nanoseconds to more understandable units such as milliseconds or seconds.

This pre-enriched data is then streamed to the central **InfraSight Server** via a **gRPC pipeline**.

### **3. InfraSight Server**

The **InfraSight Server** serves as the central processing unit, receiving the raw events from the eBPF Agents. Upon receiving the data, the server performs additional enrichment, such as:

* Converting **latency** from nanoseconds to more understandable units such as milliseconds or seconds.
* Further converting event data (e.g., converting network protocol identifiers, if necessary).
* Formatting timestamps to **ISO8601** for easier readability.

Once enriched, the data is stored in a high-performance, scalable database.

### **4. ClickHouse Database**

InfraSight uses **ClickHouse**, a columnar database optimized for fast analytical queries, as the backend to store and manage the collected events. ClickHouse allows efficient querying even with large volumes of data, which is essential for event-driven telemetry systems.

Data stored in ClickHouse includes system call events, enriched metadata, and timestamped information that can be queried and analyzed in real-time.

### **5. Data Analysis and Visualization**

The final step of the pipeline involves visualizing and analyzing the collected events. InfraSight supports integration with **Grafana** (or similar tools) to build customizable dashboards. Users can query and analyze the data stored in ClickHouse to extract valuable insights, such as:

* **Security audit logs**: Understanding access to sensitive files or network connections.
* **Performance monitoring**: Tracking resource usage by processes, network, and file system.
* **Anomaly detection**: Identifying unusual patterns of behavior based on syscall events.

### **6. Flexibility in Deployment**

This architecture is designed to work both in standalone Linux environments and in Kubernetes clusters. The modularity allows easy deployment:

* **On Kubernetes**: The controller (`infrasight-controller`) can be used to deploy eBPF agents (as DaemonSets), manage their configuration, and collect telemetry from all nodes in the cluster.
* **On standalone Linux hosts**: The eBPF agents can run on any Linux-based system, providing valuable insights in non-Kubernetes environments.
