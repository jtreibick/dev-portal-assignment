# MVP delivery context

## Working premise

This MVP assumes Compass is an orchestration layer for a narrow developer workflow: create a secure, reviewable infrastructure pull request for a new public service. It is intended to bring together systems the organization already operates, not replace them.

For planning purposes, assume the initial delivery supports one hardened production foundation in AWS `us-east-2`:

- a public containerized HTTP API;
- a managed, encrypted PostgreSQL database with no public access;
- managed TLS, WAF, rate limiting, identity, logging, metrics, and tracing;
- approved Terraform modules;
- a draft GitHub pull request as the delivery handoff.

## Proposed developer flow

1. Sign in through enterprise SSO and create or connect a service in the portal.
2. The portal gathers trusted service-catalog, repository, and deployment context.
3. The developer confirms only unresolved service requirements.
4. Compass selects the assumed approved foundation, explains it, and lets the developer revise supported requirements.
5. Compass renders the assumed approved Terraform path, invokes existing preflight checks, and creates a draft PR in the configured central Terraform repository.
6. GitHub CI and human reviewers are assumed to remain responsible for merge and deployment.

## Working ownership and authority model

Use the following ownership assignment as a working assumption until the responsible teams validate it:

| System | Owns |
| --- | --- |
| Enterprise SSO | Developer identity and session claims |
| Service catalog | Service identity, ownership, lifecycle, and delivery target |
| Repository and deployment metadata | Runtime, dependencies, environments, and approved operational signals |
| Platform team | Approved Terraform blueprint, defaults, allowed changes, policy rules, and reviewer rules |
| Compass | Orchestration, request state, context display, proposal review, and handoff |
| Existing policy and CI/CD systems | Preflight, merge/deploy controls, and authoritative enforcement |
| Developer | Service requirements, supported changes, and decision to request a variation |

## Initial scope boundary

For the initial path, developers describe service requirements; they do not configure individual cloud resources or edit Terraform in the portal. The assumed platform-managed blueprint translates supported requirements into concrete configuration.

The initial path assumes one region and one standard foundation. Multi-region, nonstandard regionality/residency, custom network design, public database access, unsupported compliance requirements, or a new cloud service are expected to become a linked GitHub platform-review issue rather than produce a partial Terraform proposal.

## Future AI assistance

V1 has no LLM dependency: service context, requirements, and approved-path selection are deterministic.

After the workflow is proven, an assistant may help summarize trusted context, avoid redundant questions, explain results, and draft variation requests. It remains advisory; platform rules, policy checks, GitHub CI, and human reviewers retain authority.

## Proposed rollout

1. Exercise the workflow in an internal or sandbox environment.
2. Enable early access for eligible new services in `us-east-2`.
3. Define how existing services are catalog-linked or imported before broader release.
4. Add multi-region support and additional approved foundations after the initial path is proven.

## What would make this MVP worth expanding

The MVP is worth broadening if an eligible developer can create a secure, reviewable Terraform pull request through existing DevSecOps controls without requiring the portal to approve, merge, or provision infrastructure.
