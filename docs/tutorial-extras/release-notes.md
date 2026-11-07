---
id: release-notes
title: Release Notes
sidebar_label: Release Notes
description: New features, improvements, fixes, and breaking changes in each Driftwatch CLI release.
---

# Driftwatch Release Notes

This page lists changes in each Driftwatch CLI release, newest first.
To upgrade, see the [Installation Guide](/tutorial-basics/installation-guide.md#upgrade-driftwatch).

:::note
Driftwatch follows [semantic versioning](https://semver.org). Minor versions
(2.x) add features. Patch versions (2.4.x) contain only bug fixes and security
updates. Breaking changes appear only in major versions, except where a
deprecation notice says otherwise.
:::

---

## 2.4.1

**Released:** 28 October 2026
**Type:** Patch release

### Fixed

- Fixed a crash when scanning Terraform state files larger than 50 MB.
- Fixed `driftwatch login` failing with `Error 401` when the API key
  contained trailing whitespace.
- Fixed incorrect "missing" status for resources created with `for_each`
  and keys that contain special characters.
- Fixed `driftwatch scan --output json` omitting the `scan_duration_seconds`
  field.

### Security

- Updated the bundled TLS library to address
  [CVE-2026-0000](https://example.com/security/cve-2026-0000) (example).
  Upgrade is recommended for all users.

---

## 2.4.0

**Released:** 14 October 2026
**Type:** Minor release

### Highlights

- **New `doctor` command** checks your installation, Terraform version, and
  network access in one step.
- **Targeted scans** let you scan specific resource types instead of your
  whole account.
- **Ignore rules** let you mark expected drift so it stops appearing in
  reports.

### New

- Added `driftwatch doctor` to diagnose installation and connectivity
  problems. See [`doctor`](/tutorial-basics/cli-command-reference.md#driftwatch-doctor).
- Added the `--target` option to `driftwatch scan`. Use it to limit a scan to
  one or more resource types, for example `--target aws_s3_bucket`.
- Added `driftwatch ignore` to create and manage ignore rules. Rules are
  stored in `.driftwatch/ignore.yaml` in your project.
- Added `--output sarif` for scan results, so you can upload reports to code
  scanning tools.
- Added support for Terraform 1.7.

### Improved

- Scans are up to 40% faster on accounts with more than 500 resources,
  because resources are now queried in parallel.
- Error messages now include a link to the matching troubleshooting entry.
- `driftwatch scan` shows a progress indicator when a scan runs longer than
  10 seconds.

### Fixed

- Fixed `driftwatch connect aws` ignoring the `--profile` option when
  `AWS_PROFILE` was also set.
- Fixed drift reports showing `null → null` for attributes that were not
  actually changed.
- Fixed Docker image failing to start on arm64 hosts.

### Deprecated

- The `--format` option on `driftwatch scan` is deprecated and will be
  removed in 3.0. Use `--output` instead. Both options work in 2.x.

### Breaking changes

:::warning
**Minimum Terraform version is now 1.5.** Driftwatch 2.4.0 no longer supports
Terraform 1.3 and 1.4. If you use an older version, stay on Driftwatch 2.3.x
or upgrade Terraform first.
:::

### Upgrade notes

1. Run `terraform version` and confirm you are on 1.5 or later.
2. Upgrade the CLI. Your configuration file and scan history are preserved.
3. Run `driftwatch doctor` to confirm everything works.
4. If your scripts use `--format`, replace it with `--output`.

---

## 2.3.2

**Released:** 2 September 2026
**Type:** Patch release

### Fixed

- Fixed scan reports listing the same drifted resource twice when it appeared
  in multiple Terraform modules.
- Fixed `driftwatch logout` not removing the API key from the system
  keychain on macOS.

---

## 2.3.0

**Released:** 12 August 2026
**Type:** Minor release

### New

- Added `driftwatch schedule` to run scans automatically on a cron schedule.
- Added Slack notifications with `driftwatch notify slack`.
- Added support for AWS SSO profiles.

### Improved

- Reports now group drifted resources by Terraform module.

---

## Known issues

| Issue | Affects | Workaround |
|---|---|---|
| Scans of accounts with more than 5,000 resources can exceed 10 minutes | 2.4.0 and later | Use `--target` to scan resource types separately |
| `driftwatch doctor` reports a false network warning behind some proxies | 2.4.0 and later | Set `HTTPS_PROXY` as described in the [Installation Guide](/tutorial-basics/installation-guide.md#use-a-proxy) |

## Support policy

We provide bug fixes and security updates for the **latest two minor
versions**. Older versions receive security fixes only, for 6 months after the
next minor release.