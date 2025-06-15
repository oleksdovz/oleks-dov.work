# MinIO: Memory Optimization Tips for Linux

Optimizing MinIO's performance is crucial for any cloud-native application, and efficient memory utilization on Linux is a significant part of that. While MinIO is designed for high performance out-of-the-box, fine-tuning your Linux system can yield substantial benefits, especially for memory-intensive workloads.

In this blog post, we'll dive into two key Linux kernel parameters that can dramatically improve MinIO's memory behavior: `vm.vfs_cache_pressure` and `vm.swappiness`.

## Unlock Peak Performance: Tuning MinIO Memory Usage on Linux
MinIO, the high-performance, S3 compatible object storage, is built for speed and efficiency. However, even the most optimized applications can benefit from a finely tuned operating system. When running MinIO on Linux, understanding and adjusting certain kernel parameters can significantly impact how your system utilizes memory, leading to better performance and stability.

Today, we're focusing on two critical parameters: `vm.vfs_cache_pressure` and `vm.swappiness`. Let's explore what they do and how setting them to optimal values can enhance your MinIO deployment.

---
### Understanding `vm.vfs_cache_pressure`
The `vm.vfs_cache_pressure` parameter controls the kernel's tendency to reclaim memory used for directory and inode caches. These caches are vital for fast file system operations, as they store metadata about files and directories.

- High Value (Default is 100): A higher value means the kernel is more aggressive about reclaiming memory from the VFS cache. This can be problematic for applications like MinIO, which frequently access and store a large number of objects (files). If the kernel constantly flushes these caches, MinIO will have to re-read metadata from disk, leading to increased I/O and slower performance.

- Low Value (e.g., 50): By setting `vm.vfs_cache_pressur`e to a lower value (we recommend 50 for MinIO), you instruct the kernel to be less aggressive in reclaiming this valuable cache memory. This allows directory and inode information to stay in RAM longer, resulting in:

---
### Faster Object Access: Quicker lookups for frequently accessed objects.
Reduced Disk I/O: Less need to hit the disk for metadata.
Improved Overall Performance: Especially for workloads involving many small objects or frequent directory listings.

---
### Understanding `vm.swappiness`
`vm.swappiness` is a kernel parameter that controls how aggressively the system swaps out anonymous pages (memory not associated with a file, like application data) to swap space on disk.

- High Value (Default is 60): A higher swappiness value means the kernel is more inclined to move inactive memory pages from RAM to swap space. While this frees up physical RAM, swapping to disk is orders of magnitude slower than accessing RAM. For high-performance applications like MinIO, even minimal swapping can introduce significant latency and cripple performance.

- Low Value (e.g., 5): Setting vm.swappiness to a very low value (we recommend 5) tells the kernel to avoid swapping as much as possible. It will only resort to swap if absolutely necessary to prevent out-of-memory (OOM) situations. This ensures that MinIO's critical data structures and caches remain in fast RAM, providing:

- Consistent Low Latency: Predictable response times without swap-induced delays.
- Higher Throughput: Data stays in memory, enabling faster processing.
- Increased Stability: Prevents performance degradation under heavy load.

---
### How to Implement These Changes
- Modifying these parameters is straightforward. You'll edit the /etc/sysctl.conf file, which persists settings across reboots.
- Open the sysctl.conf file:
You'll need root privileges for this.

```Bash
sudo nano /etc/sysctl.conf
# Add or modify the following lines:
# Append these lines to the end of the file, or find and update them if they already exist.

vm.vfs_cache_pressure = 50
vm.swappiness = 5
```
- Save and exit the file. (In nano, press Ctrl+O, then Enter, then Ctrl+X).
- Apply the changes immediately:
To make the changes active without rebooting, run:

```Bash
sudo sysctl -p
```
- Verify the settings (optional but recommended):
You can check the current values by running:

```Bash
cat /proc/sys/vm/vfs_cache_pressure
cat /proc/sys/vm/swappiness
```
- The output should reflect the new values (50 and 5, respectively).

---
### Before and After: The Impact
While the exact performance gains depend on your specific workload and hardware, implementing these two memory tuning steps can lead to:
- Reduced Disk I/O: Less reliance on slow disk reads for metadata.
- Improved Latency: Faster response times for object operations.
- Higher Throughput: The ability to handle more concurrent requests.
- Greater Stability: A more robust MinIO instance less prone to performance bottlenecks under load.

# Conclusion
Tuning vm.vfs_cache_pressure and vm.swappiness are two simple yet powerful steps to optimize MinIO's memory usage on Linux. By keeping more essential data in RAM and minimizing slow disk swapping, you can significantly boost the performance and reliability of your MinIO object storage. Always remember to test these changes in a non-production environment first, and monitor your system's performance to validate the improvements.

`Happy tuning!`