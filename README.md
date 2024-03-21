# (SCTP) Cloud Infrastructure Engineering Capstone Projection

## Overview:
Case 3 - DevSecOps

The team is engaged by a client (N Repairs) to deploy their website securely using AWS infrastructure. 

## Project Requirements 
1. CI/CD Pipeline
2. Implement vulnerability/dependency scanning in CI Script 
3. Ensure proper authentication and authorization in each environment in CI Script 
4. Proper handling of CI/CD Pipeline secrets

## Branching strategy
![[branching strategy.drawio.png]]

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
-insert workflow diagram-

The GitHub action workflow would trigger upon a push to either `main` or `dev`.

- `synk-scan` - Perform [Synk](https://github.com/snyk/actions/tree/master/python-3.10)security scan on python app
- `bandit-scan` - Perform [Bandit](https://github.com/PyCQA/bandit) scan on python app to find common security issues
- `build-image-ecr` - Builds image and push it to ECR
- `trivy-vul-scan` - Perform [Trivy](https://github.com/aquasecurity/trivy-action) vulnerability scan on image stored in ECR
- `ecr-to-ecs` - Deploy image from ECR to ECS
- `create-pr-to-main` - Create automated pull request to main after successful deployment (only from `dev')

For the final job all jobs in the GitHub Actions workflow must pass successfully. This ensures that no unverified code enters `main`.

## Backend (Website)
-insert website link-

## Roadmap
- Implement dynamic application security test
- 

## Troubleshooting/Learning points: 
1. if existing an pull request exists, the job wouldn't create another pull request. 
2. `dev` branch lacks protection, might cause merge conflict if developers forget to pull before hand/did not make their own feature branch