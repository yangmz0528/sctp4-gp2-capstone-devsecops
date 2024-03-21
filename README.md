# (SCTP) Cloud Infrastructure Engineering Capstone Projection

## Overview:
Case 3 - DevSecOps

The team is engaged by a startup company (N Repairs) to implement CI/CD pipeline with security scans for their company website. The primary objective of this CI/CD pipeline is to ensure swift and secure deployment processes. By integrating automated security scans into the CICD workflow, security vulnerabilities and compliance issues will be raised early in the development cycle. This not only accelerates the deployment of new features and updates but also fortifies the website against potential threats, safeguarding sensitive data and maintaining user trust.

## Project Requirements 
1. CI/CD Pipeline
2. Implement vulnerability/dependency scanning in CI Script 
3. Ensure proper authentication and authorization in each environment in CI Script 
4. Proper handling of CI/CD Pipeline secrets

## Branching strategy

![branching strategy drawio](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/145353293/a8363f4d-4352-4e3d-975a-f418d9fd5021)

The branching strategy is a simple one, priority has been placed on stability of the codebase and developers to work efficiently. To achieve this balance, we've set up specific protections on `main` branch.

##### The current `main` branch protection rules:
1. Requires pull request before merging
2. Requires at least `1` approval before merging
3. Requires status checks before merging 

The `main` branch remains protected, which contains the stable and thoroughly tested production version of the client's website. 
A merge to `main` will only be allowed after passing all the security/vulnerability scans in `Dev`. 

`Dev` serves as our default branch, where most of the work is done. Over here, new features are developed and bugs are squashed. Ideally, all developers are to create their feature branches from `dev` so they can work on different features concurrently. After completing their features they will merge back into `dev`. 

## CI/CD Pipeline Secrets
-insert screenshot-

## CI/CD Pipeline

![GitHub Workflow - Capstone Project](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/145353293/6e0e9844-0010-44f5-a226-97636cf5358e)

The GitHub action workflow would trigger upon a push to either `main` or `dev`.

- `synk-scan` - Perform [Synk](https://github.com/snyk/actions/tree/master/python-3.10) identifying and addressing vulnerabilities in dependencies and container images
- `bandit-scan` - Perform [Bandit](https://github.com/PyCQA/bandit) analyze Python code for security issues
- `build-image-ecr` - Builds image and push it to ECR
- `trivy-vul-scan` - Perform [Trivy](https://github.com/aquasecurity/trivy-action) vulnerability scan on image stored in ECR
- `ecr-to-ecs` - Deploy image from ECR to ECS
- `zap-scan` - Using [Zap Baseline](https://github.com/marketplace/actions/zap-baseline-scan)to find vulnerabilities in the web application after it has been deployed
- `create-pr-to-main` - Create automated pull request to main after successful deployment (only from `dev')

For the final job all jobs in the GitHub Actions workflow must pass successfully. This ensures that no unverified code enters `main`.

## Roadmap
- Implement automated deployment of AWS assets within the CI/CD pipeline

## Troubleshooting/Learning points: 
1. If an existing pull request exists, the `create-pr-to-main` job will not create a new pull request. 
2. `Dev` branch lacks protection which is likely to cause merge conflicts if developers fail to pull anytime before working on their independent feature/bugfixes. 
