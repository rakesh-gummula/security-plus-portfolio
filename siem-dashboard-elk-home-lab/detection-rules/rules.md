# ELK SIEM Detection Rules

These rules are designed for a Linux-focused home-lab SIEM in Kibana Elastic Security.

They use rule types supported by Elastic Security, including custom query rules and EQL event-correlation rules. Kibana saved-object exports are NDJSON, so this repo keeps the rules in readable JSON/Markdown form for copy-and-paste setup and documentation.

## SSH Brute Force

- **Rule ID:** `rule-ssh-bruteforce`
- **Type:** `custom_query`
- **Severity:** `high`
- **Risk score:** `73`
- **Schedule:** every `5m` with `10m` look-back
- **Data sources:** `filebeat-*`, `logs-*`
- **Description:** Detect repeated SSH authentication failures that may indicate brute-force attempts.

**Query**
```
event.dataset : ("system.auth" or "auth.log") and message : ("Failed password" or "authentication failure" or "Invalid user")
```

**False positives**
- Users mistyping passwords
- Automated health checks using wrong credentials
- Misconfigured scripts

**Recommended response**
- Identify source IP and target user
- Check for successful login after the failures
- Block or rate-limit the source if confirmed malicious
- Hunt for lateral movement from the same IP

## Multiple Failures Then Success

- **Rule ID:** `rule-fail-then-success`
- **Type:** `eql`
- **Severity:** `high`
- **Risk score:** `80`
- **Schedule:** every `5m` with `15m` look-back
- **Data sources:** `filebeat-*`, `logs-*`
- **Description:** Detect a suspicious pattern where repeated failures are followed by a success.

**Query**
```
sequence by user.name, source.ip with maxspan=15m
  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "success"]
```

**False positives**
- Users correcting passwords after several failed attempts
- Password manager sync issues

**Recommended response**
- Validate whether the success is expected
- Check MFA status
- Review interactive session activity after the login
- Reset credentials if compromise is suspected

## Suspicious Sudo Usage

- **Rule ID:** `rule-suspicious-sudo`
- **Type:** `custom_query`
- **Severity:** `medium`
- **Risk score:** `55`
- **Schedule:** every `5m` with `10m` look-back
- **Data sources:** `filebeat-*`, `logs-*`
- **Description:** Detect privileged command execution on Linux hosts.

**Query**
```
event.dataset : ("system.auth" or "auth.log") and message : ("sudo:" or "COMMAND=") and not user.name : ("root")
```

**False positives**
- Normal admin maintenance
- Automated patching or configuration management

**Recommended response**
- Confirm whether the user is authorized
- Review the command executed
- Correlate with login origin and time of day
- Escalate if the command is unusual

## New Local User Created

- **Rule ID:** `rule-new-user-created`
- **Type:** `custom_query`
- **Severity:** `medium`
- **Risk score:** `60`
- **Schedule:** every `10m` with `15m` look-back
- **Data sources:** `filebeat-*`, `logs-*`
- **Description:** Detect creation of a new local account on Linux systems.

**Query**
```
event.dataset : ("system.auth" or "auditd") and message : ("useradd" or "adduser" or "new user")
```

**False positives**
- Admin provisioning a new account
- Lab setup activity

**Recommended response**
- Identify who created the account
- Review privilege level of the new user
- Check for immediate login activity
- Disable or remove the account if unauthorized

## Possible Log Tampering

- **Rule ID:** `rule-log-tampering`
- **Type:** `eql`
- **Severity:** `high`
- **Risk score:** `85`
- **Schedule:** every `10m` with `20m` look-back
- **Data sources:** `filebeat-*`, `logs-*`
- **Description:** Heuristic rule to catch suspicious log file changes or disappearance patterns.

**Query**
```
sequence by host.name with maxspan=20m
  [any where event.dataset == "system.syslog" and message like "*logrotate*"]
  [any where event.dataset == "system.auth" and message like "*removed*" ]
```

**False positives**
- Routine log rotation
- System maintenance

**Recommended response**
- Verify file integrity on the host
- Check whether logging services are still running
- Look for tampering with audit or auth logs
- Preserve evidence before remediation
