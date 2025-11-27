https://docs.docker.com/engine/daemon/remote-access/


# 🐧 WSL and Docker Desktop on Windows: Developer Guide

> This guide explains the basics of WSL2 and Docker Desktop on Windows, their internal workings, and how they integrate. It’s useful for developers working with Linux and Docker in a Windows environment.

---

## 📚 Table of Contents

* [What is WSL2?](#what-is-wsl2)
* [Installing WSL and Linux Distributions](#installing-wsl-and-linux-distributions)
* [Checking Installed WSL Distributions](#checking-installed-wsl-distributions)
* [Switching Between Distributions](#switching-between-distributions)
* [What is Docker Desktop?](#what-is-docker-desktop)
* [How Docker Desktop Uses WSL2 Internally](#how-docker-desktop-uses-wsl2-internally)
* [Docker Command Flow](#docker-command-flow)
* [Differences: `docker-desktop` vs `Ubuntu`](#differences-docker-desktop-vs-ubuntu)
* [Visual Architecture](#visual-architecture)
* [Extras and Tips](#extras-and-tips)

---

## What is WSL2?

**WSL2 (Windows Subsystem for Linux v2)** is a lightweight virtualization layer that allows you to run a full Linux environment on Windows without a dual-boot or traditional VM setup.

### ✅ Advantages:

* Run Linux apps, tools, and services natively on Windows.
* Interoperability between Windows and Linux file systems.
* Lightweight and fast — ideal for devs.

### 📌 Supported Linux distributions:

* Ubuntu
* Debian
* Kali
* openSUSE
* Arch Linux
* …and more

---

wsl --list --online

## Installing WSL and Linux Distributions

Refer to Microsoft’s official install guide:
🔗 [https://learn.microsoft.com/en-us/windows/wsl/install](https://learn.microsoft.com/en-us/windows/wsl/install)

Once installed, your system will have at least one distribution, typically **Ubuntu** by default.

---

## Checking Installed WSL Distributions

Use the following command in CMD or PowerShell:

```bash
wsl -l -v
```

Example output:

```plaintext
  NAME                   STATE           VERSION
* Ubuntu-22.04           Running         2
  docker-desktop         Running         2
```

* The `*` marks the currently active distro when using `wsl` without `-d`.
* `docker-desktop` is a special distro managed by Docker Desktop.

---

## Switching Between Distributions

To switch to a specific WSL distro:

```bash
wsl -d Ubuntu-22.04
wsl -d docker-desktop
```

To set a default distro:

```bash
wsl --set-default Ubuntu-22.04
```

WSL Global configuration file:

Doc link: https://learn.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig

C:\Users\<UserName>\.wslconfig



---

## What is Docker Desktop?

**Docker Desktop** is an application that allows developers to build and share containerized applications and microservices on Windows.

With WSL2 integration, Docker Desktop:

* Uses WSL2 distros instead of Hyper-V or VirtualBox
* Runs faster and uses less system resources

---

## How Docker Desktop Uses WSL2 Internally

Docker Desktop installs two special WSL2 distributions:

### 1. `docker-desktop`

* Hosts the Docker Engine (`dockerd`)
* Manages containers, images, networking
* **Runtime environment** for all container activity

### 2. `docker-desktop-data`

* Stores persistent Docker data (volumes, images, config)
* **Isolated data volume distro** — not for shell access

### 3. Docker CLI

* When you run `docker` in CMD, PowerShell, or WSL, it talks to `dockerd` inside `docker-desktop`
* Communication happens via a socket: `\\.\pipe\docker_engine`

---

## Docker Command Flow

Example:

```bash
docker run alpine echo "hello"
```

### Flow:

1. You run `docker` in your terminal (Windows or Ubuntu).
2. CLI forwards command to Docker Engine inside `docker-desktop`.
3. Container is created and started inside WSL2 environment.
4. Output is sent back to your terminal.

---

## Differences: `docker-desktop` vs `Ubuntu`

| Feature         | `docker-desktop`           | `Ubuntu-22.04`              |
| --------------- | -------------------------- | --------------------------- |
| Purpose         | Runs Docker Engine         | General Linux development   |
| Access          | Limited shell (debug only) | Full Linux shell            |
| Data Location   | Runtime + container data   | User-managed home and files |
| Who manages it? | Docker Desktop             | You (user)                  |

---

## Visual Architecture

You can use a diagram like this (use [draw.io](https://draw.io), [Excalidraw](https://excalidraw.com), etc.):

```plaintext
+---------------------------+
|        Windows OS         |
+-----------+---------------+
            |
      Docker CLI (docker.exe)
            |
            v
+-----------+--------------------------------------------+
|                    WSL2                                |
|  +----------------------+    +----------------------+  |
|  | docker-desktop       |    | docker-desktop-data  |  |
|  | - Docker Engine      |    | - Image/Volume store |  |
|  +----------------------+    +----------------------+  |
|  | Ubuntu-22.04         | <- your dev shell          |
|  +----------------------+                            |
+-------------------------------------------------------+
```

> Save the diagram in `/diagrams/wsl-docker-architecture.png` in your repo.

---

## Extras and Tips

* 🔍 To view OS info inside `docker-desktop`:

  ```bash
  wsl -d docker-desktop
  cat /etc/os-release
  ```

  Example output:

  ```
  PRETTY_NAME="Docker Desktop"
  ```

* 🛑 Don’t install packages or dev tools inside `docker-desktop` — it’s not persistent and managed by Docker.

* 🗂 Docker Desktop shares your Windows files via `/mnt/host/c/` inside containers.

* 🧪 Advanced users can run Docker inside Ubuntu-WSL by installing Docker CLI and pointing to the Docker socket.

---

## 📂 Recommended Repo Structure

```plaintext
your-wsl-docker-docs/
├── README.md
├── docs/
│   ├── wsl-overview.md
│   ├── docker-desktop-internals.md
│   └── diagrams/
│       └── wsl-docker-architecture.png
```

---

## 🧵 Final Thoughts

* WSL2 + Docker is a powerful combo for devs on Windows.
* Avoid modifying `docker-desktop` manually.
* Prefer using your own WSL distro (e.g., Ubuntu) for coding and scripting.
* This doc is your go-to when debugging or setting up new environments.

---

