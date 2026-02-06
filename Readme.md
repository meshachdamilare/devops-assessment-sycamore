# Sycamore DevOps Assessment
This repository contains solutions to the Sycamore DevOps assessment tasks.

### Interview Questions
- Answers to the interview questions are provided in the Google Doc below:
https://docs.google.com/document/d/1trRkwu5bmZLGsuqxtLiaUnKQ2FPK7ZoCVAc8yC5oQWs/edit?usp=sharing

### Task A: DevOps Troubleshooting (The 502 Mystery)
- The root cause analysis is provided in `DevOps Troubleshooting/RCA.md`
- A stable deployment manifest is provided in `DevOps Troubleshooting/fixed-manifest.yaml`

### Task B: Secure CI/CD Pipeline
- The GitHub Actions workflow is located at `.github/workflows/pipeline.yml`
- The pipeline builds a Docker image from the `Secure Pipeline/` directory
- Trivy is integrated as a security scanner and the pipeline fails on HIGH or CRITICAL vulnerabilities
- Only images that pass the security gate are pushed to GitHub Container Registry (GHCR)
- Observations made while running the pipeline are documented in `Secure Pipeline/Note.md`


### Architectural choices
- GitHub Actions was used for CI/CD because it integrates natively with the repository and GitHub Container Registry (GHCR), and it reduces external dependencies and secret management.
- Trivy was integrated as a security gate before pushing images to GHCR to prevent publishing images with HIGH or CRITICAL vulnerabilities.
- Images are scanned before being pushed to ensure only compliant artifacts reach the registry.
- The built-in `GITHUB_TOKEN` was used for authentication to GHCR to avoid managing long-lived credentials.
- The pipeline builds images from the `Secure Pipeline` directory and uses commit SHA–based tagging for traceability.
- For Kubernetes stability, resource limits and health probes were added to prevent uncontrolled memory usage and ensure reliable pod lifecycle management.
