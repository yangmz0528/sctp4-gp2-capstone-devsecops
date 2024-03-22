# (SCTP) Cloud Infrastructure Engineering Capstone Projection

## Overview:
**Case 3 - DevSecOps**

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
To prevent the exposure of sensitive information such as credentials and API keys in the codebase, it is important we store such data in the repository's secrets. This is to ensure they are encrypted at rest and prevent exposure as well as allowing the workflow to access the credentials multiple times without exposing the risk of exposing them in logs and other workflow files. 

``` yaml
# Job 2: Build, tag image and store it in ECR
build-image-ecr:
  runs-on: ubuntu-latest
  needs:
    - snyk-scan
    - sast-scan
  outputs:
    image_uri: ${{ steps.build-image.outputs.image_uri }}
  environment: ${{ github.ref_name }}
  steps:
    - name: Checkout
    uses: actions/checkout@v4
    - name: Configure AWS credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      aws-region: ${{ vars.AWS_REGION }}
```
## CI/CD Pipeline

![GitHub Workflow - Capstone Project](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/145353293/6e0e9844-0010-44f5-a226-97636cf5358e)

The GitHub action workflow would trigger upon a push to either `main` or `dev`.

- `snyk-scan` - Perform [Snyk](https://github.com/snyk/actions/tree/master/python-3.10) identifying and addressing vulnerabilities in dependencies and container images
- `bandit-scan` - Perform [Bandit](https://github.com/PyCQA/bandit) analyze Python code for security issues
- `build-image-ecr` - Builds image and push it to ECR
- `trivy-vul-scan` - Perform [Trivy](https://github.com/aquasecurity/trivy-action) vulnerability scan on image stored in ECR
- `ecr-to-ecs` - Deploy image from ECR to ECS
- `zap-scan` - Using [Zap Baseline](https://github.com/marketplace/actions/zap-baseline-scan) to find vulnerabilities in the web application after it has been deployed
- `create-pr-to-main` - Create automated pull request to main after successful deployment (only from `dev')

For the final job all jobs in the GitHub Actions workflow must pass successfully. This ensures that no unverified code enters `main`.

## Security Scans
There is a total of 4 security scans within the CI/CD pipeline, 
- Software Composition Analysis (SCA) - Snyk

Snyk is a developer-first cloud-native security tool that finds and automatically fix vulnerabilities in your code, open-source dependencies, containers, and infrastructure as code. Snyk uses a severity level system to classify the severity of vulnerabilities found in software dependencies. In the context of current use case, as the company is a start up and would want to focus on faster deployment and application development, we would set the severity threshold to be high. 

![image](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/108774198/14b9e5a5-007d-4c2a-99ca-62d72aaa6dc9)


- Static Application Security Testing (SAST) - Bandit

As N Repairs has highlighted specifically that they will be mainly developing the application in Python, we will use Bandit as it is a tool designed to find common security issues in Python code. We have set the threshold to be high confidence and high severity inn vulnerability.

![image](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/108774198/b39e1a5d-8859-4fa9-9090-7147b37fa75b)

- Image Scanning - Trivy
  
Trivy is an open-source vulnerability scanner designed specifically for container images. It helps developers and security teams identify vulnerabilities in container images by scanning their layers and providing detailed reports on any security issues found. Its fast scanning capabilities and easy integration with CI/CD pipelines make it a popular choice for ensuring the security of containerized environments.

![image](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/108774198/0f15ff92-8258-4817-a199-f9a0ad285913)

The above is a sample of result returned by Trivy. 

Some of the precautions and measure to take is to patch these vulnerabilities. Check if newer versions of the affected libraries or packages have been released with security fixes. Update your Docker images to use patched versions of the vulnerable dependencies.
However, there is also some libaries that does not have a fix (refer to screenshot below).

![image](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/assets/108774198/2deea4d0-9bc8-43b9-be4e-e2954de5213f)

- Dynamic Application Security Testing (DAST) - OWASP ZAP

OWASP ZAP (Zed Attack Proxy) is a widely used open-source web application security testing tool. It is designed to help developers and security professionals find security vulnerabilities in web applications during development and testing phases. ZAP offers a range of features including automated scanning, manual testing tools, and a proxy intercepting HTTP requests and responses to identify potential security flaws such as injection attacks, cross-site scripting (XSS), and broken authentication. With its user-friendly interface and extensive documentation, OWASP ZAP is a valuable tool for improving the security posture of web applications.

Refer to the results: [link](https://github.com/yangmz0528/sctp4-gp2-capstone-devsecops/issues/15)

The report highlights a few vulnerabilities such as absence of Anti-CSRF Tokens, Content Security Policy (CSP) Header Not Set etc. Based on the vulnerabilities reports, the security teams can identify the possible vulnerabilities that the website can be facing and do the necessary remediation actions to strengthen the security posture of the application, such as implementing Anti-CSRF Tokens to mitigate Cross-Site Request Forgery (CSRF) attacks, configuring Content Security Policy (CSP) headers to mitigate various types of attacks including XSS (Cross-Site Scripting), and conducting thorough code reviews and security assessments to identify and fix any other potential security weaknesses.

## Roadmap
- Implement automated deployment of AWS assets within the CI/CD pipeline

## Troubleshooting/Learning points: 
1. If an existing pull request exists, the `create-pr-to-main` job will not create a new pull request. 
2. `Dev` branch lacks protection which is likely to cause merge conflicts if developers fail to pull anytime before working on their independent feature/bugfixes.


