# Change Log

All notable changes to this project will be documented in this file.

## Important

We highly recommend that you keep your environments up to date by upgrading to the latest version.

## [1.1.0] - 2025-12-11

### CRITICAL

If you're on an LZA installation in a non-standard AWS partition (e.g., AWS GovCloud (US)), please do not upgrade to 1.14.0 and use 1.14.1 instead. Please see this [issue](https://github.com/awslabs/landing-zone-accelerator-on-aws/issues/972).

### New Features

- **Regional Guidance**: [Added US Federal region and industry guidance](./docs/08-Region-and-Industry-Guidance/us-federal/)
- **Cost Information**: Added pricing information to help estimate deployment costs
- **AWS Control Tower**: Updated to AWS Control Tower Landing Zone 4.0

### Improvements

- **Automated Builds**: Enhanced CI/CD with AWS GovCloud (US) package generation and security scanning
- **Documentation Updates**: Added high-level architecture and organizational unit (OU) structure diagrams

### Bug Fixes

- **Networking**: Removed internet gateway from inspection VPC
- **Documentation**: Fixed broken links and documentation errors
- **Control Tower**: Fixed issue where controls were not deploying to sub-organizational units

### Contributors

We want to extend a special thank you to the following external users who have contributed to this release:

- @Conklin-Spencer-bah

## [1.0.0] - 2025-09-29

### General Changes

- Initial General Availability (GA) release