# Docker Basics — Section 1

## Overview

Install Docker Desktop on Windows using the WSL 2 backend and verify the installation by running a first container.

This lab also records my first application image build and attempts to run it.

## Contents

- [Setup](#setup)
- [Commands and Verification](#commands-and-verification)
- [Build and Run the Application Image](#build-and-run-the-application-image)
- [Troubleshooting](#troubleshooting)
- [What I Learned](#what-i-learned)

## Setup

### 1. Check WSL 2

Open PowerShell and check the WSL status:

```powershell
wsl --status
```

If WSL isn't installed, open PowerShell as Administrator and run:

```powershell
wsl --install
```

Restart your computer if prompted, and complete any Ubuntu setup prompts.

### 2. Download Docker Desktop

Download and run the Windows installer from the official [Docker Desktop for Windows page](https://docs.docker.com/desktop/setup/install/windows-install/).

### 3. Install Docker Desktop

During installation:

1. Select **Use WSL 2 instead of Hyper-V** if offered.
2. Complete the installation.
3. Restart your computer if requested.

You generally do not need to manually enable Hyper-V and Containers when using the WSL 2 backend.

### 4. Start Docker Desktop

1. Open Docker Desktop from the Start menu.
2. Accept the terms if prompted.
3. Wait until Docker Desktop indicates the engine is running.

You should also see the Docker whale icon in your system tray.

## Commands and Verification

Open the PowerShell terminal in VS Code.

Check the Docker CLI version:

```powershell
docker --version
```

Display information about the Docker engine:

```powershell
docker info
```

Finally, test your first container:

```powershell
docker run hello-world
```

If you see `Hello from Docker!`, your installation is working.

## Build and Run the Application Image

Run these commands in **Ubuntu / Bash** from `docker/01-docker-basics`, where the project's Dockerfile is located.

### 1. Build the image

```bash
docker build .
```

The `.` selects the current directory as the build context. Docker used the Dockerfile to build from `node:14`, set `/app` as the working directory, copy `package.json`, run `npm install`, and copy the remaining application files.

My build completed successfully. The output included:

```text
[+] Building 3.8s (10/10) FINISHED
```

The export steps finished with this reference:

```text
moby-dangling@sha256:0980bdf75817bf46c1f1884325cba1bcb7cd3149acb2b659e776a48703ccf841
```

I built without a `-t` option, so I did not assign the image a name and tag.

### 2. List local images

```bash
docker image ls
```

My output listed `hello-world:latest` with ID `11a3eb1597e8` and `node:latest` with ID `b46a10d964ad`. These were existing images, not the application image from the build above. The default listing did not show that untagged build in this session.

### 3. Record the initial run attempts

I first tried the ID listed for `node:latest`:

```bash
docker run -p 3000:3000 b46a10d964ad
```

The command returned to the prompt without application output. It selected the Node base image rather than my application image.

I also tried the ID for `hello-world:latest`:

```bash
docker run -p 3000:3000 11a3eb1597e8
```

This printed `Hello from Docker!`. It verified that the hello-world container ran, but did not verify my Node.js application.

### 4. Attempt to run the application image

I then used the digest from the build output:

```bash
docker run -p 3000:3000 0980bdf75817bf46c1f1884325cba1bcb7cd3149acb2b659e776a48703ccf841
```

The `-p 3000:3000` option maps port 3000 on my computer to port 3000 inside the container. The application still needs to start and listen on that container port.

**Verification status:** The image build succeeded. After running the application container, I opened `http://localhost:3000` in my browser and confirmed that the page displayed **Hi there!** This verified that the application was running and accessible through the published port.

## Troubleshooting

The following issues occurred while using Ubuntu in WSL 2. Run the commands in this section in **Ubuntu / Bash**, one line at a time.

### Docker command unavailable in Ubuntu

Docker worked from PowerShell, but Ubuntu displayed:

```text
The command 'docker' could not be found in this WSL 2 distro.
```

To enable access from Ubuntu:

1. Open Docker Desktop on Windows.
2. Go to **Settings → Resources → WSL Integration**.
3. Enable integration for **Ubuntu**.
4. Select **Apply** or **Apply & Restart**.
5. Open a new Ubuntu terminal in VS Code.

Check that the Docker CLI is available and can reach the engine:

```bash
docker --version
docker info
```

See the official [Docker Desktop WSL 2 documentation](https://docs.docker.com/desktop/features/wsl/) for more about integration.

### Permission denied when accessing the Docker socket

After enabling integration, the CLI was available, but `docker info` reported:

```text
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

Check the current user, active groups, socket permissions, and configured Docker group membership:

```bash
whoami
id
ls -l /var/run/docker.sock
getent group docker
```

The socket was owned by `root:docker` with permissions `srw-rw----`, allowing access to its owner and group. `getent group docker` listed my user, but `id` did not include the Docker group. My group membership was configured, but my current shell had not picked it up.

Because my user was already listed in the Docker group, I started a shell with that group active:

```bash
newgrp docker
```

Then I checked the active groups and engine connection again:

```bash
id
docker info
```

The Docker group appeared in `id`, and `docker info` returned a populated **Server** section without the permission error. No additional group membership or socket permission changes were needed.

### Verify container execution after the fix

I ran the following command from Ubuntu:

```bash
docker run hello-world
```

Docker downloaded the image and printed:

```text
Hello from Docker!
```

The initial message that the image could not be found locally was expected: Docker downloaded it from Docker Hub before creating and running the container. This confirmed image downloads and container execution from Ubuntu were working.

## What I Learned

- Docker working from PowerShell does not confirm that access from an Ubuntu terminal is configured.
- WSL integration lets my Ubuntu distribution use the Docker Desktop engine.
- `getent group docker` shows configured group membership, while `id` shows the groups active in my current shell.
- `newgrp docker` starts a shell with the Docker group active when membership is already configured.
- `docker --version` checks the CLI version; a successful `docker info` also confirms access to the engine.
- Building an image successfully does not prove the application starts successfully.
- The image passed to `docker run` determines which program runs. An existing Node or hello-world image is different from my application image.
- Publishing a port with `-p` does not make an application start listening on that port.
