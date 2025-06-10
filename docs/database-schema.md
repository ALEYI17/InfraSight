# 🧬 Database Schema

InfraSight stores all enriched telemetry data in **ClickHouse**, using two primary tables: `tracing_events` and `network_events`. Below you’ll find their schemas and a breakdown of the meaning of each field.

## 📁 `tracing_events` Table

<details>
<summary>Click to show table schema</summary>

```sql
CREATE TABLE IF NOT EXISTS audit.tracing_events (
  pid UInt32,
  uid UInt32,
  gid UInt32,
  ppid UInt32,
  user_pid UInt32,
  user_ppid UInt32,
  cgroup_id UInt64,
  cgroup_name String,
  comm String,
  filename String,
  monotonic_ts_enter_ns UInt64,
  monotonic_ts_exit_ns UInt64,
  return_code Int64,
  latency_ns UInt64,
  event_type String,
  node_name String,
  user String,
  latency_ms Float64, 
  wall_time_ms Int64,
  wall_time_dt DateTime64(3),
  container_id String,
  container_image String,
  container_labels_json JSON
)
ENGINE = MergeTree()
ORDER BY wall_time_ms;
```
</details>

### 🔍 Field Descriptions

| Field                  | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `pid`                 | Process ID of the event-emitting process                                    |
| `uid`                 | User ID under which the process is running                                  |
| `gid`                 | Group ID of the process                                                     |
| `ppid`                | Parent Process ID                                                           |
| `user_pid`            | Userspace PID as seen by the process itself (may differ in containers)      |
| `user_ppid`           | Userspace PPID from the process's PID namespace                             |
| `cgroup_id`           | CGroup ID the process belongs to                                            |
| `cgroup_name`         | Human-readable name or resolved path of the cgroup                          |
| `comm`                | Command name (basename of the process)                                      |
| `filename`            | Name of the file involved in the syscall (e.g., for `open`)                 |
| `monotonic_ts_enter_ns` | Timestamp (monotonic clock) when the syscall started (in nanoseconds)     |
| `monotonic_ts_exit_ns`  | Timestamp when the syscall exited (in nanoseconds)                         |
| `return_code`         | Return value of the syscall             |
| `latency_ns`          | Duration of the syscall in nanoseconds                                      |
| `event_type`          | Type of syscall event (e.g., `execve`, `open`, `chmod`, etc.)               |
| `node_name`           | Hostname of the node where the event occurred                               |
| `user`                | Username resolved from the UID                                              |
| `latency_ms`          | Latency converted to milliseconds                                           |
| `wall_time_ms`        | Wall-clock timestamp (milliseconds since epoch)                             |
| `wall_time_dt`        | ISO8601-formatted timestamp with millisecond precision                      |
| `container_id`        | Container ID, if the process is running in a container                      |
| `container_image`     | Name of the container image, if available                                   |
| `container_labels_json` | Labels from the container, stored as JSON                                  |


## 🌐 `network_events` Table

<details>
<summary>Click to show table schema</summary>

```sql
CREATE TABLE IF NOT EXISTS audit.network_events (
  pid UInt32,
  uid UInt32,
  gid UInt32,
  ppid UInt32,
  user_pid UInt32,
  user_ppid UInt32,
  cgroup_id UInt64,
  cgroup_name String,
  comm String,

  sa_family String,
  saddr_ipv4 String,
  daddr_ipv4 String,
  sport String,
  dport String,
  saddr_ipv6 String,
  daddr_ipv6 String,

  monotonic_ts_enter_ns UInt64,
  monotonic_ts_exit_ns UInt64,
  return_code Int64,
  latency_ns UInt64,

  event_type String,
  node_name String,
  user String,

  latency_ms Float64,
  wall_time_ms Int64,
  wall_time_dt DateTime64(3),

  container_id String,
  container_image String,
  container_labels_json JSON
)
ENGINE = MergeTree()
ORDER BY wall_time_ms;
```
</details>

### 🔍 Field Descriptions

Includes all common fields described above, plus:

| Field           | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| `sa_family`     | Socket address family (e.g., `AF_INET`, `AF_INET6`, `AF_UNIX`)              |
| `saddr_ipv4`    | Source IPv4 address (if applicable)                                         |
| `daddr_ipv4`    | Destination IPv4 address (if applicable)                                    |
| `sport`         | Source port                                                                 |
| `dport`         | Destination port                                                            |
| `saddr_ipv6`    | Source IPv6 address (if applicable)                                         |
| `daddr_ipv6`    | Destination IPv6 address (if applicable)                                    |


