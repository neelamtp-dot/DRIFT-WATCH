---
Id: installation
Title: Installation Guide
sidebar_label: Installation
Description: Install, verify, upgrade, and uninstall the Driftwatch CLI on macOS, Linux, Windows (WSL2), and Docker.
---

# Installing Driftwatch

This guide covers all supported ways to install the Driftwatch CLI, and how
to verify, upgrade, and remove it.

**Audience:** DevOps engineers, platform engineers, and system administrators
**Applies to:** Driftwatch CLI v2.4 and later

:::tip
If you only want to try Driftwatch quickly, follow the
[Quick Start Guide](quick-start-guide.md) instead.
:::

## System requirements

| Component | Minimum | Recommended |
|---|---|---|
| Operating system | macOS 12, Ubuntu 20.04, Windows 10 (WSL2) | macOS 14, Ubuntu 22.04 |
| CPU architecture | x86_64 or arm64 | arm64 or x86_64 |
| Memory | 512 MB free | 2 GB free |
| Disk space | 100 MB | 500 MB (for scan cache) |
| Terraform | 1.5 | Latest 1.x release |
| Network | HTTPS access to `api.driftwatch.example.com` on port 443 | Same |

:::note
Native Windows (PowerShell or Command Prompt) is not supported. Use WSL2.
:::

## Choose an installation method

