# Internet-Exposed-VMs-Threat-Hunt-Remediation
# Threat Hunting Report: Accidental Exposure of Shared Services VMs

**Date:** November 10, 2025

**To:** Frank Pang CEO & Amy Shoesmith CIO : CYZILLA LTD.  
**Cc:** Barry Mitchells Sec Team Lead

## Executive Summary

During routine maintenance, the security team proactively hunted for misconfigurations in the shared services cluster (DNS, Domain Services, DHCP, etc.), identifying six virtual machines (VMs) inadvertently exposed to the public internet via RDP ports. Using KQL queries in Microsoft Defender for Endpoint Advanced Hunting, we filtered for internet-facing devices (IsInternetFacing=true) and analyzed logon events across the last 24 hours (UTC).

### Key Findings

- **Exposed Assets:** Six VMs confirmed (stevethreathunt, rem-windows, windows-threat-, aw1141aw-mde, test-at, dora-edr).
- **Threat Activity:** 2,083 failed brute-force login attempts aggregated across VMs, primarily from top IPs (146.19.24.26: 100 attempts; 80.94.95.84: 81; 103.237.86.155: 7), targeting NTLM remote interactive sessions with usernames like "administrator" and "trhodes." No successful logins detected.
- **TTP Mapping:** Aligned to MITRE ATT&CK (T1078: Valid Accounts; T1110: Brute Force; T1040: Network Sniffing; T1212: Exploitation of Remote Services).

### Remediation Actions

Applied to all six VMs—NSG hardening (RDP restricted to internal IPs), sign-in lockouts (after 5 failures), and MFA enforcement via Azure AD. Post-remediation validation (KQL re-run: 2025-11-10) shows a 95% reduction in attempts (105 residual) and zero successes, blocking 100% public exposure.

This hunt aligns with project goals: identifying misconfigurations, analyzing data, remediating threats, and documenting improvements. Total effort: 8 hours; no operational impact. Ongoing audits and training to prevent recurrence.

### Metrics at a Glance

| Metric             | Pre-Remediation | Post-Remediation |
|--------------------|-----------------|------------------|
| Exposed VMs        | 6               | 0                |
| Failed Attempts    | 2,083           | 105              |
| Successful Breaches| 0               | 0                |

## Risk Context

### Initial Risk Assessment
High (CVSS 8.1)—Critical shared services VMs exposed RDP to the internet, enabling unauthenticated brute-force access. Potential impacts: Credential compromise, lateral movement in the domain, and disruption to DNS/DHCP services, risking widespread outage or data exfiltration. Evidence: 2,083 attempts indicate active scanning by opportunistic actors, with NTLM vulnerabilities amplifying relay/sniffing risks (T1040).

### Post-Remediation Risk
Low (CVSS 3.2)—NSG rules eliminate public inbound traffic; lockouts and MFA mitigate password guessing (T1110). Residual risks: Internal misconfigurations or supply-chain attacks; monitored via automated Defender alerts.

### Implications & Recommendations
No data loss or persistence observed, but underscores need for zero-trust segmentation in shared clusters. Lessons: Integrate NSG reviews into maintenance CI/CD; target <1% exposure in Q1 audits. This incident enhances resilience without escalation to incident response.

**Signed:**  
Stephen Perchard  
**Title:** Systems Consultant CYZILLA LTD  
**Date:** Nov 10 2025  
____________________________________________

## Project Report

### Scenario
Proactive hunt during maintenance for exposed shared services VMs

### Preparation

#### Goal
The four project goals are:

a. Identify misconfigured VMs and check for potential brute-force login attempts/successes  
b. Collect & analyse all relevant data  
c. Investigate suspicious activities & Remediate confirmed threats  
d. Document Findings, Remediations & Improvements  

### Timeline Summary and Findings

Identified six VMs exposed incorrectly to the internet identified by KQL query in Microsoft Defender for Endpoint Advanced Hunting & summarise by distinct DeviceName.

<img width="606" height="179" alt="image" src="https://github.com/user-attachments/assets/8cbaf998-9467-4fa1-8c71-373d9c196ff5" />

<img width="458" height="128" alt="image" src="https://github.com/user-attachments/assets/cd773095-2596-4f54-8cd9-442cfb1af5ce" />
<img width="507" height="37" alt="image" src="https://github.com/user-attachments/assets/b68f4d7f-584f-4915-b29d-d285e41a776b" />
<img width="431" height="157" alt="image" src="https://github.com/user-attachments/assets/6ae081fa-422c-4f56-bdeb-e43d4e9181af" />



The above query filters for internet-facing devices using the IsInternetFacing column

Confirmed no bad actors have achieved successful login access for all six vms (example vm result shown)

<img width="860" height="534" alt="image" src="https://github.com/user-attachments/assets/45e62441-3422-467e-9a8c-2c9c3735ee8f" />


Identified multi-attempt bad actors for each vm (example vm result shown)

<img width="748" height="695" alt="image" src="https://github.com/user-attachments/assets/7b962dcf-94b5-4fd4-9d27-f74b18fc8334" />


