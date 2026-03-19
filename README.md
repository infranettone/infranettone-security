# infranettone-security

## Content index

- [1. Analyse the vulnerabilities of a Docker image](#1-analyse-the-vulnerabilities-of-a-docker-image)
  - [1.1 Bash Tools](#11-bash-tools)
    - [1.1.1 trivy](#111-trivy)
    - [1.1.2 grype](#112-grype)
    - [1.1.3 docker scout (Docker official)](#113-docker-scout-docker-official)
- [2. Docker images without vulnerabilities](#2-docker-images-without-vulnerabilities)
  - [2.1 distroless](#21-distroless)
  - [2.2 alpine](#22-alpine)
  - [2.3 scratch](#23-scratch)
  - [2.4 Multi-stage builds examples](#24-multi-stage-builds-examples)
    - [2.4.1 Node](#241-node)
    - [2.4.2 Go](#242-go)
- [3. In Kubernetes](#3-in-kubernetes)
  - [3.1 Root filesystem read-only](#31-root-filesystem-read-only)
  - [3.2 Run as non-root](#32-run-as-non-root)
  - [3.3 Capabilities](#33-capabilities)
  - [3.4 Don't allow privilege escalation](#34-dont-allow-privilege-escalation)
  - [3.5 Enable Linux Security Modules:](#35-enable-linux-security-modules)

## 1. Analyse the vulnerabilities of a Docker image

### 1.1 Bash Tools

#### 1.1.1 trivy

* sudo apt install trivy

* trivy image image-name:tag

  * Operating system CVEs (Common Vulnerabilities and Exposures)

  * Libraries CVEs

  * Vulnerable packages

  * Insecure configurations

* trivy image --severity CRITICAL,HIGH

#### 1.1.2 grype

  * Useful for Software Bill of Materials (SBOM)

  * brew install grype

  * grype image-name:tag

     * Scan tarballs

     * Scan filesystem

     * Scan SBOM

#### 1.1.3 docker scout (Docker official)

  * docker scout cves image-name:tag

    * Integration with Docker Hub

    * Compares vulnerable layers

    * Shows which base image you should update

## 2. Docker images without vulnerabilities

### 2.1 distroless

### 2.2 alpine

### 2.3 scratch

### 2.4 Multi-stage builds examples

#### 2.4.1 Node

```Dockerfile
# build stage
FROM node:18 AS builder

WORKDIR /app
COPY package*.json .
RUN npm install

COPY . .

# runtime stage
FROM gcr.io/distroless/nodejs18

WORKDIR /app
COPY --from=builder /app .

CMD ["server.js"]
```  

#### 2.4.2 Go

```Dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM scratch
COPY --from=builder /app/app /

CMD ["/app"]
```



## 3. In Kubernetes

### 3.1 Root filesystem read-only

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

### 3.2 Run as non-root

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

### 3.3 Capabilities

* Linux has ~40 capabilities that enable dangerous things. For example:
  * CAP_SYS_ADMIN
  * CAP_NET_ADMIN
  * CAP_SYS_PTRACE
```yaml
securityContext:
  capabilities:
    drop:
      - ALL
```

### 3.4 Don't allow privilege escalation
```yaml
securityContext:
  allowPrivilegeEscalation: false
```

### 3.5 Enable Linux Security Modules:

* seccomp + AppArmor:

```yaml
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
```

* eBPF runtime protection: It works with tools such as **Cilium** or **Falco** that monitor syscalls using eBPF:

```yaml
metadata:
  annotations:
    container.apparmor.security.beta.kubernetes.io/app: runtime/default
```
