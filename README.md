# disable-aslr-nerc

This repository deploys the necessary resources for allowing the personality and ptrace system calls in the rhods-notebooks namespace. This allows us to get the functionality the systems courses need from GDB. This solution consists of 3 parts:

1. **Custom seccomp profile**: We must first apply the custom seccomp profile (allow-personality.json) to all the nodes at the `/var/lib/kubelet/seccomp` path. We do this by using a privileged daemonset to install a custom seccomp profile on all current and future nodes in a cluster. This is found at this repository: https://github.com/IsaiahStapleton/k8s-seccomp-profile-installer

2. **Custom Security Context Constraint (SCC)**: The custom SCC inherits from the restricted-v2 scc (default scc) and specifies the custom seccomp profile to be used. This SCC is applied to all of the service accounts in the namespace in order to allow the jupyter instances to use the custom seccomp profile.

3. **Watch Service Accounts Script (watchsa)**: This script watches for service accounts in the given namespace and applies the custom SCC to all current and future service accounts created in the namespace.

## Prerequisites

Before applying this solution, you must first deploy the custom seccomp profile to all nodes using the [k8s-seccomp-profile-installer](https://github.com/IsaiahStapleton/k8s-seccomp-profile-installer).

## Installation

### 1. Apply the solution

```bash
oc apply -k disable-aslr/overlays/nerc-ocp-prod/
```

### 2. Verify the deployment

Check that the watchsa deployment is running:

```bash
oc get deployment watchsa -n rhods-notebooks
```

Check the logs to see the script watching for service accounts:

```bash
oc logs -n rhods-notebooks deployment/watchsa -f
```

## Testing

To verify that ASLR is properly disabled and GDB can function correctly, run the validation script inside a Jupyter notebook terminal:

```bash
./validateASLR.sh
```

The script runs GDB 100 times and checks if the instruction address remains constant (ASLR disabled). Expected output:

```
OK:  Looks good
```

If ASLR is not properly disabled, you will see:

```
ERROR: Failed to start a binary with a constant instruction address: <count>
```

### Manual Test

You can also manually verify by running:

```bash
gdb -ex starti -ex quit -q --batch /usr/bin/date 2>/dev/null | grep ld
```

Run this command multiple times. If ASLR is disabled, the memory address should be the same each time.
