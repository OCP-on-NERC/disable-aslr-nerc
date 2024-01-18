# disable-aslr-nerc

This repository deploys the necessary resources for allowing the personality and ptrace system calls in the rhods-notebooks namespace. This allows us to get the functionality the systems courses need from GDB. This solution consists of 3 parts:

1. Custom seccomp profile: We must first apply the custom seccomp profile (allow-personality.json) to all the nodes at the `/var/lib/kubelet/seccomp` path. We do this by using a privileged daemonset to install a custom seccomp profile on all current and future nodes in a cluster. This can be found in the `k8s-seccomp-profile-installer` directory.

2. Custom Security Context Constraint (SCC): The custom SCC inherits from the restricted-v2 scc (default scc) and specifies the custom seccomp profile to be used. This SCC is applied to all of the service accounts in the namespace in order to allow the jupter instances to use the custom seccomp profile. 

3. Watch Service Accounts Script (watchsa): This script watches for service accounts in the given namespace and applies the custom SCC to all current and future service accounts created in the namespace. 