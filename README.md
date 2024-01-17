# disable-aslr-nerc

This repository deploys the necessary resources for allowing the personality and ptrace system calls in the rhods-notebooks namespace. This allows us to get the functionality the systems courses need from GDB. This solution consists of 3 parts:

1. Custom seccomp profile: We must first apply the custom seccomp profile (allow-personality.json) to all the nodes at the `/var/lib/kubelet/seccomp` path. We do this by using a privileged daemonset to install a custom seccomp profile on all current and future nodes in a cluster. This can be found in the `k8s-seccomp-profile-installer` directory.

2. Custom Security Context Constraint (SCC): 