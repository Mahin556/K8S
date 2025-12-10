```bash
sudo unshare --net --pid --uts --mount --fork bash

ip a
hostname
hostname isolated-demo
hostname

mount | head
touch /tmp/testfile
ls /tmp/testfile
sudo unshare --mount --fork bash
mount -o bind /tmp /tmp
touch /tmp/newfile
```
```bash
# Create a cgroup directory called "demo"
sudo mkdir /sys/fs/cgroup/demo/

# Enable CPU, Memory and PID controllers for children
echo "+cpu +memory +pids" | sudo tee /sys/fs/cgroup/cgroup.subtree_contr

# Set memory limit = 50 MB for demo cgroup
echo $((50*1024*1024)) | sudo tee /sys/fs/cgroup/demo/memory.max

# Move current shell into the cgroup, then run Python script
# This Python script tries to allocate ~100 MB (will fail!)
( echo $$ | sudo tee /sys/fs/cgroup/demo/cgroup.procs ; \
python3 -c "a=['A'*1024*1024 for i in range(100)]" )

# Limit CPU usage to 10% of a single core
echo 10000 | sudo tee /sys/fs/cgroup/demo/cpu.max

# Add current shell to the cgroup
echo $$ | sudo tee /sys/fs/cgroup/demo/cgroup.procs

# Start a CPU-heavy workload in background
sha1sum /dev/zero &
```
```bash
# Create isolated PID, UTS, Network, and Mount namespaces
sudo unshare --pid --uts --net --mount --fork bash

# Inside the new namespace shell:
# Move this shell (PID 1 inside namespace) into cgroup
echo $$ | sudo tee /sys/fs/cgroup/demo/cgroup.procs

# Now this isolated shell has:
# • its own hostname
# • its own PID tree (you are PID 1)
# • no network interfaces except loopback
# • CPU + Memory limits enforced by cgroups

# Show PIDs (you will see very few)
ps -ef

# Show network (only the loopback interface)
ip a

# Change hostname (only affects namespace)
hostname new-container
#Open top in another terminal to observe ~10% CPU usage
```
