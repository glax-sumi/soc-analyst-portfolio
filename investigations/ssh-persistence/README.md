# Linux SSH Persistence Investigation

## Project Type

controlled Linux SOC lab

## Objective

Investigate a suspected Linux compromise involving unexpected SSH
authentication, privilege escalation, unauthorized account creation, and
SSH key-based persistence.

## Scenario

A Linux host generated an alert after an unexpected SSH authentication.
The investigation focused on determining whether the activity represented
unauthorized access and whether persistence or privileged access had been
established.

## Investigation Timeline

| Time | Event |
|---|---|
| 01:42:17 | Successful SSH authentication for `sumi` from `10.0.2.2` |
| 01:43:02 | `sumi` executed `/usr/bin/bash` through `sudo` as `root` |
| 01:44:11 | Account `backupsvc` was created |
| Shortly after account creation | `backupsvc` was assigned sudo privileges |
| 01:46:30 | `authorized_keys` was created |
| 01:46:32 | `authorized_keys` contents were modified |

## Key Evidence

### 1. SSH Authentication

```text
This indicated that SSH public-key authentication had been configured for backupsvc.
Accepted publickey for sumi from 10.0.2.2 port 52144 ssh2
