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
  resolved_domain Nullable(String),
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
| `resolved_domain` | Reverse-resolved domain of the destination IP (if public and resolvable); `NULL` if not resolvable or internal/private IP |

## 🧩 `ptrace_events` Table

<details>
<summary>Click to show table schema</summary>

```sql
CREATE TABLE IF NOT EXISTS audit.ptrace_events (
  pid UInt32,
  uid UInt32,
  gid UInt32,
  ppid UInt32,
  user_pid UInt32,
  user_ppid UInt32,
  cgroup_id UInt64,
  cgroup_name String,
  comm String,

  request Int64,
  target_pid Int64,
  addr UInt64,
  data UInt64,
  request_name String,
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

| Field          | Description                                                                                |
| -------------- | ------------------------------------------------------------------------------------------ |
| `request`      | Raw numeric value passed to `ptrace` (e.g., `0`, `16`, `24`)                               |
| `target_pid`   | The PID of the target process being traced                                                 |
| `addr`         | Memory address or register offset used in the `ptrace` call (depends on the `request`)     |
| `data`         | Auxiliary data or pointer used by the syscall (e.g., value to write, pointer to structure) |
| `request_name` | Human-readable name for the `request` (e.g., `PTRACE_ATTACH`, `PTRACE_SYSCALL`)            |

## 🧠 `mmap_events` Table

<details>
<summary>Click to show table schema</summary>

```sql
CREATE TABLE IF NOT EXISTS audit.mmap_events (
  pid UInt32,
  uid UInt32,
  gid UInt32,
  ppid UInt32,
  user_pid UInt32,
  user_ppid UInt32,
  cgroup_id UInt64,
  cgroup_name String,
  comm String,
  
  addr UInt64,
  len UInt64,
  prot UInt64,
  flags UInt64,
  fd UInt64,
  off UInt64,

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

| Field   | Description                                                        |
| ------- | ------------------------------------------------------------------ |
| `addr`  | Starting address of the mapped memory region                       |
| `len`   | Length of the memory mapping in bytes                              |
| `prot`  | Protection flags (e.g., `PROT_READ`, `PROT_WRITE`, `PROT_EXEC`)    |
| `flags` | Mapping flags (e.g., `MAP_PRIVATE`, `MAP_ANONYMOUS`, `MAP_SHARED`) |
| `fd`    | File descriptor, or `-1` if the mapping is anonymous               |
| `off`   | Offset into the file from which mapping starts                     |

## 🗂️ `mount_events` Table

<details>
<summary>Click to show table schema</summary>

```sql
CREATE TABLE IF NOT EXISTS audit.mount_events (
  pid UInt32,
  uid UInt32,
  gid UInt32,
  ppid UInt32,
  user_pid UInt32,
  user_ppid UInt32,
  cgroup_id UInt64,
  cgroup_name String,
  comm String,
  
  dev_name String,
  dir_name String,
  type String,
  flags UInt64,

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

| Field      | Description                                                            |
| ---------- | ---------------------------------------------------------------------- |
| `dev_name` | Source device or pseudo-device (e.g., `proc`, `overlay`, `tmpfs`)      |
| `dir_name` | Target directory where the filesystem is to be mounted                 |
| `type`     | Filesystem type (e.g., `ext4`, `overlay`, `nfs`)                       |
| `flags`    | Mount flags (e.g., `MS_RDONLY`, `MS_NOSUID`, `MS_BIND`) as raw bitmask |


## 🗂️ `resource_events` Table

<details>
<summary>Click to show table schema</summary>

```sql
CREATE TABLE IF NOT EXISTS audit.resource_events (
  pid UInt32,
  comm String,

  uid UInt32,
  gid UInt32,
  ppid UInt32,
  user_pid UInt32,
  user_ppid UInt32,
  cgroup_id UInt64,
  cgroup_name String,
  user String,

  cpu_ns UInt64,
  user_faults UInt64,
  kernel_faults UInt64,
  vm_mmap_bytes UInt64,
  vm_munmap_bytes UInt64,
  vm_brk_grow_bytes UInt64,
  vm_brk_shrink_bytes UInt64,
  bytes_written UInt64,
  bytes_read UInt64,
  isActive UInt32,

  wall_time_dt DateTime64(3),
  wall_time_ms Int64,
  
  container_id String,
  container_image String,
  container_labels_json JSON
  
) ENGINE = MergeTree()
ORDER BY wall_time_ms;
```

</details>

### 🔍 Field Descriptions

Includes all common fields described above, plus:

| Field                 | Description                                                              |
| --------------------- | ------------------------------------------------------------------------ |
| `cpu_ns`              | Total CPU time consumed by the task in nanoseconds (from context switch) |
| `user_faults`         | Number of page faults occurring in user space                            |
| `kernel_faults`       | Number of page faults occurring in kernel space                          |
| `vm_mmap_bytes`       | Total bytes allocated via `mmap`                                         |
| `vm_munmap_bytes`     | Total bytes released via `munmap`                                        |
| `vm_brk_grow_bytes`   | Bytes of heap memory grown via `brk`                                     |
| `vm_brk_shrink_bytes` | Bytes of heap memory released via `brk`                                  |
| `bytes_written`       | Total bytes written by the process (from syscalls like `write`)          |
| `bytes_read`          | Total bytes read by the process (from syscalls like `read`)              |
| `isActive`            | Indicates if the process is still active (`1`) or has exited (`0`)       |