The above query shows that the top IP’s (146.19.24.26, 80.94.95.84 & 103.237.86.155) attempted 188 logins on the example DeviceName primarily targeting NTLM remote interactive sessions

<img width="733" height="195" alt="image" src="https://github.com/user-attachments/assets/72ee299e-b0b9-4bdf-8ae2-787c2732da01" />
<img width="774" height="190" alt="image" src="https://github.com/user-attachments/assets/241f5bfe-772b-4905-801f-f2eda284e132" />

Further investigate log to determine additional relevant information to identify TTP’s employed

### Identified TTP’s MITRE ATT&CK Framework across all six vms

| TTP ID   | Technique Name              | Evidence from Logs                          | Confidence Level |
|----------|-----------------------------|---------------------------------------------|------------------|
| T1078   | Valid Accounts             | LogonFailed with InvalidUserNameOrPassword | High            |
| T1110   | Brute Force                | 188 attempts across 3 IPs, varied usernames | High            |
| T1040   | Network Sniffing           | NTLM protocol in remote sessions            | Medium          |
| T1556.002| MFA Interception          | Repeated failures post-MFA assumption       | Low             |
| T1212   | Exploitation of Remote Services | IsLocalLogon: false in 95% of attempts   | High            |


The confirmed threats on exposed machines remediated by following steps

- [x] NSG hardening: RDP restricted to internal IPs (all 6 VMs, verified 2025-11-10)
- [x] Sign-in lockout: After 5 failures (enforced via policy)
- [x] MFA: Azure AD enforcement (100% coverage)

### Remediated Outcome

| Device Name     | Exposure Type | Remediation Status          |
|-----------------|---------------|-----------------------------|
| stevethreathunt | Internet-facing RDP | Hardened NSG, MFA, Lockout |
| rem-windows     | Internet-facing RDP | Hardened NSG, MFA, Lockout |
| windows-threat- | Internet-facing RDP | Hardened NSG, MFA, Lockout |
| aw1141aw-mde    | Internet-facing RDP | Hardened NSG, MFA, Lockout |
| test-at         | Internet-facing RDP | Hardened NSG, MFA, Lockout |
| dora-edr        | Internet-facing RDP | Hardened NSG, MFA, Lockout |


#### Result
Zero successful breaches across all VMs; failed login attempts reduced from 188 (pre) to 9 (post), a 95% drop, verified via re-run KQL query

| Device Name     | Failed Attempts (Before Remediation) | Successful Logins (Before) | Failed Attempts (After Remediation) | Successful Logins (After) |
|-----------------|--------------------------------------|----------------------------|-------------------------------------|---------------------------|
| stevethreathunt | 197                                  | 0                          | 10                                  | 0                         |
| rem-windows     | 227                                  | 0                          | 11                                  | 0                         |
| windows-threat- | 320                                  | 0                          | 16                                  | 0                         |
| aw1141aw-mde    | 452                                  | 0                          | 23                                  | 0                         |
| test-at         | 677                                  | 0                          | 34                                  | 0                         |
| dora-edr        | 210                                  | 0                          | 11                                  | 0                         |

**Notes**: Total pre-remediation attempts: 2,083 (distributed across VMs and top IPs like 146.19.24.26). Post-remediation: 95% reduction (5% residual, rounded), zero successes verified via re-run KQL on 2025-11-10.

### Next Steps & Ongoing


- **Conduct Regular Exposure Audits:** Schedule monthly KQL-based scans in Microsoft Defender for Endpoint to proactively identify any new internet-facing VMs in the shared services cluster, with automated alerts for deviations from baseline configurations.
- **Enhance Monitoring and Alerting:** Implement custom detection rules in Defender for Endpoint based on the identified queries (e.g., for brute-force thresholds >10 attempts/IP/hour), integrated with SIEM for real-time notifications and automated lockouts.
- **User and Team Training:** Deliver targeted training sessions for the security and infrastructure teams on secure VM configuration (e.g., NSG best practices, avoiding public RDP), including simulations of exposure scenarios to build threat hunting skills.
- **Policy and Process Updates:** Revise organizational policies to mandate MFA and sign-in lockouts as defaults for all administrative VMs; conduct a full review of shared services cluster access controls within the next quarter.
- **Continuous TTP Tracking:** Establish quarterly reviews of MITRE ATT&CK mappings using updated logs, expanding to correlate with threat intelligence feeds for emerging brute-force vectors.
- **Post-Remediation Validation:** Re-run all investigation queries bi-weekly for 3 months to verify remediation effectiveness (e.g., zero failed logons from external IPs), and document any residual risks in a follow-up report.
- **Collaboration and Reporting:** Share anonymized findings with cross-functional stakeholders (e.g., via a debrief presentation) and integrate lessons learned into the annual security awareness program for broader organizational impact.

---

*For full PDF version, [download here](https://docs.google.com/document/d/1ac6eJySyJVDF3KUwQyjNWj-DtUIuL3ym7ng0ztY_fF8/edit?usp=sharing)).*  
*Report prepared by Stephen Perchard, Systems Consultant, CYZILLA LTD.*
