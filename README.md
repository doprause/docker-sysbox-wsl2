# Setting up Docker with Sysbox (CE) on WSL2

Installation instructions for Docker with Sysbox runtime in WSl2.

Based on: https://nickjanetakis.com/blog/install-docker-in-wsl-2-without-docker-desktop#demo-video

## Step 1: Uninstall Docker Desktop (Windows)

- Uninstall Docker Desktop in Windows
- Remove `~/.docker` directory in WSL2
- Remove `/etc/docker` if exists in WSL2

## Step 2: Install Docker in WSL2

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

You might see the following warning:

```text
Warning: the "docker" command appears to already exist on this system.

If you already have Docker installed, this script can cause trouble, which is
why we're displaying this warning and provide the opportunity to cancel the
installation.

If you installed the current Docker package using this script and are using it
again to update Docker, you can ignore this message, but be aware that the
script resets any custom changes in the deb and rpm repo configuration
files to match the parameters passed to the script.

You may press Ctrl+C now to abort this script.
+ sleep 20

WSL DETECTED: We recommend using Docker Desktop for Windows.
Please get Docker Desktop from https://www.docker.com/products/docker-desktop/

You may press Ctrl+C now to abort this script.
```

::: tip
Ignore the warning
:::

### Add your user to the `docker` group

```sh
sudo usermod -aG docker $USER
```

Check if installed correctly:

```sh
docker --version
docker compose --version
```

Check if docker engine runs

```sh
sudo systemctl status docker
docker run hello-world
```

## Step 3: Install sysbox

Based on: https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-package.md#installing-sysbox

1. Download the latest Sysbox package from the [release](https://github.com/nestybox/sysbox/releases) page:

```sh
wget https://downloads.nestybox.com/sysbox/releases/v0.6.6/sysbox-ce_0.6.6-0.linux_amd64.deb
```

2. Verify that the checksum of the downloaded file fully matches the expected/published one. For example:

```sh
sha256sum sysbox-ce_0.6.6-0.linux_amd64.deb
87cfa5cad97dc5dc1a243d6d88be1393be75b93a517dc1580ecd8a2801c2777a  sysbox-ce_0.6.6-0.linux_amd64.deb
```

3. If Docker is running on the host, we recommend stopping and removing all Docker containers as follows (if an error is returned, it simply indicates that no existing containers were found):

```sh
docker rm $(docker ps -a -q) -f
```

This is recommended because the Sysbox installer may need to configure and restart Docker (to make Docker aware of Sysbox). For production scenarios, it's possible to avoid the Docker restart; see Installing Sysbox w/o Docker restart below for more on this.

4. Install the Sysbox package and follow the installer instructions:

```sh
sudo apt-get install jq
sudo apt-get install ./sysbox-ce_0.6.6-0.linux_amd64.deb
```

NOTE: the jq tool is used by the Sysbox installer.

5. Verify that Sysbox's Systemd units have been properly installed, and associated daemons are properly running:

```sh
sudo systemctl status sysbox
```

## Step 4: Post install actions

Create a `/etc/docker/daemon.json` file if not exists and adding `sysbox-runc` to `runtimes`:

```json
{
  "runtimes": {
    "sysbox-runc": {
      "path": "/usr/bin/sysbox-runc"
    }
  }
}
```

If you want to set sysbox as the default runtime, add the following to `/etc/docker/daemon.json`:

````diff
{
+  "default-runtime": "sysbox-runc",
  "runtimes": {
     "sysbox-runc": {
        "path": "/usr/bin/sysbox-runc"
     }
  }
}```
````

## Uninstallation

Prior to uninstalling Sysbox, make sure all Sysbox containers are removed. There is a simple shell script to do this [here](https://github.com/nestybox/sysbox/blob/master/scr/rm_all_syscont). Non-Sysbox containers can remain untouched.

```sh
sudo apt-get purge sysbox-ce -y
```

## Upgrading

To upgrade Sysbox, first uninstall Sysbox and re-install the updated version.

Finding the latest [Sysbox Community Releases](https://github.com/nestybox/sysbox/releases/tag/v0.6.6)

Note that you must stop all Sysbox containers on the host prior to uninstalling Sysbox (see previous section).

During the uninstall and re-install process, the Sysbox installer will restart Docker if needed.

## Command Summary

### Preparation (uninstall Docker Desktop residues from WSL2)

```sh
rm -rf ~/.docker
rm -rf /etc/docker
```

### Install docker in WSL2

#### Install docker

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

```sh
sudo usermod -aG docker $USER
```

#### Check if installed correctly:

```sh
docker --version
docker compose --version
```

#### Check if docker engine runs

```sh
sudo systemctl status docker
docker run hello-world
```

### Install Sysbox in WSL2

[Latest Sysbox Releases](https://github.com/nestybox/sysbox/releases)

Download package

```sh
wget https://downloads.nestybox.com/sysbox/releases/v0.6.6/sysbox-ce_0.6.6-0.linux_amd64.deb
```

Verify package

```sh
sha256sum sysbox-ce_0.6.6-0.linux_amd64.deb
87cfa5cad97dc5dc1a243d6d88be1393be75b93a517dc1580ecd8a2801c2777a  sysbox-ce_0.6.6-0.linux_amd64.deb
```

Stop Docker

```sh
docker rm $(docker ps -a -q) -f
```

Install Sysbox

```sh
sudo apt-get install jq
sudo apt-get install ./sysbox-ce_0.6.6-0.linux_amd64.deb
```

Check if Sysbox is running

```sh
sudo systemctl status sysbox
```

