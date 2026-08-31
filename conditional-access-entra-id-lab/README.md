# Conditional Access & Entra ID Security Lab

*Template — replace the bracketed prompts as you build the project. Delete these italic notes once filled in.*

## Overview

A short paragraph: what this project is and why you built it. Example starting point — "A hands-on lab designing and documenting Microsoft Entra ID Conditional Access Policies, built in a personal sandbox tenant to demonstrate identity security concepts relevant to SOC analyst work."

## Objective

What you set out to learn or demonstrate. Example: "Design a layered set of Conditional Access Policies covering MFA enforcement, legacy authentication blocking, risk-based access, and device compliance, and document the reasoning and attack scenarios behind each one."

## Environment

- Tenant: Microsoft 365 Developer Program sandbox (free, personal, not affiliated with any employer or client)
- - Licensing: [note if you have Entra ID P1/P2 features available, since risk-based policies need P2]
  - - Test users/groups: [how many, what roles you set up to test policies against]
   
    - ## Policies Designed
   
    - For each policy, a short entry works well, condition, why it matters, and what it defends against. Example structure:
   
    - ### 1. MFA Enforcement for All Users
    - - What it does: [assignment scope, grant controls]
      - - Why: Reduces risk from credential compromise; even a leaked password isn't enough on its own.
        - - What it would catch in the real world: Sign-in attempts with a valid password but no second factor, a classic sign of a phished or breached credential.
         
          - ### 2. Block Legacy Authentication
          - - What it does: [scope, protocols blocked]
            - - Why: Legacy protocols (IMAP, POP, SMTP AUTH) don't support MFA, making them a favourite target for credential-stuffing and password-spray attacks.
              - - What it would catch: A spike in failed legacy-auth sign-ins is a common early indicator of a password-spray campaign.
               
                - ### 3. Risk-Based Access (Sign-in Risk / User Risk)
                - - What it does: [risk levels used, action taken, block, require MFA, require password change]
                  - - Why: Uses Microsoft's threat intelligence signals (impossible travel, anonymised IPs, leaked credentials) to react automatically to risky sign-ins.
                    - - What it would catch: Impossible travel alerts, sign-ins from anonymising proxies/Tor, or use of credentials found in a breach dump.
                     
                      - ### 4. Device Compliance Requirement
                      - - What it does: [scope, compliance requirement]
                        - - Why: Ensures access only happens from devices meeting security baselines (encryption, OS patch level, etc.), reducing the blast radius of a compromised personal device.
                          - - What it would catch: Access attempts from unmanaged or non-compliant devices, a common pattern when an attacker is using a device that isn't the legitimate user's.
                           
                            - Add more policies as you build them.
                           
                            - ## Screenshots
                           
                            - [Add screenshots of the policy configuration screens, blur/redact anything tenant-specific like tenant ID if you don't want it public.]
                           
                            - ## What I'd Investigate as a SOC Analyst
                           
                            - A short section connecting this to actual SOC triage, e.g. "Each of these policies generates sign-in log entries and, where applicable, risk detections in Entra ID. In a SOC role, these would feed into a SIEM like Microsoft Sentinel, where an analyst would triage alerts such as [example], correlate them with [other signal], and decide whether to escalate."
                           
                            - ## Lessons Learned
                           
                            - Honest notes on what was tricky, what surprised you, what you'd do differently. This section often reads best to reviewers, it shows genuine understanding rather than a checklist.
                           
                            - ## Resume/Interview Summary
                           
                            - One or two sentences you could reuse elsewhere, written once this project is done. Draft to refine later:
                           
                            - "Designed and documented a layered Microsoft Entra ID Conditional Access strategy in a personal lab tenant, covering MFA enforcement, legacy authentication blocking, risk-based access, and device compliance controls, mapped to real-world SOC detection and triage scenarios."
