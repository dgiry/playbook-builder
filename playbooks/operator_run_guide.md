# Operator Run Guide
## Playbook: Response to Active CVE Exploitation — CVE-2009-0238 / CVE-2026-32201
**Canonical playbook:** `v1-pb-cve-2009-0238-cve-2026-32201` v2.0  
**Platform:** Trend Vision One  
**Prerequisite before running:** tenant manifest filled, all `confirmed: false` items in activation checklist resolved.

---

## Notation

| Symbol | Meaning |
|--------|---------|
| **[APR-001]** | Formal approval gate — IR Lead must approve before step executes. Do not auto-approve on timeout; escalate to IR Manager. |
| **[AUTH]** | IR Lead verbal authorization — documented in incident record with timestamp. Not a formal APR gate. |
| **[EH-n]** | External handoff — Vision One cannot execute this action. Operator triggers the handoff and tracks SLA. |
| **[COND]** | Conditional step — execute only when the stated condition is true. |
| **[BLOCK]** | This step cannot start until the named prerequisite is confirmed. |

---

## Pre-Activation

Before running any step:

1. Confirm playbook trigger: Workbench or OAT alert on T1203 or T1068 at high/critical severity, or analyst-confirmed exploitation (TRG-001, TRG-002, TRG-003).
2. Open tenant manifest — confirm all `confirmed: false` items in activation checklist are resolved.
3. Set `TIME_WINDOW_START` from the Workbench incident timeline.
4. Document playbook activation in the Workbench incident record: timestamp, operator identity, trigger event.

---

## Phase 1 — Detection

### D-001 · Review Workbench for T1203 / T1068 correlated incidents
**Module:** Workbench · **Type:** manual

- Filter open incidents by technique IDs T1203 and T1068.
- Extract `AFFECTED_ENDPOINTS` from Impact Scope panel.
- Set `TIME_WINDOW_START` from incident timeline if not already set.
- **Evidence:** EV-001 (Workbench incident record — also referenced by E-003, R-003, PI-003).
- **→ Next:** D-002

---

### D-002 · Review OAT alerts for Exploitation and Privilege Escalation
**Module:** Detection Model Management · **Type:** manual  
**[COND]** Skip if `FEATURE_DETECTION_MODEL_MGMT = false` — document detection coverage gap, proceed to D-003.

- Filter OAT alerts by categories: Exploitation, Privilege Escalation — scoped to `AFFECTED_ENDPOINTS`.
- Record alert severity, technique ID, and matched endpoint per alert.
- **Evidence:** EV-002
- **→ Next:** D-003

---

### D-003 · Search for exploit child process chain
**Module:** XDR Search App · **Type:** manual · **Query:** Q-001  
**[BLOCK]** Q-001 field names validated in tenant (see tenant manifest query_field_validation).

- Substitute `AFFECTED_ENDPOINTS`, `TIME_WINDOW_START`, `TIME_WINDOW_END` into Q-001 and execute.
- Extract dropped file paths → input for C-005.
- Add confirmed malicious hashes to `MALICIOUS_HASHES`.
- **Evidence:** EV-003
- **→ Next:** D-004

---

### D-004 · Search for post-exploit outbound network connections
**Module:** XDR Search App · **Type:** manual · **Query:** Q-002  
**[BLOCK]** `INTERNAL_IP_RANGES` populated in tenant config; Q-002 fields validated.

- Substitute `AFFECTED_ENDPOINTS`, `INTERNAL_IP_RANGES`, time window into Q-002 and execute.
- Add confirmed external malicious IPs to `MALICIOUS_IPS`.
- Add malicious domains/URLs to `MALICIOUS_DOMAINS` and `SUSPICIOUS_URLS`.
- If `FEATURE_NETWORK_SENSOR = false`: coverage may be incomplete — document telemetry scope.
- **Evidence:** EV-004
- **→ Next:** T-001

---

## Phase 2 — Triage

### T-001 · Review Risk Index for affected endpoints
**Module:** Attack Surface Risk Management · **Type:** manual  
**[COND]** Skip if `FEATURE_ASRM = false` — manually prioritize based on known asset sensitivity, document ASRM gap.

