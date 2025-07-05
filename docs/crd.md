
## 📦 Custom Resource Definition (CRD)

InfraSight provides a Kubernetes-native way to manage and deploy its eBPF agent using a **Custom Resource Definition (CRD)** called `EbpfDaemonSet`. This resource allows you to declaratively configure which probes to run, where to send data, and how to control runtime behavior.

The CRD and its controller logic are defined in the [**infrasight-controller**](https://github.com/ALEYI17/infrasight-controller) repository. This controller watches for changes to `EbpfDaemonSet` resources and reconciles them into actual `DaemonSet` objects that run the configured probes on cluster nodes.

### 📄 Example CR

Here’s an example `EbpfDaemonSet` resource that enables the `connect` and `accept` probes and sends data to the InfraSight server:

[🔗 View on GitHub](https://github.com/ALEYI17/infrasight-controller/blob/main/config/samples/ebpf_v1alpha1_ebpfdaemonset.yaml)

```yaml
apiVersion: ebpf.monitoring.dev/v1alpha1
kind: EbpfDaemonSet
metadata:
  labels:
    app.kubernetes.io/name: kube-ebpf-monitor
    app.kubernetes.io/managed-by: kustomize
  name: ebpfdaemonset-sample
spec:
  image: ghcr.io/aleyi17/ebpf_loader:latest
  nodeSelector:
    kubernetes.io/os: linux 
  enableProbes:
    - connect
    - accept
  serverPort: "8080"
  serverAddress: ebpfplatform-ebpf-server
  prometheusPort: "9090" 
```

### 🧾 Field Reference

| Field           | Type                   | Description                                                                                       |
| --------------- | ---------------------- | ------------------------------------------------------------------------------------------------- |
| `image`         | `string`               | Container image to run (e.g., the eBPF loader agent).                                             |
| `nodeSelector`  | `map[string]string`    | Restrict the DaemonSet to specific nodes. Typically used to target Linux nodes.                   |
| `tolerations`   | `[]Toleration`         | Tolerations for scheduling on tainted nodes (optional).                                           |
| `resources`     | `ResourceRequirements` | CPU and memory requests/limits for the eBPF agent pods (optional).                                |
| `runPrivileged` | `bool`                 | If set to true, runs the container in privileged mode. Required for most eBPF operations.         |
| `enableProbes`  | `[]string`             | List of enabled eBPF probes (e.g., `execve`, `connect`, `accept`, etc.).                          |
| `serverAddress` | `string`               | The address of the InfraSight server to send data to (typically a Kubernetes service name or IP). |
| `serverPort`    | `string`               | The gRPC port on which the InfraSight server is listening.                                        |
| `prometheusPort` | `string`               | The HTTP port on which the agent exposes Prometheus metrics (e.g., `9090`).                       |


