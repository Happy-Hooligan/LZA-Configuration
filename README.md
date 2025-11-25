# Landing Zone Accelerator on AWS (LZA) Universal Configuration

The LZA Universal Configuration provides enterprise-ready configuration templates that utilize the [Landing Zone Accelerator on AWS](https://aws.amazon.com/solutions/implementations/landing-zone-accelerator-on-aws/) (LZA) to establish secure, scalable, and well-architected multi-account AWS environments. It enables rapid deployment of baseline environments supporting multiple global compliance frameworks and industry-specific requirements.

## Specific Regional and Industry Guidelines

For detailed information about supported regions and industry-specific configurations, including compliance frameworks, regulatory requirements, and implementation guidance, please refer to:

**[Support for Regions and Industries](https://docs.aws.amazon.com/solutions/latest/landing-zone-accelerator-on-aws/support-for-regions-and-industries.html)**

This comprehensive guide covers:

- Regional compliance requirements
- Industry-specific security controls and frameworks
- Regulatory compliance mappings

## What's Included

The LZA Universal Configuration provides ready-to-deploy configuration files and implementation guidance for LZA Universal Configuration baseline:

### Configuration Modules

Pre-configured LZA Universal Configuration files organized by the following deployment patterns:

- **[Base configurations](./modules/base/default)** - Security, Governance, and Organization settings
- **[Network configurations](./modules/network)** - Hub-and-spoke, and Shared VPC networking patterns

### Documentation

Architecture guides, deployment procedures, and detailed design principles that explain the implementation rationale and configuration decisions, supported by architectural diagrams and technical documentation for secure multi-account AWS environments.

### Cost

You are responsible for the cost of the AWS services used while running this solution. As of this revision, the cost for running the Universal Configuration in its current form through the Landing Zone Accelerator on AWS with AWS Control Tower in the US East (N. Virginia) Region within a non-critical sandbox environment with no activity or workloads is approximately **$101.06 (USD)** each month.

We recommend creating a budget through AWS Cost Explorer to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this solution.

## Sample Cost Table

The following table provides a sample cost breakdown for deploying this solution with the default parameters in the US East (N. Virginia) Region, with no activity, for one month.

| AWS Service | Monthly Cost [USD] |
|-------------|-------------------|
| Amazon Network Firewall | $585.40 |
| AWS CloudTrail | $29.46 |
| AWS Config | $55.95 |
| AWS KMS | $367.10 |
| Amazon Kinesis | $61.85 |
| Amazon Data Firehose | $2.99 |
| Amazon S3 | $4.86 |
| Amazon VPC | $458.53 |
| Amazon CloudWatch | $22.21 |
| AWS Security Hub | $221.92 |
| Amazon GuardDuty | $10.84 |
| Amazon Route 53 | $5.00 |
| Amazon Macie | $7.16 |
| AWS Secrets Manager | $0.44 |
| AWS CodePipeline | $1.00 |
| AWS CodeBuild | $1.52 |
| **Total** | | **$1,842.08** |

> **Note:** The VPC calculation is done if you're utilizing a hub-and-spoke topology. Data transfer, AWS CodeArtifact, Amazon Detective, Amazon DynamoDB, AWS Lambda, AWS Service Catalog, Amazon Simple Notification Service (Amazon SNS), Amazon Simple Queue Service (Amazon SQS), AWS Step Functions, and AWS Systems Manager are priced at the Free Tier or less than $0.01 each month.



### Scripts and CI/CD Integration

Automation utilities for configuration management and environment setup, including account configuration generation and deployment helpers. The `.gitlab-ci.yml` file demonstrates how to integrate these configurations with CI/CD options for automated multi-environment deployments.

---

IMPORTANT

LZA Universal Configuration will not, by themselves, make you compliant. They offer an opinionated, best-practice approach to establishing a well-architected AWS multi-account environment from which additional complementary solutions can be integrated. The information is not exhaustive - you must review, evaluate, and approve the solution according to your organization's security requirements. You are responsible for determining applicable regulatory requirements and ensuring compliance. While this solution addresses technical and administrative requirements, it does not help with non-technical administrative compliance requirements.

---

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