- Review Risk Index per endpoint in `AFFECTED_ENDPOINTS` via Executive Dashboard.
- Produce prioritized response sequence: highest risk first.
- Flag domain controllers, database servers, and privileged account hosts — service impact warning applies before C-001.
- **Evidence:** EV-005
- **→ Next:** T-002

---

### T-002 · Search for lateral movement from compromised endpoint
**Module:** XDR Search App · **Type:** manual · **Query:** Q-003  
**[BLOCK]** T-001 completed; primary endpoint identified in `AFFECTED_ENDPOINTS`.

- Execute Q-003 for primary endpoint.
- If lateral movement confirmed: add target hostnames to `AFFECTED_ENDPOINTS` and loop back to T-001 for expanded Risk Index, then repeat T-003 for new endpoints.
- **Evidence:** EV-006
- **→ Next:** T-003

---

### T-003 · Search for LSASS process access on affected endpoints
**Module:** XDR Search App · **Type:** manual · **Query:** Q-004

- Execute Q-004 for all `AFFECTED_ENDPOINTS`.
- If `TELEMETRY_PROCESS_ACCESS` events absent from tenant: document telemetry gap — step cannot confirm or deny LSASS access.
- Build per-tenant exclusion list for known legitimate LSASS-accessing processes before classifying results.
- **Evidence:** EV-007
- **→ Next:** T-004

---

### T-004 · Submit payload to Threat Analysis Center
**Module:** Threat Analysis Center · **Type:** manual  
**[COND]** Skip if `FEATURE_SANDBOX = false` or `SANDBOX_QUOTA_REMAINING = 0` — proceed to T-005, document no behavioral analysis performed.

- Submit suspicious file or URL. Do not submit known-clean or already-classified files — conserve quota.
- Extract network IOCs → `MALICIOUS_IPS`, `MALICIOUS_DOMAINS`; extract dropped payload hashes → `MALICIOUS_HASHES`.
- **Evidence:** EV-008
- **→ Next:** T-005

---

### T-005 · Threat Intelligence IOC reputation lookup
**Module:** Threat Intelligence · **Type:** manual

- Look up every IOC in `MALICIOUS_HASHES`, `MALICIOUS_IPS`, `MALICIOUS_DOMAINS`.
- Remove IOCs that return clean or unknown verdict from variable lists.
- Finalize confirmed IOC lists — these are the input for C-002.
- **Evidence:** EV-009
- **→ Next:** C-001

---

## Phase 3 — Containment

> **Parallel track:** Trigger **[EH-002]** at C-006 as soon as `MALICIOUS_IPS` is finalized — EH-002 does not block C-001 through E-001 but must complete before R-001.

---

### C-001 · Isolate affected endpoints
**Module:** Response Management · **Type:** native · **Action:** RA-001  
**[APR-001]** IR Lead approval required before execution — no exceptions.  
**[BLOCK]** T-005 completed; `AFFECTED_ENDPOINTS` finalized; service impact assessment completed for critical hosts.

- Execute isolation for each endpoint in `AFFECTED_ENDPOINTS` in priority order from T-001.
- Confirm Vision One management channel remains active post-isolation per endpoint.
- Record isolation timestamp per endpoint.
- If isolation fails on any endpoint: retain in scope, document failure, escalate.
- **Evidence:** EV-010
- **→ Next:** C-002

---

### C-002 · Block confirmed IOCs in Suspicious Object Management
**Module:** Suspicious Object Management · **Type:** native · **Action:** RA-006  
**[BLOCK]** T-005 completed — IOC lists finalized.

- Submit all confirmed `MALICIOUS_HASHES`, `MALICIOUS_IPS`, `MALICIOUS_DOMAINS`, `SUSPICIOUS_URLS` with TTL = `SOL_TTL_DAYS`.
- Validate deployment status per sensor group after submission.
- Record TTL expiry date per entry — required for R-004.
- **Evidence:** EV-011
- **→ Next:** C-003

---

### C-003 · Terminate active malicious processes
**Module:** Response Management · **Type:** native · **Action:** RA-002  
**[AUTH]** IR Lead verbal authorization required — document in incident record with timestamp. This is NOT a formal APR gate.  
**[BLOCK]** C-001 isolation confirmed for all target endpoints.

