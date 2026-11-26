# iac/aws-emulator

Optional Terraform / CLI demos against **[AWS Emulator](https://aws.amazon.com/blogs/aws/test-your-applications-against-aws-locally-with-aws-emulator/)** for AWS-shaped narrative (**IAM**, **S3**, etc.). Container images use **GHCR** in CI—not an emulated registry. Not required for the Next.js MVP.

**Milestone:** [M5](../../docs/milestones.md). Implementation: **TBD**. See [ADR 0001](../../docs/adr/0001-local-dev-stack.md).

Do **not** use the emulator to validate real IAM/resource-policy enforcement the way production AWS does.
