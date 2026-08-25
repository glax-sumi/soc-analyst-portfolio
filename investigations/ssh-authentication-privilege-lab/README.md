# Hands-on Linux SSH Authentication & Privilege Escalation Lab

## Project Type

Hands-on Linux security lab performed on a controlled VirtualBox environment.

## Objective

Demonstrate and investigate SSH public-key authentication, Linux account
configuration, sudo privilege assignment, and root-level command execution
using a controlled local lab environment.

## Lab Environment

- Host environment: VirtualBox
- Operating system: Linux
- Hostname: `sumi-VirtualBox`
- Lab IP address: `1X.X.X.X`
- SSH service: OpenSSH
- SSH port: `22`
- Test account: `soclab`
- SSH key type: ED25518
- SSH source: `127.0.0.1` (loopback)

## Lab Procedure

### 1. Create the Lab Account

Created a dedicated Linux account named `soclab` for the controlled
experiment and verified its UID, GID, and group memberships.

### 2. Generate an SSH Key Pair

Generated an ED25519 SSH key pair:

- Private key: `soclab_test_key`
- Public key: `soclab_test_key.pub`

The private key remained on the SSH client and was not copied to the
server account.

### 3. Configure SSH Key Authentication

Created `/home/soclab/.ssh/` and configured the public key in:

`/home/soclab/.ssh/authorized_keys`

The directory and file were given restrictive permissions and appropriate
ownership.

### 4. Test SSH Authentication

Connected to the local SSH service as `soclab` using the generated private
key.

The authentication log recorded a successful public-key authentication
from `127.0.0.1`.

### 5. Configure Sudo Privileges

Added `soclab` to the `sudo` group and verified the resulting sudo rule:

`(ALL : ALL) ALL`

### 6. Verify Root-Level Execution

Compared:

`whoami` → `soclab`

with:

`sudo whoami` → `root`

This demonstrated that the `soclab` account could execute commands with
root privileges.

### 7. Analyze Authentication and Sudo Logs

Reviewed `/var/log/auth.log` to identify:

- Successful SSH authentication
- SSH session creation and closure
- Sudo activity
- Root session creation by `soclab`

### 8. Correlate the SSH Key

Calculated the fingerprint of the public key stored in
`authorized_keys` and compared it with the fingerprint reported in the
SSH authentication log.

## Evidence Collected

### Evidence 01 — SSH Public-Key Authentication

The SSH authentication log recorded successful public-key authentication
for `soclab` from the local loopback address `127.0.0.1`.

### Evidence 02 — Sudo Configuration

The sudo policy for `soclab` was:

`(ALL : ALL) ALL`

This allowed `soclab` to execute commands through `sudo` as any user,
including `root`.

### Evidence 03 — Root-Level Execution

The privilege difference was demonstrated by:

`soclab` → `whoami`

`soclab` → `sudo whoami` → `root`

This confirmed that `soclab` could execute commands with root privileges.

### Evidence 04 — Sudo Audit Event

The authentication log recorded:

`session opened for user root(uid=0) by soclab(uid=1001)`

This provided audit evidence that `soclab` initiated a sudo session with
root privileges.

### Evidence 05 — SSH Key Fingerprint Correlation

The fingerprint of the public key stored in
`/home/soclab/.ssh/authorized_keys` matched the fingerprint reported by
the successful SSH authentication event.

## Investigation Findings

The lab demonstrated that SSH public-key authentication can provide access
to a specific Linux account when the corresponding public key is present in
that account's `authorized_keys` file.

The `soclab` account initially operated as a standard user. After being
added to the `sudo` group, it received unrestricted sudo privileges according
to the observed policy `(ALL : ALL) ALL`.

The successful `sudo whoami` test returned `root`, and the authentication log
confirmed that a root sudo session was opened by `soclab`.

The SSH key fingerprint calculated from the public key stored in
`authorized_keys` matched the fingerprint reported during successful SSH
authentication. This correlated the configured public key with the observed
login event.

## SOC Analysis

From a SOC perspective, the lab demonstrates two separate security
relationships:

1. SSH public-key authentication controls access to the `soclab` account.
2. Sudo policy determines what that authenticated account can do after login.

The SSH authentication itself was legitimate within this controlled lab
because the key pair and account were intentionally created for the
experiment.

However, if the same sequence appeared unexpectedly on a production host,
the combination of a newly configured SSH key and unrestricted sudo
privileges would require immediate investigation because successful access
to the account could be followed by root-level command execution.

The investigation also demonstrates why analysts should correlate
authentication logs, account configuration, sudo policy, and process or
audit
evidence instead of relying on a single event.

## Security Lessons & Detection Opportunities

### Security Lessons

- SSH public-key authentication should be monitored for unexpected
  accounts, keys, and source addresses.
- Privileged group membership should be reviewed regularly.
- Unrestricted sudo rules should be tightly controlled because they can
  provide a direct path to root-level execution.
- Authentication, sudo, and audit logs should be correlated during
  investigations.

### Detection Opportunities

Potential detection logic for a SOC environment could include:

- Alert when a new SSH public key is added to a privileged account.
- Alert when an account is added to the `sudo` or another administrative
  group.
- Detect successful SSH authentication followed shortly by root-level
  sudo activity.
- Correlate SSH authentication source, account, key fingerprint, and
  subsequent privileged commands.
- Monitor changes to `/home/*/.ssh/authorized_keys`.

## Evidence Index

| Evidence | Description |
|---|---|
| `01-ssh-authentication.png` | Successful SSH public-key authentication for `soclab` |
| `02-sudo-configuration.png` | Unrestricted sudo policy for `soclab` |
| `03-root-execution.png` | Demonstration of root-level command execution |
| `04-sudo-audit.png` | Audit evidence showing a root sudo session initiated by `soclab` |

## Conclusion

This hands-on lab demonstrated the complete relationship between SSH
authentication, Linux account configuration, sudo privileges, and
root-level command execution.

The lab also showed how authentication logs and sudo audit records can be
used to reconstruct user activity and correlate a successful SSH login with
subsequent privileged actions.

Although the activity was intentional and benign in this controlled
environment, the same sequence involving an unexpected account or SSH key
on a production host could indicate unauthorized access, privilege
escalation, or persistence and would require further investigation.

## Skills Demonstrated

- Linux user and group management
- SSH public-key authentication
- `authorized_keys` analysis
- Linux file permissions
- Sudo privilege analysis
- Authentication log analysis
- Sudo audit log analysis
- SSH key fingerprint correlation
- Timeline construction
- SOC investigation methodology
- Security detection thinking
- Evidence-based reporting
