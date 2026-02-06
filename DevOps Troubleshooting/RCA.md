
## DevOps Troubleshooting Task (The 502 Mystery)

The provided manifest, `broken-manifest.yaml`, deploys a simple Node.js application that becomes unstable shortly after startup.

**Findings**
The pod is crashing due to memory exhaustion.
From `kubectl describe`, the container’s Last State shows `Reason: OOMKilled` with Exit Code 137, and Kubernetes keeps restarting it which leads to `CrashLoopBackOff`.

**Discovery Process**
First, apply the `broken-manifest.yaml`. This creates the Deployment and pod in the default namespace.
```
kubectl apply broken-manifest.yaml
```
Next, get the list of running pods and identify the affected pod:
```
kubectl get pods
```
Then, investigate the pod events:
```
kubectl describe pod sycamore-api-76b89cd7f8-wzhkr
```
The output shows:
```
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Thu, 05 Feb 2026 21:21:21 +0100
      Finished:     Thu, 05 Feb 2026 21:21:22 +0100
    Ready:          False
    Restart Count:  4
    Limits:
      memory:  64Mi
    Requests:
      memory:     32Mi
    Environment:  <none>
```
The next step is to check the application/container logs:
```
kubectl logs sycamore-api-76b89cd7f8-wzhkr
```
There are no application-level logs before termination, which suggests the container is being killed abruptly rather than crashing due to an application error.

**Why it happens:**
The container runs:
```
let arr=[]; setInterval(() => { arr.push(new Array(1000000).fill('data')) }, 100)
```
This code continuously allocates memory every 100ms without releasing it. As a result, memory usage grows until it hits the 64Mi memory limit, at which point the kernel kills the process.

**Evidence:**
Last State: Terminated `Reason: OOMKilled`
Exit Code: 137
Repeated `Back-off restarting failed container` events
Very low memory limit (64Mi) vs an unbounded memory allocation loop

**Solution**
To fix this issue, the container is updated to run a simple HTTP server using the Node.js http module. A lightweight endpoint, /healthz, is exposed and used by Kubernetes for readiness and liveness probes. This allows Kubernetes to determine when the application is ready to receive traffic and whether the container is still running correctly.

The `fixed-manifest.yaml` includes a stable start command, health probes, and sane resource limits.
After applying it, the expected result is:
- The pod remains in a Running state
- The Restart Count stays at 0