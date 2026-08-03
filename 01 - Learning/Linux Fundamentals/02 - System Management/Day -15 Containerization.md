## Containers vs VMs

VM           → full separate OS, own kernel, heavy, minutes to start
Container    → shares HOST's kernel, lightweight, seconds to start

VM: Hardware → Hypervisor → Full OS → App  
Container: Hardware → Host OS → Container Engine → App

---
## Docker — Core Concepts

Dockerfile → instructions to BUILD an image  
Image → read-only template/blueprint  
Container → a RUNNING instance of an image

> Dockerfile = recipe, Image = cooked dish, Container = served portion
> Multiple containers can run from the SAME image

---

## Installing Docker
```bash
sudo apt update -y
sudo apt install docker-ce docker-ce-cli containerd.io -y
sudo usermod -aG docker $USER          # run docker without sudo
```
> Log out and back in after usermod -aG for group changes to apply

```bash
docker run hello-world          # test installation
```

---

## Writing a Dockerfile
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y apache2 openssh-server && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -m docker-user && \
    echo "docker-user:password" | chpasswd

EXPOSE 22 80

CMD service ssh start && /usr/sbin/apache2ctl -D FOREGROUND
```

### Dockerfile Instructions
| Instruction | Meaning |
|-------------|---------|
| FROM | base image to start from |
| RUN | execute command DURING build |
| COPY / ADD | copy files from host into image |
| EXPOSE | document which ports container listens on (metadata only) |
| CMD | command that runs when container STARTS |
| ENTRYPOINT | like CMD but harder to override |

> rm -rf /var/lib/apt/lists/* — cleans package cache, keeps image smaller
> EXPOSE doesn't actually open ports — just documentation, still need -p when running

---

## Building the Image
```bash
docker build -t FS_docker .
```
-t FS_docker  → tag/name for image
.             → build context, current directory with Dockerfile

---

## Running a Container
```bash
docker run -p 8022:22 -p 8080:80 -d FS_docker
```
-p 8022:22   → map HOST port 8022 to CONTAINER port 22
-p 8080:80   → map HOST port 8080 to CONTAINER port 80
-d           → detached — run in background
> Format always: host_port:container_port

```bash
ssh user@localhost -p 8022      # reaches SSH inside container
curl http://localhost:8080      # reaches Apache inside container
```

---

## Docker Management Commands
```bash
docker ps                          # list running containers
docker ps -a                       # list ALL containers (incl. stopped)
docker stop <container>            # stop
docker start <container>           # start
docker restart <container>         # restart
docker rm <container>              # delete container
docker rmi <image>                 # delete image
docker logs <container>            # view logs
docker exec -it <container> bash   # shell INSIDE running container
```
> -it = interactive + terminal — like SSHing into the container directly

---

## Containers Are Stateless (CRITICAL)
Changes made INSIDE a running container = LOST when stopped/removed

### Option 1 — Bake changes into new image
```bash
docker build -t FS_docker_v2 .
```

### Option 2 — Use Volumes (persistent storage)
```bash
docker run -v /host/path:/container/path -d FS_docker
```
-v mounts host folder into container — data persists after container removed

---

## Docker for Pentesting — File Hosting Workflow

1. Build Docker image with Apache + SSH
2. Run container, expose ports
3. scp tools INTO the container
4. Target downloads via curl/wget from container's Apache


```bash
scp exploit.sh docker-user@localhost:/var/www/html/ -P 8022
# accessible at http://your-ip:8080/exploit.sh
```
> Why Docker over plain Apache — isolation, portability, easy cleanup

---

## LXC — Linux Containers
Older, lower-level container tech that Docker is built on top of.

```bash
sudo apt install lxc -y
sudo lxc-create -n linuxcontainer -t ubuntu      # create container
```

### LXC Management
```bash
lxc-ls                          # list containers
lxc-start -n linuxcontainer     # start
lxc-stop -n linuxcontainer      # stop
lxc-restart -n linuxcontainer   # restart
lxc-attach -n linuxcontainer    # shell into container
lxc-attach -n <container> -f /path/to/share   # connect + share directory
```

---

## Docker vs LXC
| | Docker | LXC |
|---|--------|-----|
| Focus | single application | full lightweight OS environment |
| Ease of use | simple, beginner friendly | needs Linux admin knowledge |
| Portability | excellent — Docker Hub | limited — tied to host config |
| Security defaults | stronger out of box | needs manual hardening |
| Image format | standardized Dockerfile | manual rootfs setup |

---

## Namespaces — The Isolation Mechanism
Kernel feature that makes containers isolated from host and each other.

| Namespace | Isolates |
|-----------|----------|
| pid | process IDs — container can't see host's processes |
| net | network interfaces, routing, firewall rules |
| mnt | filesystem — container has own root filesystem |

> Containers share the kernel, but namespaces make each container THINK it's alone

---

## Cgroups — Resource Limiting
Controls how much CPU/memory/disk a container can use.

```bash
sudo vim /usr/share/lxc/config/linuxcontainer.conf
```

```
lxc.cgroup.cpu.shares = 512  
lxc.cgroup.memory.limit_in_bytes = 512M
```

```
cpu.shares = 512        → half default (1024) CPU priority
memory.limit_in_bytes   → hard memory cap

```bash
sudo systemctl restart lxc.service    # apply changes
```

---

## Security — Container Escape Risk
If misconfigured (root inside, privileged mode, host filesystem mounted) —
attacker INSIDE container might break OUT to host system.

```bash
docker inspect <container> | grep -i privileged    # check if running privileged
```

### Hardening Checklist
- Don't run containers as root inside
- Avoid --privileged flag unless necessary
- Limit resources via cgroups
- Don't mount sensitive host directories into containers
- Keep images updated — old images = old vulnerabilities

---

## Quick Command Reference
| Command | Purpose |
|---------|---------|
| docker build -t name . | build image from Dockerfile |
| docker run -p host:container -d image | run container, map ports, background |
| docker ps | list running containers |
| docker exec -it container bash | shell into running container |
| docker logs container | view logs |
| docker stop/start/rm container | manage lifecycle |
| lxc-create -n name -t ubuntu | create LXC container |
| lxc-attach -n name | shell into LXC container |

---

## TODO — Hands-On Practice Needed
- [ ] docker run hello-world
- [ ] Build a custom Dockerfile and run it
- [ ] Map ports and actually connect to a running container
- [ ] Try docker exec -it into a running container
- [ ] Install LXC and create a container
- [ ] Compare experience of Docker vs LXC hands-on

Previous:
[[Day -14 File System Management]]

Next:
[[Day -16 Networking Configuration]]

Index:
[[00 - Index Linux Fundamentals]]