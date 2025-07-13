### Policy-as-Code

Policy as Code (PaC) is the practice of defining compliance, security, and governance rules in machine-readable formats that can be automatically enforced across infrastructure, applications, and data.
Instead of manual audits and checklists, policies are:

- Version-controlled (Git)
- Automatically validated (CI/CD pipelines)
- Enforced in real-time (pre-deployment & runtime)

*Why is Policy as Code Relevant for Financial Institutions?*
- Regulatory Compliance and adherence to RBZ, FIU, and POTRAZ etc rules programmatically.
- Eliminates manual errors in audits.
- Speed & Agility
- Enables secure DevOps (DevSecOps).
- Auditability
- Cryptographic proof for regulators.


## Examples

* 1. AWS Service Control Policy (SCP) json *
Rule: *No resources can be created without RBZ-mandated tags.*

```
{
  "Version": "2025-07-05",
  "Statement": [
    {
      "Sid": "RequireRBZTags",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "Null": {
          "aws:RequestTag/DataClassification": "true",
          "aws:RequestTag/RetentionPeriod": "true"
        }
      }
    }
  ]
}

```

* 2. Open Policy Agent (OPA) for On-Prem rego *
Rule: *Virtual machines must use FIPS 140-2 encryption.*

```
rego
package fips.encryption

deny[msg] {
    vm := input.virtual_machines[_]
    not vm.encryption.fips_validated
    msg := sprintf("VM %s violates RBZ CDPA Section 9.1", [vm.name])
}
```
 
## Implementation Roadmap

1. Start Simple
2. Record all rules for refence
3. Enforce tagging policies in AWS SCPs or respective platform.
4. Automate Checks
5. Integrate OPA/Rego for on premise solutions into CI/CD pipeline.
6. Expand Coverage
7. Add runtime checks e.g AWS Config Rules.
8. Regulator Integration if available or auto-generate audit reports for regulator.

## Expected Outcome/benefits
1. At least 50% faster compliance reviews
2. Zero non-compliant deployments
3. Real-time reporting
