---
Id: cli-reference
Title: CLI Command Reference
sidebar_label: CLI Reference
Description: Complete reference for all Driftwatch CLI commands, options, environment variables, and exit codes.
---

# CLI Command Reference

This page describes every Driftwatch CLI command, with its syntax, options,
and examples.

**Applies to:** Driftwatch CLI v2.4 and later

## Command syntax

```text
driftwatch <command> [subcommand] [arguments] [options]
```
In this Reference:

- `<value>` is a required value.
- `[value]` is optional.
- `a | b` means choese one.

## Commands at a glance:

| Command | Description |
|---|---|
| [`login`](#driftwatch-login) | Authenticate with your Driftwatch account |
| [`logout`](#driftwatch-logout) | Remove Stored Credentials |
| [`connect`](#driftwatch-connect) | Connect a cloud account |
| [`scan`](#driftwatch-scan) | Scan for infrastructure drift |
| [`ignore`](#driftwatch-ignore) | Manage rules for expected drift |
| [`schedule`](#driftwatch-schedule) | Run scans automatically |
| [`notify`](#driftwatch-notify) | Configure drift alerts |
| [`doctor`](#driftwatch-doctor) | Diagnose your installation |

## Global options

These options work with every command.

| Option | Description | Default |
|---|---|---|
| `-h`, `--help` | Show help for a command | |
| `--version` | Print the CLI version and exit | |
| `--config <path>` | Use a specific configuration file | `~/.config/driftwatch/config.yaml` |
| `--debug` | Print detailed diagnostic output | `false` |
| `--no-color` | Turn off colored output | `false` |
| `-q`, `--quiet` | Print errors only | `false` |

---

## driftwatch login

Authenticate the CLI with your Driftwatch account.

### Syntax

```text
driftwatch login [options]
```

### Options

| Option | Description | Default |
|---|---|---|
| `--api-key <key>` | Provide the API key without a prompt | |
| `--stdin` | Read the API key from standard input | `false` |

### Examples

Sign in interactively:

```bash
driftwatch login
```

Sign in from a script, without exposing the key in your shell history:

```bash
echo "$DRIFTWATCH_API_KEY" | driftwatch login --stdin
```

:::tip
In CI pipelines, skip `login` and set the `DRIFTWATCH_API_KEY` environment
variable instead.
:::

---

## driftwatch logout

Remove stored credentials from your machine. This does not revoke the key.
To revoke it, go to **Settings > API Keys** in the dashboard.

### Syntax

```text
driftwatch logout
```

---

## driftwatch connect

Connect Driftwatch to a cloud account so it can read live resources.

### Syntax

```text
driftwatch connect <provider> [options]
```

### Arguments

| Argument | Description |
|---|---|
| `provider` | Cloud provider. Supported value: `aws` |

### Options

| Option | Description | Default |
|---|---|---|
| `--profile <name>` | AWS profile to use | `default` |
| `--region <region>` | AWS region to scan | Region from the profile |
| `--role-arn <arn>` | IAM role to assume for read-only access | |

### Examples

Connect with the default profile:

```bash
driftwatch connect aws
```

Connect with a named profile and a specific region:

```bash
driftwatch connect aws --profile production --region eu-west-1
```

---

## driftwatch scan

Compare live infrastructure with your Terraform definitions and report drift.
Run this command from the directory that contains your Terraform files.

### Syntax

```text
driftwatch scan [options]
```

### Options

| Option | Description | Default |
|---|---|---|
| `--target <type>` | Scan only this resource type. Repeat the option for multiple types | All types |
| `--dir <path>` | Terraform project directory | Current directory |
| `--output <format>` | Output format: `text`, `json`, or `sarif` | `text` |
| `--out-file <path>` | Write the report to a file instead of the terminal | |
| `--fail-on-drift` | Exit with code `2` if drift is found | `false` |
| `--timeout <duration>` | Maximum scan time, for example `10m` | `15m` |
| `--format <format>` | **Deprecated.** Use `--output` | |

### Examples

Scan everything in the current project:

```bash
driftwatch scan
```

Scan only S3 buckets and security groups:

```bash
driftwatch scan --target aws_s3_bucket --target aws_security_group
```

Save a JSON report and fail a CI job if drift is found:

```bash
driftwatch scan --output json --out-file report.json --fail-on-drift
```

### Output

```text
Scan complete: 42 resources checked

  ✔ 39 in sync
  ⚠ 2 drifted
  ✖ 1 missing
```

For a description of each symbol, see
[Read the report](quick-start.md#step-5-read-the-report).

---

## driftwatch ignore

Create and manage rules that hide expected drift from scan reports. Rules are
saved in `.driftwatch/ignore.yaml` in your project directory.

### Syntax

```text
driftwatch ignore add <resource> [options]
driftwatch ignore list
driftwatch ignore remove <resource>
```

### Subcommands

| Subcommand | Description |
|---|---|
| `add` | Add an ignore rule |
| `list` | Show all ignore rules |
| `remove` | Delete an ignore rule |

### Options for `add`

| Option | Description | Default |
|---|---|---|
| `--attribute <path>` | Ignore only this attribute, for example `tags.updated_at` | Whole resource |
| `--reason <text>` | Explain why the drift is expected | |
| `--expires <date>` | Remove the rule automatically on this date (`YYYY-MM-DD`) | Never |

### Examples

Ignore a tag that another tool updates automatically:

```bash
driftwatch ignore add aws_instance.web \
  --attribute tags.last_patched \
  --reason "Updated by patch management tool"
```

:::note
Always add a `--reason`. It helps your team understand the rule later.
:::

---

## driftwatch schedule

Run scans automatically on a recurring schedule.

### Syntax

```text
driftwatch schedule create [options]
driftwatch schedule list
driftwatch schedule delete <schedule-id>
```

### Options for `create`

| Option | Description | Default |
|---|---|---|
| `--cron <expression>` | Schedule in cron format, for example `"0 6 * * *"` | Required |
| `--dir <path>` | Terraform project directory | Current directory |
| `--name <name>` | Friendly name for the schedule | |

### Example

Run a scan every day at 06:00 UTC:

```bash
driftwatch schedule create --cron "0 6 * * *" --name "daily-prod-scan"
```

---

## driftwatch notify

Send drift alerts to a messaging tool.

### Syntax

```text
driftwatch notify slack --webhook-url <url> [options]
```

### Options

| Option | Description | Default |
|---|---|---|
| `--webhook-url <url>` | Slack incoming webhook URL | Required |
| `--channel <name>` | Channel to post to | Webhook default |
| `--min-severity <level>` | Alert only at or above this level: `low`, `medium`, `high` | `medium` |

### Example

```bash
driftwatch notify slack \
  --webhook-url "$SLACK_WEBHOOK_URL" \
  --min-severity high
```

---

## driftwatch doctor

Check that Driftwatch is installed correctly and can reach the services it
needs.

### Syntax

```text
driftwatch doctor [options]
```

### Options

| Option | Description | Default |
|---|---|---|
| `--verbose` | Show detailed results for each check | `false` |

### Output

```text
✔ Driftwatch CLI 2.4.1
✔ Terraform 1.7.0 found
✔ Network: api.driftwatch.example.com reachable
⚠ Not authenticated. Run "driftwatch login".
```

---

## Environment variables

Environment variables override values in the configuration file.

| Variable | Description |
|---|---|
| `DRIFTWATCH_API_KEY` | API key. Use this in CI instead of `driftwatch login` |
| `DRIFTWATCH_CONFIG` | Path to the configuration file |
| `DRIFTWATCH_CA_BUNDLE` | Path to a custom CA certificate bundle |
| `DRIFTWATCH_NO_COLOR` | Set to `1` to turn off colored output |
| `HTTPS_PROXY` | Proxy server for outgoing connections |
| `NO_PROXY` | Hosts that should bypass the proxy |

## Exit codes

Use exit codes to control CI pipelines and scripts.

| Code | Meaning |
|---|---|
| `0` | Success. No drift found, or `--fail-on-drift` was not set |
| `1` | General error, such as invalid options or a network failure |
| `2` | Drift detected (only with `--fail-on-drift`) |
| `3` | Authentication failed |
| `4` | Terraform state not found or unreadable |

## Related topics

- [Quick Start Guide](quick-start.md)
- [Installation Guide](installation.md)
- [Release Notes](release-notes.md)