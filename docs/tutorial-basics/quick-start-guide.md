---
Id: quick-start
Title: Quick Start Guide
sidebar_label: Quick Start
Description: Install Driftwatch, connect your cloud account, and run your first drift
---
# Driftwatch Quick Start Guide

**Time to complete:** about 10 mins
**Auidence:** Devops Engineers and Platform Engineers
**Applies to:** Driftwatch CLI v2.4 and later

## What is Driftwatch?

Driftwatch is a command-line tool that compares the live state of your cloud
infrastructure against your Terraform definitions and reports any differences.
These differences are called **drift**. Drift happens when someone changes a
resource manually in the cloud console, or when another tool modifies it
outside your normal deployment process.

In this guide, you will:

1. Install the Driftwatch CLI.
2. Connect it to your clound account.
3. Run your first drift scan.
4. Read the scan report.

## Before you Begin

Make Sure you have following:

| Requirement | Details |
|---|---|
| Operating System | macOS 12+, Ubuntu 20.4, or Windows 10+ (with WLS2)|
| Terraform | Version 1.5 or later, with an existing project |
| Cloud account | An AWS account with read-only access |
| Driftwatch account | [Sign up for free](https://example.com/signup) to get an API key |

:::note
Driftwatch only needs **read-only** permissions. It never changes your
infrastructure.
:::

### Step 1: Install the CLI
Choose the method of operating system.
**macOS (Homebrew)**

```bash
brew install driftwatch/tap/driftwatch
```

**Linux (script)**
```bash
curl -fsSL https://example.com/install.sh | sh
```
**Windows (WSL2)**

Open your WSL2 terminal and run the Linux command above.

Verify the installation:

```bash
driftwatch --version
```

Expected output:

```text
driftwatch version 2.4.1
```
## Step 2: Authenticate

1. Sign in to the [Driftwatch dashboard](https://example.com/dashboard).
2. Go to **Settings > API Keys** and select **Create key**.
3. Copy the key. It is shown only once.
4. Run the login command and paste your key when prompted:

   ```bash
   driftwatch login
   ```

   Expected output:

   ```text
   ✔ Authenticated as jane@example.com
   ```

:::warning
Never commit your API key to version control. For CI pipelines, store it in
the `DRIFTWATCH_API_KEY` environment variable instead.
:::

## Step 3: Connect your cloud account

Driftwatch uses your existing AWS credentials. Run:

```bash
driftwatch connect aws --profile default
```

If you use a named profile, replace `default` with your profile name.

## Step 4: Run your first scan

Go to your Terraform project directory and run:

```bash
cd ~/projects/my-infrastructure
driftwatch scan
```

Driftwatch reads your Terraform state, queries the live resources, and
compares them. A typical scan takes 30 to 90 seconds.

## Step 5: Read the report

When the scan finishes, you will see a summary like this:

```text
Scan complete: 42 resources checked

  ✔ 39 in sync
  ⚠ 2 drifted
  ✖ 1 missing

Drifted resources:
  aws_security_group.web
    ~ ingress.0.cidr_blocks: ["10.0.0.0/16"] → ["0.0.0.0/0"]

  aws_s3_bucket.logs
    ~ versioning.enabled: true → false
```

### Understanding the symbols

| Symbol | Status | Meaning |
|---|---|---|
| ✔ | In sync | Live resource matches your Terraform code |
| ⚠ | Drifted | Live resource differs from your Terraform code |
| ✖ | Missing | Resource exists in Terraform but not in the cloud |

In the example above, someone opened the `web` security group to the whole
internet (`0.0.0.0/0`) and disabled versioning on the `logs` bucket. Both
changes were made outside Terraform.

## Next steps

- [Schedule automatic scans](#) to catch drift as it happens
- [Send alerts to Slack](#) when drift is detected
- [Add Driftwatch to your CI pipeline](#) to block risky changes
- [CLI command reference](#) for all commands and flags

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| `command not found: driftwatch` | CLI is not on your PATH | Restart your terminal, or add the install directory to your PATH |
| `Error 401: invalid API key` | Key was copied incorrectly or revoked | Create a new key and run `driftwatch login` again |
| `Error: no Terraform state found` | You are not in a Terraform project folder | Run the scan from the directory that contains your `.tf` files |
| Scan takes more than 5 minutes | Large account or slow network | Limit the scan with `driftwatch scan --target aws_s3_bucket` |

Still stuck? Contact [support@example.com](mailto:support@example.com) and
include the output of `driftwatch --version` and `driftwatch scan --debug`.
