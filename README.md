# infranettone-security

## Analyse the vulnerabilities of a Docker image

### Bash Tools

* trivy

  * sudo apt install trivy

  * trivy image image-name:tag

    * Operating system CVEs (Common Vulnerabilities and Exposures)

    * Libraries CVEs

    * Vulnerable packages

    * Insecure configurations

  * trivy image --severity CRITICAL,HIGH

* grype

  * Useful for Software Bill of Materials (SBOM)

  * brew install grype

  * grype image-name:tag

     * Scan tarballs

     * Scan filesystem

     * Scan SBOM

* docker scout (Docker official)

  * docker scout cves image-name:tag

    * Integration with Docker Hub

    * Compares vulnerable layers

    * Shows which base image you should update

## Docker images without vulnerabilities

* distroless

* alpine

* scratch

* Multi-stage builds examples

  * Node

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

  * Go

```Dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM scratch
COPY --from=builder /app/app /

CMD ["/app"]
```



## In Kubernetes

* Root filesystem read-only

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

* Run as non-root

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

* Capabilities

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

* Don't allow privilege escalation
```yaml
securityContext:
  allowPrivilegeEscalation: false
```

* Enable Linux Security Modules:

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