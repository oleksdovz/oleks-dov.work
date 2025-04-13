# 🌍 How to Build Multi-Architecture Docker Images with Podman

With the rise of ARM devices like the Raspberry Pi and Apple's M1/M2 Macs, building container images for multiple architectures has become increasingly important. Luckily, tools like Podman make this easier than ever — **without requiring Docker or root access**.

# 🧰 Prerequisites

Before we begin, make sure you have:

- Podman 4.0+ installed (`podman --version`)
- Optional: A container registry like Docker Hub, GHCR, or Quay.io

---

# ⚙️ Option 1: Build Images for Each Architecture

You'll build a separate image for each architecture using the --arch flag. Podman uses buildah under the hood, which supports this natively.

Example: Build for linux/amd64 and linux/arm64:

```bash
podman build --arch amd64 -t odovzh/socat .
podman build --arch arm64 -t odovzh/socat .

```

If you want to specify OS and variant, you can use:
```bash
podman build --arch arm64 --os linux --variant v8 -t odovzh/socat:arm64 .
```

# 📦 Option 3: Create a Multi-Arch Manifest

```bash
version=odovzh/socat:0.0.6
podman manifest create $version

platarch=linux/amd64,linux/ppc64le,linux/arm64
podman build --platform $platarch  --manifest $version  .

podman manifest push $version

```
---
Result
![Docker Hub](https://github.com/oleksdovz/oleks-dov.work/raw/refs/heads/main/how-to/media/p1-how-to-build-mutliarch-docker-images.png)

