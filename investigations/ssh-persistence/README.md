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

`Accepted publickey for sumi from 10.0.2.2 port 52144 ssh2`

This established that `sumi` successfully authenticated to the Linux host
through SSH using a public key from `10.0.2.2`.

### 2. Privilege Escalation

`sudo: sumi : TTY=pts/0 ; PWD=/home/sumi ; USER=root ; COMMAND=/usr/bin/bash`

This showed that `sumi` used `sudo` to start a Bash shell with root privileges.

### 3. New Account Creation

`useradd: new user: name=backupsvc, UID=1001, GID=1001`

A new local account named `backupsvc` was created shortly after the
privilege escalation event.

### 4. Sudo Privileges

`uid=1001(backupsvc) gid=1001(backupsvc) groups=1001(backupsvc),27(sudo)`

The `backupsvc` account belonged to the `sudo` group.

The observed sudo rule was:

`User backupsvc may run the following commands: (ALL : ALL) ALL`

This indicated that `backupsvc` was permitted to execute commands through
sudo as any user, including `root`.

### 5. SSH Persistence

The account contained:

`/home/backupsvc/.ssh/authorized_keys`

An SSH public key was present in the `authorized_keys` file.

This indicated that SSH public-key authentication had been configured
for `backupsvc`.

### 6. File Metadata

The `authorized_keys` file showed:

`Birth: 01:46:30`

`Modify: 01:46:32`

`Permissions: 0600 / -rw-------`

The file was created at 01:46:30 and its contents were last modified at
01:46:32. The permissions restricted read and write access to the file
owner.

### 7. Audit Evidence

Audit evidence showed a Bash process executing:

`echo 'ssh-ed25519 ... attacker@unknown' >> /home/backupsvc/.ssh/authorized_keys`

The associated audit information showed:

`uid=1000`

`euid=0`

`auid=1000`

`exe="/usr/bin/bash"`

This showed that the command was executed with root effective privileges
while remaining associated with the original authenticated user.

## Assessment

**Severity: Critical**

The combined evidence is consistent with unauthorized privileged activity
and establishment of SSH key-based persistence.

The investigation demonstrates a sequence involving SSH access, privilege
escalation, creation of a privileged account, and modification of SSH
authorization data.

## Evidence Limitations

The investigation does not by itself establish the identity of the person
holding the private SSH key corresponding to the public key found in
`backupsvc/.ssh/authorized_keys`.

The presence of the public key establishes that SSH key-based authentication
was configured for `backupsvc`, but additional evidence would be required
to attribute the key to a specific individual or system.

## Recommended Response

- Isolate the affected host from unnecessary network access.
- Preserve relevant authentication, audit, and system logs.
- Disable or quarantine the suspicious `backupsvc` account.
- Preserve the suspicious SSH key as evidence before removing it.
- Review sudo configuration and recently modified system files.
- Investigate outbound network connections from the host.
- Rotate potentially compromised credentials and SSH keys.
- Search other systems for the same indicators of compromise.

## Skills Demonstrated

- Linux authentication analysis
- SSH security investigation
- User and group analysis
- Sudo privilege analysis
- Linux file permission analysis
- File timestamp analysis
- Audit log interpretation
- Persistence analysis
- Attack timeline construction
- Incident severity assessment
- Evidence-based reporting

## Lessons Learned

This investigation reinforced the importance of separating what evidence
proves from what it suggests.

Authentication logs helped establish the initial access event, while sudo,
account, file, and audit evidence helped reconstruct the subsequent activity.

The investigation also demonstrated that SSH key-based persistence becomes
particularly significant when the associated account has unrestricted sudo
privileges.