- Confirm target process is still running via XDR Search App before executing.
- Execute termination by PID or image name.
- Run follow-up XDR Search to confirm process absence.
- If process respawns: do not re-terminate without additional investigation — active persistence not yet eradicated.
- **Evidence:** EV-012
- **→ Next:** C-004

---

### C-004 · Reset compromised account credentials
**Module:** Identity Security · **Type:** native · **Action:** RA-007  
**[APR-001]** IR Lead approval required before execution.  
**[COND]** If `FEATURE_IDENTITY_SECURITY = false` OR `IDENTITY_INTEGRATION_TYPE = none`: skip native action — **activate [EH-004] immediately**. Compromised credentials remain exploitable until reset confirmed.  
**[BLOCK]** `AFFECTED_ACCOUNTS` identified; APR-001 received; service account impact assessment completed.

- After reset: verify no re-authentication from `AFFECTED_ACCOUNTS` via XDR Search.
- For Azure AD: confirm refresh token revocation in addition to password reset.
- Notify service owner if any reset account is a service account.
- **Evidence:** EV-013
- **→ Next:** C-005

---

### C-005 · Collect file artifacts from affected endpoints
**Module:** Response Management · **Type:** native · **Action:** RA-003  
**[BLOCK]** C-001 isolation confirmed; file paths identified from D-003 results.

> **Hard prerequisite for E-001.** Do not proceed to file deletion without confirmed artifact collection.

- Verify hash of each collected file matches hash from XDR Search results.
- If any path not found: document as self-deletion candidate — do not proceed to E-001 for that path without escalation.
- **Evidence:** EV-014
- **→ Next:** C-006

---

### C-006 · Perimeter firewall block for confirmed malicious IPs  
**Type:** external · **[EH-002]**  
**[BLOCK]** C-002 completed; `MALICIOUS_IPS` finalized from T-005.

- Submit `MALICIOUS_IPS` to `FIREWALL_TEAM_CONTACT` with incident reference ID and SLA = `FIREWALL_TEAM_SLA_HOURS`.
- Track SLA — escalate to IR Manager if `FIREWALL_TEAM_SLA_HOURS` elapses without confirmation.
- **This step runs in parallel with Eradication — does not block E-001. Must complete before R-001.**
- **Evidence:** EV-015 (firewall rule confirmation from network team — required for R-001 blocking condition)
- **→ Next:** E-001 (parallel)

---

## Phase 4 — Eradication

> **Also trigger [EH-001]** at E-004 — engage patch team at the start of Containment (or earlier) to minimize isolation duration. R-001 is hard-blocked until EH-001 patch confirmation is received.

---

### E-001 · Delete confirmed malicious files from affected endpoints
**Module:** Response Management · **Type:** native · **Action:** RA-004  
**[AUTH]** IR Lead verbal authorization required — document in incident record with timestamp. This is NOT a formal APR gate.  
**[BLOCK]** C-005 artifact collection confirmed complete for all targeted paths; C-001 isolation confirmed.

> **Irreversible.** Scope deletion strictly to confirmed malicious file paths. Do not delete system files.

- Execute delete_file per confirmed malicious path per endpoint.
- Proceed immediately to E-002 after deletion.
- **Evidence:** EV-016
- **→ Next:** E-002

---

### E-002 · Verify clean state via XDR Search App
**Module:** XDR Search App · **Type:** manual · **Query:** Q-005  
**[BLOCK]** E-001 completed.

> **Zero-result return is the evidence artifact.** Export query metadata (query text, execution timestamp, result count = 0) as EV-017.

- If any match returned: do not proceed — re-engage E-001 for the matching endpoint.
- If integrity cannot be confirmed after repeated attempts: activate **[EH-003] R-005**.
- **Evidence:** EV-017 — required input for R-001 blocking condition.
- **→ Next:** E-003

---

### E-003 · Workbench review for residual OAT alerts post-eradication
**Module:** Workbench · **Type:** manual

- Filter for open OAT alerts or new correlated incidents on `AFFECTED_ENDPOINTS` after E-001 completion.
- If new alerts present: do not proceed to E-004 — loop back to T-002 for affected endpoint.
- **Evidence:** EV-001 (reused — Workbench clear status documented with timestamp)
- **→ Next:** E-004

---

