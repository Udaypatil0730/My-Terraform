🔹 Top 10 Real-World Terraform Scenario Interview Questions
Q1. Terraform State Locking

Scenario: Multiple developers are working on the same Terraform project. How does Terraform ensure state consistency?

Explanation: Terraform uses backend state locking (e.g., S3 + DynamoDB). When a user runs terraform apply, it writes a lock entry in DynamoDB. Others cannot run Terraform until it’s unlocked. Prevents state corruption.

Interview Answer:

Terraform uses state locking via the backend. For example, with S3 + DynamoDB, when a user runs apply, a lock record is created in DynamoDB. This prevents others from making concurrent changes. When the operation finishes, the lock is released. If it’s stuck, I can use terraform force-unlock.

Keywords: Backend, S3 + DynamoDB, Locking, Force-unlock, Prevent corruption

Q2. Credential Management

Scenario: You need to pass AWS credentials in Terraform. How do you handle them securely?

Explanation: Never hardcode creds. Use env vars (AWS_ACCESS_KEY_ID), AWS profiles, IAM roles, or Vault/Secrets Manager. If inside Terraform, use sensitive variables + tfvars + gitignore.

Interview Answer:

I never hardcode credentials. For CI/CD, I inject them via environment variables or secrets manager. If I must store inside Terraform, I use sensitive variables in tfvars files excluded from Git. On AWS EC2/EKS, I rely on IAM roles so no creds are stored at all.

Keywords: Sensitive variables, tfvars, gitignore, Env vars, IAM role, Vault

Q3. Changing EC2 Instance Type

Scenario: You need to upgrade an EC2 instance from t2.micro → t2.medium.

Explanation: Just update instance_type in Terraform code. Terraform tries in-place update (stop → modify → start). Some changes (AMI, subnet) force replacement. For zero downtime → blue/green strategy.

Interview Answer:

I update the instance_type in Terraform code and run plan + apply. Terraform updates in-place if possible. If not, it destroys and recreates. In production, I prefer creating a new instance with new type and attaching via load balancer (blue/green) to avoid downtime.

Keywords: Instance_type, In-place vs Replacement, Blue/green

Q4. Rollback of Terraform Changes

Scenario: You applied Terraform, it broke production. How do you rollback?

Explanation: Terraform has no rollback button. Rollback = reapply last known good state. Either use Git versioning of .tf code + terraform apply, or restore a specific state file version from S3 backend.

Interview Answer:

Terraform doesn’t auto-rollback. To rollback, I use Git to revert to the last stable version of code and reapply. If needed, I can restore an older state version from the remote backend (like S3) and reapply it.

Keywords: No rollback, Git version control, State file restore

Q5. Terraform Drift Detection

Scenario: Someone manually changed an EC2 instance type in AWS console. How do you detect and fix this drift?

Explanation: Run terraform plan. It compares real infra vs state file. Shows drift (e.g., console changed t2.micro → t2.small). Running terraform apply will bring infra back to desired state.

Interview Answer:

I run terraform plan to detect drift. Terraform compares actual AWS infra vs state and shows mismatches. Then I apply to reconcile — Terraform enforces the desired state defined in code.

Keywords: Drift detection, terraform plan, Desired vs Actual, Reconcile

Q6. Terraform Import

Scenario: You already have resources created manually in AWS. How do you bring them under Terraform management?

Explanation: Use terraform import to map existing AWS resources into state. Then write matching resource block in code. Import only updates state, not code, so code must exist.

Interview Answer:

I use terraform import to bring existing AWS resources into Terraform state. For example, terraform import aws_instance.my_ec2 i-12345. Then I add the matching resource code so Terraform can manage it going forward.

Keywords: terraform import, Existing infra, State mapping, Code sync

Q7. Remote State Sharing

Scenario: Your infra is split across multiple teams (network team, app team). How do they share Terraform outputs?

Explanation: Use terraform_remote_state data source. One team’s Terraform state outputs can be consumed by another team. Example: network outputs VPC ID → app team uses it.

Interview Answer:

We use remote state data source. For example, the network team outputs the VPC ID to S3 backend. The app team references it using data "terraform_remote_state". This keeps teams modular and avoids duplication.

Keywords: Remote state, Data source, Outputs, Team collaboration

Q8. Terraform Modules

Scenario: You need to reuse VPC/EC2 configuration across multiple environments (dev/stage/prod). How do you handle this?

Explanation: Use Terraform modules. Create reusable module (VPC, EC2, RDS). Pass variables per environment. Use workspace or separate tfvars.

Interview Answer:

I package reusable infra as Terraform modules. For example, a VPC module with variables for CIDR and subnets. Each environment passes its own values. This avoids code duplication and ensures standardization.

Keywords: Modules, Reuse, tfvars, Workspaces

Q9. Handling Sensitive Data in State

Scenario: DB password stored in Terraform state. Isn’t that insecure? How do you protect it?

Explanation: Terraform state may contain secrets. Protect by storing state in encrypted S3 with restricted IAM. Use KMS encryption. Use sensitive = true to hide in outputs.

Interview Answer:

Sensitive data can appear in state files. To protect, we use remote backends like S3 with server-side encryption and DynamoDB for lock. IAM restricts access to state. Also, I mark variables as sensitive so they don’t appear in CLI or logs.

Keywords: State file, Encryption, S3 + KMS, IAM, Sensitive flag

Q10. Zero Downtime Deployment

Scenario: You want to upgrade an ASG (Auto Scaling Group) or ECS service via Terraform with zero downtime. How?

Explanation: Use rolling update strategy. For ASG, Terraform can create new launch template + new ASG, attach to ELB, then remove old one (create_before_destroy). For ECS, Terraform updates service definition gradually.

Interview Answer:

For zero downtime, I use create_before_destroy in lifecycle block or blue/green strategy. Example: with ASG, Terraform creates a new launch template and replaces old instances gradually while traffic is handled by the load balancer.

Keywords: Zero downtime, create_before_destroy, Blue/green, Rolling update
