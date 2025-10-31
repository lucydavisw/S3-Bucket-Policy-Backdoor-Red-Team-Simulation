# S3-Bucket-Policy-Backdoor-Red-Team-Simulation

Author: Lucy Njuguna

Environment: Stratus Red Team sandbox (AWS CLI v2, Ubuntu)

Purpose: Demonstrate a realistic red team technique where a malicious actor introduces a cross-account S3 bucket policy backdoor, validate that it is functional, and revert to baseline.

# TL;DR

This repository documents a safe, sandboxed Red Team exercise that shows how a bucket policy can be modified to grant an external principal s3:ListBucket and s3:GetObject permissions. The simulation proves the change is effective for exfiltration and demonstrates cleanup to restore baseline. The exercise trains red teamers on realistic tactics and blue teamers on detection and response. 

# S3 Bucket Policy Backdoor Disc; Why this matters for Red Teams

Red teams need repeatable, realistic scenarios that model how attackers gain persistent or stealthy exfiltration access. Modifying bucket policies is an attractive adversary tradecraft because it does not require long-lived credentials and can bypass rotated keys. This simulation teaches how to safely introduce and validate such a backdoor, how to document attack steps, and how to leave the environment clean for repeat testing. 


# Contents
#S3-Bucket-Policy-Backdoor/
├── README.md
├── S3 Bucket Policy Backdoor Discussion Logeport.pdf
└── screenshots/
    ├── baseline_check.png
    ├── detonation_output.png
    ├── verification_result.png
    └── cleanup_confirmation.png

# Scope and Rules of Engagement

This is a controlled simulation run in a Stratus Red Team sandbox. No production or third-party resources were targeted. All actions were confined to an isolated test environment and reverted during cleanup. Unauthorized reproduction of these techniques against systems you do not own or have permission to test is illegal and unethical. 



Exercise Overview (Red Team Playbook)
# 1. Recon & Baseline

Establish the bucket baseline and verify policy absence or current policy state before any changes. Example command used in the sandbox:

aws s3api get-bucket-policy --bucket stratus-red-team-bdbp-zkqgfovijw


In the simulation the command returned NoSuchBucketPolicy, confirming a clean baseline. 


# 2. Detonate (Introduce the Backdoor)

Use the Red Team harness to apply a backdoor-style bucket policy that allows an external AWS principal cross-account read/list access:

stratus detonate aws.exfiltration.s3-backdoor-bucket-policy


Verification returned a policy with Allow for s3:GetObject, s3:ListBucket, and s3:GetBucketLocation against the bucket ARNs. 


# 3. Functional Probe (Validate Access)

Simulate the external principal or use the test harness to confirm the policy permits listing and reading objects:

status aws.exfiltration.s3-backdoor
or use a simulated principal to attempt aws s3 ls / aws s3 cp


The policy string contained indicators like BackdoorReadAccess, and the permitted actions were confirmed. 


# 4. Persistence Considerations (Red Team Thought)

If an adversary wanted persistent access they might create or use an external principal they control. In this simulation, the principal was synthetic and ephemeral, mirroring an attacker who might re-use cross-account principals or IAM roles.

# 5. Cleanup (Leave No Trace)

Always revert changes and clean resources after the exercise:

stratus revert aws.exfiltration.s3-backdoor-bucket-policy
stratus cleanup


Reversion removed the policy and restored the bucket baseline in the sandbox. 



# Example (Paraphrased) Policy Observed

The following is a compact, paraphrased version of the observed allow statement (for documentation; the real JSON is in the discussion log):

{
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::EXTERNAL:root"},
  "Action": ["s3:GetObject","s3:ListBucket","s3:GetBucketLocation"],
  "Resource": ["arn:aws:s3:::stratus-red-team-bdbp-zkqgfovijw/*","arn:aws:s3:::stratus-red-team-bdbp-zkqgfovijw"]
}


This policy would permit cross-account listing and object reads, enabling exfiltration if the external principal is accessible. 

S3 Bucket Policy Backdoor Disc…

Detection Guidance (Blue Team)

To detect or limit this technique in real environments, implement the following monitoring and controls:

Alert on bucket policy changes. Configure CloudTrail to log PutBucketPolicy and trigger alerts if policy is written or modified. Monitor principal fields for external accounts.

Policy drift checks. Periodically capture and compare bucket policies. If a previously absent policy appears, trigger investigation.

IAM and cross-account governance. Block broad Principal statements and require least privilege approvals for cross-account statements.

S3 Access Analyzer & IAM Access Advisor. Use these to flag potentially risky resource-based policies.

Data exfil metrics. Alert on unusual GetObject or large-volume ListBucket operations especially from new principals or endpoints.

Red Team Variations & Exercises

Use the following variations for deeper red-team training:

Attempt time-limited policies to simulate transient backdoors.

Combine policy changes with temporary STS role assumption to model multi-stage exfiltration.

Test detection tuning by introducing small-volume reads to see if alerts trigger.

Try privilege escalation detection when both resource-based and identity-based permissions are modified.

Recommendations for Safe Practice

When you run similar exercises, follow these rules:

Always run in a sandbox or isolated account with written authorization.

Document every change and verification step to support blue team triage and remediation.

Revert and validate cleanup before ending the engagement.

Coordinate with stakeholders and include a post-exercise debrief to improve detection rules.

Findings from This Simulation

A bucket policy can be used as a stealthy exfiltration mechanism that bypasses credential rotation if the attacker controls the external principal. 

S3 Bucket Policy Backdoor Disc…

The Stratus Red Team harness made it straightforward to replicate the end-to-end scenario including warmup, detonation, verification, and cleanup. 

S3 Bucket Policy Backdoor Disc…

Ethics and Legal Notice

This repository documents techniques for authorized Red Team engagements and defensive training. Misuse on systems without explicit permission is illegal. Do not run these techniques on production systems or third-party accounts without written authorization.

Appendix — Commands Used (quick reference)
# Prepare environment
stratus warmup aws.exfiltration.s3-backdoor-bucket-policy

# Baseline check
aws s3api get-bucket-policy --bucket stratus-red-team-bdbp-zkqgfovijw

# Apply backdoor
stratus detonate aws.exfiltration.s3-backdoor-bucket-policy

# Verify policy
aws s3api get-bucket-policy --bucket stratus-red-team-bdbp-zkqgfovijw

# Functional probe
status aws.exfiltration.s3-backdoor

# Revert & cleanup
stratus revert aws.exfiltration.s3-backdoor-bucket-policy
stratus cleanup