### E-004 · Patch deployment for CVE-2009-0238 and CVE-2026-32201
**Type:** external · **[EH-001]**  
**[BLOCK]** E-001 and E-002 completed — eradication confirmed; E-003 Workbench clear confirmed.

- Notify `PATCH_TEAM_CONTACT` with CVE list, `AFFECTED_ENDPOINTS`, and incident reference ID.
- SLA: `PATCH_TEAM_SLA_HOURS` from trigger — escalate to IR Manager if missed.
- **R-001 is hard-blocked until patch deployment confirmation (EV-018) received per endpoint.**
- **Evidence:** EV-018 (patch deployment confirmation — required for R-001 blocking condition)
- **→ Next:** R-001

---

### R-005 · Backup restore for endpoint with unconfirmable integrity
**Type:** external · **[EH-003]** · **[COND]** Activate only if E-002 fails or integrity cannot be confirmed.  

- Notify `BACKUP_TEAM_CONTACT` with affected endpoint list, integrity failure evidence, and incident reference.
- Receive restore confirmation with pre-incident backup timestamp.
- **After restore: E-004 patch requirement still applies before R-001. Do not release without patch.**
- **Evidence:** EV-022
- **→ Next:** R-001 (returns to R-001 after restore; does not skip E-004)

---

## Phase 5 — Recovery

### R-001 · Release endpoint isolation
**Module:** Response Management · **Type:** native · **Action:** RA-005  
**[APR-001]** IR Lead approval required before execution — no exceptions.

> **R-001 blocking condition — all four must be confirmed before requesting APR-001 approval:**
> 1. EV-017 — Q-005 zero-result clean state verification for **all** `AFFECTED_ENDPOINTS`
> 2. EV-018 — Patch deployment confirmation for **all** `AFFECTED_ENDPOINTS` (EH-001)
> 3. EV-015 — Perimeter firewall confirmation from network team (EH-002)
> 4. APR-001 approval from IR Lead received

- Confirm endpoint status = Active in Response Management after release per endpoint.
- Set `RELEASE_TIMESTAMP` per endpoint immediately after release — required by Q-006.
- Set monitoring reminder for `MONITORING_WINDOW_HOURS` duration.
- **Evidence:** EV-019
- **→ Next:** R-002 + R-003 (parallel monitoring)

---

### R-002 · Post-release monitoring via XDR Search App
**Module:** XDR Search App · **Type:** manual · **Query:** Q-006  
**[BLOCK]** R-001 completed; `RELEASE_TIMESTAMP` set.

- Execute Q-006 at minimum once per analyst shift for full `MONITORING_WINDOW_HOURS` duration.
- If high or critical risk event found: cross-reference with R-003 Workbench before escalating.
- **Evidence:** EV-020 (each monitoring run — required for closure)
- **→ Next:** R-003

---

### R-003 · Workbench monitoring for new incident correlation on released endpoints
**Module:** Workbench · **Type:** manual  
**[BLOCK]** R-001 completed.

- Filter Workbench for new open incidents involving `AFFECTED_ENDPOINTS` post-release.
- Run minimum once per analyst shift for `MONITORING_WINDOW_HOURS`.
- If new incident found: re-activate playbook from D-001 with new `TIME_WINDOW_START`.
- **Evidence:** EV-001 (reused — Workbench monitoring status with timestamp)
- **→ Next:** R-004

---

### R-004 · Validate Suspicious Object Management entry TTL
**Module:** Suspicious Object Management · **Type:** manual

- Review TTL for all entries created in C-002 (EV-011).
- Renew any entries approaching expiry before monitoring window ends.
- Document renewal action — update EV-011.
- **Evidence:** EV-021
- **→ Next:** PI-001

---

## Phase 6 — Post-Incident

### PI-001 · Detection model review and tuning
**Module:** Detection Model Management · **Type:** manual  
**[COND]** Skip if `FEATURE_DETECTION_MODEL_MGMT = false` — document detection coverage gap, escalate tuning recommendation to tenant admin.

- Compare first exploitation event timestamp (EV-003) against first OAT alert timestamp (EV-002) — quantify detection lag.
- Apply tuning adjustments or explicitly document that no tuning was required.
- **Evidence:** EV-023
- **→ Next:** PI-002

