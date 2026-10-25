# Terp

**A sovereign cloud platform for your own servers and data centers.**

Terp turns infrastructure you own into a cloud: install it on your own hardware and get an API-driven platform for running containerized applications, with gateways, networking, firewalls, mTLS, secrets and access control built in.

Terp is built in .NET by [Tremoot B.V.](https://tremoot.com) and builds on proven open source components.

> **Status: early development.** Terp is in its first phase (walking skeleton). It is not ready for production use, and the API will change.

## Why Terp

- **Sovereign.** Runs entirely on infrastructure you control, including air-gapped environments. No dependency on a public cloud.
- **Auditable.** The full source code is available, so you can verify exactly what runs in your data center.
- **API-first.** Everything the portal can do is available through the API. The portal, the `terp` CLI and the Terraform provider all use the same API.
- **Declarative.** Every resource has a desired state (`spec`) and an observed state (`status`); reconcilers keep the two in sync.
- **Secure by default.** mTLS on by default, secrets never stored or returned in plain text, and new identities start with zero permissions.
- **Container-first.** Applications run as OCI containers on Kubernetes, managed entirely through Terp. Users never need direct access to Kubernetes.

## Core concepts

| Concept | Meaning |
|---|---|
| **Organization** | Identity boundary: users, groups and service identities |
| **Space** | Billing and quota boundary, such as "production" or "test" |
| **Bundle** | A group of resources that live and are deleted together |
| **Resource** | A managed object, such as a config setting, certificate or app |
| **Grant** | An identity, a role and a scope |

## Roadmap

1. **Platform core:** resource engine, identity, organizations, spaces, bundles, roles and grants, audit log *(in progress)*
2. **Config**
3. **Secrets**
4. **Certificates**
5. **Network:** gateways, firewalls, mTLS
6. **Apps:** container workloads
7. **Database:** managed databases with multiple engines

## Getting started

> The commands below describe the intended developer workflow. The solution is still being set up.

Prerequisites:

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- Docker (or another OCI-compatible container runtime)

Start the full platform locally (PostgreSQL, Keycloak, the API and workers) with .NET Aspire:

```bash
dotnet run --project src/Terp.AppHost
```

Run the tests:

```bash
dotnet test
```

## License

Terp is **source-available, not open source.** See [LICENSE](LICENSE) for the full terms. In short:

- You may read and audit the source code.
- Testing, development and evaluation are **free for everyone**.
- **Production use requires a license from Tremoot.** Non-profit organizations can apply for a free production license.
- Forking, redistributing or offering Terp as a service is not allowed without written permission from Tremoot.

The API clients, the `terp` CLI and the Terraform provider will be released under the Apache License 2.0.

Contact Tremoot at [info@tremoot.com](mailto:info@tremoot.com) for production licenses.

## Contributing

Contributions are welcome from contributors who have signed the Tremoot Contributor License Agreement (CLA). Please open an issue to discuss larger changes before starting work.

## Security

Please do not report security vulnerabilities in public issues. Use GitHub's private vulnerability reporting for this repository instead.

## Trademarks

"Terp" and "Tremoot" are trademarks of Tremoot. The license does not grant permission to use them.

---

Copyright © 2026 Tremoot B.V. All rights reserved.