| Method | Best for | Auto-updates |
|---|---|---|
| [Homebrew](#install-with-homebrew) | macOS and Linux developers | Yes, with `brew upgrade` |
| [Install script](#install-with-the-script) | Quick setup on Linux servers | No |
| [Manual binary](#install-from-a-binary) | Locked-down or air-gapped environments | No |
| [Docker](#run-with-docker) | CI pipelines and containers | Pin the image tag |

## Install with Homebrew

1. Add the Driftwatch tap and install:

   ```bash
   brew install driftwatch/tap/driftwatch
   ```

2. Verify the installation:

   ```bash
   driftwatch --version
   ```

   Expected output:

   ```text
   driftwatch version 2.4.1
   ```

## Install with the script

The script detects your operating system and architecture, downloads the
correct binary, and places it in `/usr/local/bin`.

1. Run the script:

   ```bash
   curl -fsSL https://example.com/install.sh | sh
   ```

2. To install a specific version, set the `DRIFTWATCH_VERSION` variable:

   ```bash
   curl -fsSL https://example.com/install.sh | DRIFTWATCH_VERSION=2.4.1 sh
   ```

3. To install to a different directory, set `DRIFTWATCH_INSTALL_DIR`:

   ```bash
   curl -fsSL https://example.com/install.sh | DRIFTWATCH_INSTALL_DIR=$HOME/.local/bin sh
   ```

:::warning
Piping a script to a shell runs code on your machine. If your security policy
does not allow this, download the script first and review it, or use the
[manual binary](#install-from-a-binary) method.
:::

## Install from a binary

Use this method when you cannot run the install script, or when you need to
verify the download before installing.

1. Download the archive for your platform from the
   [releases page](https://example.com/releases). For example, for Linux
   x86_64:

   ```bash
   curl -LO https://example.com/releases/v2.4.1/driftwatch_2.4.1_linux_amd64.tar.gz
   curl -LO https://example.com/releases/v2.4.1/checksums.txt
   ```

2. Verify the checksum:

   ```bash
   sha256sum --check --ignore-missing checksums.txt
   ```

   Expected output:

   ```text
   driftwatch_2.4.1_linux_amd64.tar.gz: OK
   ```

   :::danger
   If the result is `FAILED`, do not install the file. Delete it and
   download it again.
   :::

3. Extract the archive and move the binary onto your PATH:

   ```bash
   tar -xzf driftwatch_2.4.1_linux_amd64.tar.gz
   sudo mv driftwatch /usr/local/bin/
   ```

4. Confirm that it runs:

   ```bash
   driftwatch --version
   ```

## Run with Docker

The official image contains the Driftwatch CLI and Terraform.

1. Pull the image. Pin a version tag for repeatable builds:

   ```bash
   docker pull driftwatch/driftwatch:2.4.1
   ```

2. Run a scan by mounting your Terraform project into the container:

   ```bash
   docker run --rm \
     -v "$(pwd)":/workspace \
     -w /workspace \
     -e DRIFTWATCH_API_KEY \
     -e AWS_ACCESS_KEY_ID \
     -e AWS_SECRET_ACCESS_KEY \
     driftwatch/driftwatch:2.4.1 scan
   ```

:::note
The `latest` tag always points to the newest release. Avoid it in production
pipelines because a new release can change behavior without warning.
:::

## Install on Windows (WSL2)

1. Install WSL2 and Ubuntu if you have not already. In PowerShell
   (as Administrator), run:

   ```powershell
   wsl --install -d Ubuntu
   ```

2. Restart your computer, then open the **Ubuntu** app.
3. Follow the [install script](#install-with-the-script) steps inside the
   Ubuntu terminal.

Store your Terraform projects inside the WSL2 file system (for example,
`~/projects`) rather than under `/mnt/c`. Scans run noticeably faster there.

## Verify your installation

Run the built-in check. It tests the CLI, your Terraform version, and your
network access:

```bash
driftwatch doctor
```

Expected output:

```text
✔ Driftwatch CLI 2.4.1
✔ Terraform 1.7.0 found
✔ Network: api.driftwatch.example.com reachable
⚠ Not authenticated. Run "driftwatch login".
```

The warning is expected on a fresh install. Continue with
[authentication](quick-start-guide.md#step-2-authenticate).

## Configure the CLI

Driftwatch reads settings from a configuration file. It is created the first
time you run `driftwatch login`.

| Operating system | Default location |
|---|---|
| macOS and Linux | `~/.config/driftwatch/config.yaml` |
| WSL2 | `~/.config/driftwatch/config.yaml` |

### Use a proxy

If your network requires an HTTP proxy, set these environment variables
before you run Driftwatch:

```bash
export HTTPS_PROXY=http://proxy.example.com:8080
export NO_PROXY=localhost,127.0.0.1
```

### Use a custom CA certificate

If your organization inspects TLS traffic, point Driftwatch to your
certificate bundle:

```bash
export DRIFTWATCH_CA_BUNDLE=/etc/ssl/certs/corporate-ca.pem
```

## Upgrade Driftwatch

| Installed with | Upgrade command |
|---|---|
| Homebrew | `brew upgrade driftwatch` |
| Install script | Run the install script again |
| Manual binary | Download the new archive and replace the old binary |
| Docker | Pull the new image tag |

After upgrading, run `driftwatch --version` to confirm the new version.

:::note
Upgrading never deletes your configuration file or scan history.
:::

## Uninstall Driftwatch

1. Remove the CLI:

   | Installed with | Command |
   |---|---|
   | Homebrew | `brew uninstall driftwatch` |
   | Script or binary | `sudo rm /usr/local/bin/driftwatch` |
   | Docker | `docker rmi driftwatch/driftwatch:2.4.1` |

2. (Optional) Remove configuration and cache files:

   ```bash
   rm -rf ~/.config/driftwatch ~/.cache/driftwatch
   ```

3. (Optional) Revoke your API key in the dashboard under
   **Settings > API Keys**.

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| `command not found: driftwatch` | Install directory is not on your PATH | Add it to your PATH, then restart your terminal |
| `Permission denied` when moving the binary | `/usr/local/bin` needs elevated rights | Use `sudo`, or install to `$HOME/.local/bin` |
| `x509: certificate signed by unknown authority` | A proxy or firewall is inspecting TLS traffic | Set `DRIFTWATCH_CA_BUNDLE` as described above |
| `driftwatch doctor` reports Terraform not found | Terraform is not installed or not on your PATH | Install Terraform 1.5 or later |
| `exec format error` | Wrong CPU architecture | Download the `arm64` or `amd64` build that matches `uname -m` |
| Docker scan cannot find cloud credentials | Credentials are not passed into the container | Pass the `AWS_*` variables with `-e`, as shown above |

Still stuck? Contact [support@example.com](mailto:support@example.com) and
include the output of `driftwatch doctor --verbose`.