---

### PI-002 · Estate-wide unpatched asset scan via Attack Surface Risk Management
**Module:** Attack Surface Risk Management · **Type:** manual  
**[COND]** Skip if `FEATURE_ASRM = false` — document that estate-wide exposure assessment was not performed.

- Review asset exposure for CVE-2009-0238 and CVE-2026-32201 across full tenant estate.
- Export report and deliver to patch team via EH-001 for follow-on estate-wide remediation.
- **Evidence:** EV-024
- **→ Next:** PI-003

---

### PI-003 · Export Workbench incident record
**Module:** Workbench · **Type:** manual  
**[BLOCK]** All required closure criteria confirmed (see Closure Criteria below).

> Export before closing or archiving the incident — closed incidents may be purged per tenant retention policy.

- Export full incident record including correlated events, OAT alerts, affected endpoints, and timeline.
- Verify completeness — all correlated events from D-001 through R-004 included.
- **Evidence:** EV-025 — required for PI-004 and audit.
- **→ Next:** PI-004

---

### PI-004 · Post-incident report and IR documentation update
**Type:** external · **[EH-005]**  
**[BLOCK]** PI-003 Workbench incident export completed.

- Transfer full evidence package (EV-001 through EV-025) to `POST_INCIDENT_REPORT_OWNER`.
- Confirm post-incident report distributed to management.
- Confirm IR documentation and this playbook updated with lessons learned.
- Confirm vulnerability management team notified for ongoing CVE tracking.
- **Evidence:** EV-026
- **→ Next:** CLOSED

---

## Stop Conditions

Halt the playbook and escalate immediately in any of these situations:

| Condition | Stop at | Action |
|-----------|---------|--------|
| Isolation fails on any endpoint — agent offline or unreachable | C-001 | Escalate to IR Lead; engage network team for VLAN isolation via EH-002 as fallback |
| Process respawns after C-003 termination | C-003 | Do not re-terminate; re-triage for persistence mechanism before proceeding |
| C-005 file collection fails for a targeted path | C-005 | Retain endpoint in isolation; document failure; escalate before proceeding to E-001 |
| Q-005 returns any match after E-001 | E-002 | Re-engage E-001 for matching endpoint; do not proceed to R-001 |
| New OAT alert on any affected endpoint after E-001 | E-003 | Loop back to T-002 for affected endpoint; do not proceed to E-004 |
| EH-001 SLA breached without patch confirmation | E-004 | Escalate to IR Manager; endpoints remain isolated until patching confirmed |
| EH-002 SLA breached without firewall confirmation | C-006 | Escalate to IR Manager; R-001 remains blocked |
| New correlated incident on any released endpoint during monitoring | R-002 / R-003 | Re-activate playbook from D-001 with new TIME_WINDOW_START |
| R-001 release fails for any endpoint | R-001 | Escalate to Trend Micro support; do not leave endpoints isolated indefinitely without escalation |

---

## Closure Criteria

**All required items must be confirmed before running PI-003.**

### Required
- [ ] All `AFFECTED_ENDPOINTS` released from isolation — R-001 confirmed for all (EV-019)
- [ ] Q-005 zero-result verification confirmed for all `AFFECTED_ENDPOINTS` — EV-017
- [ ] Patch deployment confirmation received per endpoint — EV-018
- [ ] Suspicious Object Management entries active for all confirmed IOCs — R-004 validated (EV-021)
- [ ] `MONITORING_WINDOW_HOURS` post-release monitoring completed with no new alerts — R-002 (EV-020) and R-003
- [ ] Workbench incident export completed — EV-025

### Conditional
- [ ] EH-002 perimeter firewall confirmation received — **required if `MALICIOUS_IPS` is non-empty** (EV-015)
- [ ] EH-003 restore confirmation received — **required if R-005 was activated** (EV-022)
- [ ] EH-004 identity team reset confirmation received — **required if `FEATURE_IDENTITY_SECURITY = false`** (EV-013)

### Recommended (non-blocking)
- [ ] PI-002 Attack Surface Risk Management report delivered to patch team (EV-024)
- [ ] PI-001 detection model tuning applied or documented (EV-023)
- [ ] EH-005 post-incident report distributed (EV-026)
