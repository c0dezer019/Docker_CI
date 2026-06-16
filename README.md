# Secure Docker Build

Builds a Docker Hardened Image (DHI) with SBOM generation and supply chain attestations baked in via Docker Buildx.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `docker_image` | ✅ | — | Docker image name (e.g. `user/repo`) |
| `docker_build_dir` | ❌ | `.` | Build context directory |

## Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: your-username/secure-docker-build@v1
        with:
          docker_image: myuser/myrepo
          docker_build_dir: ./server
```

> **Note:** This action only builds the image. Pushing to Docker Hub should be handled in a subsequent step using [`docker/login-action`](https://github.com/docker/login-action) and `docker push`.

## How Attestations Work

This action uses two Docker Buildx features during the build step:

**SBOM (`--sbom=true`)**
A Software Bill of Materials is a machine-readable inventory of every package and dependency inside your image — the OS packages, language runtimes, libraries, and their versions. It's generated automatically by Buildx using the [Syft](https://github.com/anchore/syft) scanner and attached to the image as an OCI artifact.

**Provenance (`--attest type=provenance,mode=max`)**
A provenance attestation records *how* the image was built — the source repo, the commit SHA, the build arguments, the Dockerfile used, and the build environment. `mode=max` captures the full build graph rather than a minimal summary.

Both are stored as signed attestations in the image manifest and are pushed alongside the image when you run `docker push`.

## Why Include Them

- **Auditability** — Anyone pulling your image can verify exactly what's in it and how it was built, without trusting your word for it.
- **Supply chain security** — Attestations are a core part of [SLSA](https://slsa.dev/) (Supply-chain Levels for Software Artifacts), a security framework increasingly required for enterprise and open source projects.
- **Vulnerability scanning** — Tools like Docker Scout, Grype, and Trivy can consume the SBOM to scan for known CVEs without having to re-analyze the image layers themselves.
- **Compliance** — US Executive Order 14028 and frameworks like FedRAMP now expect SBOMs for software used in federal systems. Getting in the habit early is low-cost and high-value.
