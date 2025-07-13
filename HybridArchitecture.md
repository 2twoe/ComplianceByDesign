# Hybrid On-Premise/Cloud Architecture
To enhance compliant architecture, developing a hybrid solution which has co-existance of on-premise and cloud compmentes that keeps sensitive customer and transaction data within local control, while using cloud services for auxiliary workloads for example non-PII services and analytics.
This is a strategy for balancing regulatory control with cloud agility for regulators e.g RBZ/FIU/POTRAZ compliance

## Core Design Principles
1. Data Sovereignty First
   - PII/core banking data stays on-premise (RBZ compliant)
   - Non-sensitive workloads e.g analytics, HR hosted in AWS 

2. Secure Hybrid Fabric
   - AWS Outposts/Direct Connect for encrypted low-latency links
   - FIPS 140-2 validated encryption at rest/transit (POTRAZ CDPA)

3. Unified Policy Enforcement
   - Consistent controls across cloud/on-prem via HashiCorp Boundary/OPA

## Key Components

| On-Premise DC| 	AWS Region | 	Shared Tools| 
| :---:   | :---: | :---: |
|Core banking VMs (SWIFT Alliance) | Outpost for regulated workloads	| Terraform for hybrid provisioning|
| Thales HSM for key management	| Local Zone for non-PII apps	|OPA policy engine|
| FIU transaction monitoring	| Macie for data classification	| Splunk hybrid SIEM |

## Implementation Examples
1. Data Residency Guardrails python code

```
# Data routing decision engine
def route_transaction(tx):
    if tx["amount"] > 10000 or tx["type"] == "PII":
        return onprem_processing(tx)  # RBZ compliance
    else:
        return aws_processing(tx)     # Cloud scale
````
2. AWS Outpost Configuration (terraform

```
   module "rbz_outpost" {
  source = "terraform-aws-modules/outposts/aws"
  
  name              = "core-banking-outpost"
  availability_zone = "af-south-1a"
  vpc_id           = aws_vpc.compliance_vpc.id
  
  # POTRAZ encryption requirements
  storage_encryption = {
    enabled    = true
    kms_key_id = aws_kms_key.rbz_key.arn
  }
  
  tags = {
    DataClassification = "PII",
    RetentionPeriod    = "7y" # FIU mandate
  }
}
```
3. On-Prem Security Controls (powershell)

```
# Enforce FIPS 140-2 on Hyper-V Cluster
Set-VMHost -Name "ZW-CORE-01" -EnhancedSessionMode $true
Enable-VMTPM -VMName "SWIFT-Gateway" -ProtectorType HSMBased -ThalesHSM "hsm-01.zw.rbz"
```

## Compliance Evidence Generated
Automated RBZ reports showing:

```
bash
aws config get-compliance-details-by-config-rule \
  --config-rule-name "rbz-data-residency" \
  --compliance-types NON_COMPLIANT \
  --query "EvaluationResults[].EvaluationResultIdentifier"
```
- Immutable audit logs signed by on-prem HSM
- FIU transaction monitoring alerts (Splunk CIM format)

## Regulatory Mapping
| Requirement |	Cloud Implementation |	On-Prem Implementation|
| :---:   | :---: | :---: |
| RBZ Localization (Circular 5/2022) | Outpost resource tagging|	StorageGRID WORM locking|
| FIU AML Monitoring|  GuardDuty + Macie|	Splunk ES with FIU rules|
| POTRAZ Encryption| KMS with CMKs| Thales CipherTrust HSM| 

