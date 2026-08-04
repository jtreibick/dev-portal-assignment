# Compass deployment workspace

[Repository](https://github.com/jtreibick/intuitive-assignment) · [Prototype](prototype/index.html) · [Backlog](https://github.com/jtreibick/intuitive-assignment/issues?q=is%3Aissue%20state%3Aopen%20sort%3Acreated-asc)

## Executive summary

AI is accelerating the path from an idea to working application code, but the path to compliant infrastructure is still slow and knowledge-intensive. Developers often have to assemble an answer from documentation, service catalogs, infrastructure-as-code (IaC) modules, policies, CI/CD systems, and conversations with platform engineers. Each source is useful; the coordination burden sits with the developer.

This proposal explores an agent-powered internal developer portal that guides a developer from a selected application repository to a reviewable infrastructure pull request (PR). The portal uses available application context to infer a structured profile, asks only the questions needed to fill important gaps, recommends an approved infrastructure pattern, explains the relevant guardrails, validates the proposed change through existing deterministic systems, and hands the change to the organization’s normal delivery workflow.

The AI helps a developer understand options and prepare a proposal. It does not become the authority for policy, approval, or execution. Existing internal developer portal (IDP), IaC, CI/CD, identity, and policy systems remain authoritative and are orchestrated rather than replaced. This is a focused hypothesis about making a common infrastructure request more self-service, reviewable, and trustworthy—not a claim to manage the full software delivery lifecycle.

## User and problem

The primary user hypothesis is an application developer who needs an approved infrastructure change but is not an infrastructure specialist. Their job is to move a service toward deployment without spending hours discovering which approved patterns, ownership boundaries, and controls apply. A secondary user hypothesis is a platform engineer who wants routine requests to be easier to complete while retaining clear governance and useful evidence when human help is needed.

Today, a developer may inspect a repository, find an IaC template, read policy guidance, ask in a chat channel, and then create or request a change. This fragmented journey creates cognitive load, uneven outcomes, and late discovery of constraints. These personas, needs, and baselines are hypotheses to validate through journey research and a limited pilot.

## Why now

Organizations have already invested in catalogs, reusable IaC modules, CI/CD pipelines, identity controls, policy engines, and operating guidance. At the same time, AI can summarize context, identify missing information, and explain choices in natural language. Combining those capabilities can reduce navigation and translation work without bypassing the systems that enforce controls.

Structured workflow events can reveal where requests stall, which approved patterns are missing, and when policy exceptions or platform assistance recur, subject to clear privacy limits.

## Proposed experience

The narrow end-to-end outcome is a developer selecting an application repository, confirming an inferred application profile, reviewing an approved infrastructure recommendation and its governance rationale, validating a proposed change, and creating a reviewable PR through the existing delivery workflow.

1. The developer selects a repository in the portal and starts an infrastructure request.
2. The workspace presents inferred facts—such as runtime, environment, network, availability, and data needs—and asks targeted questions only where confidence or policy-relevant context is missing.
3. The developer can correct the application profile and explicitly confirms it before the workflow proceeds.
4. The portal presents a golden-path recommendation, key components, tradeoffs, expected operational characteristics, applicable guardrails, and a credible alternative. The developer may accept, tune supported settings, request a custom variation, or reject it.
5. A review page shows the change summary and file-level preview. Deterministic policy validation is visibly separate from AI guidance and is run by the established validation system.
6. When validation is complete, the portal creates or prepares a PR in the existing workflow and explains what review or approval happens next.

The portal is the persistent workspace; embedded AI is contextual assistance within it. Direct manipulation remains available for confirming facts and making decisions.

## Product principles and boundaries

- **Developer control:** inferred facts and recommendations are editable; developers can tune supported settings, reject a path, or request a custom variation.
- **Explainable guidance:** recommendations make their inputs, rationale, guardrails, and alternatives visible.
- **Deterministic authority:** policy validation, approvals, and execution stay with the existing authoritative systems.
- **Orchestration over replacement:** the product connects existing IDP, IaC, CI/CD, identity, and policy capabilities instead of recreating them.
- **Reviewable outputs:** the end product is a clear infrastructure change and PR, not an opaque autonomous action.

## Assumptions to validate

We need to learn whether repository context is sufficient to build a useful first profile; whether targeted questions are genuinely fewer and clearer than the current research work; which application archetype has repeatable approved paths; and whether developers trust a recommendation when its rationale and validation status are visible. We also need to establish baseline time, completion, assistance, rework, and policy outcomes before asserting improvement.

## MVP scope and non-goals

The MVP focuses on one pilot persona, a small set of application archetypes, repository selection, profile confirmation, approved-path recommendation, deterministic validation, and PR handoff. It should capture decision and funnel evidence from day one.

It does not replace source control, CI/CD, policy engines, identity systems, cloud consoles, service catalogs, or platform review. It does not autonomously provision infrastructure, generate a production backend, support every repository type, or promise a general AI assistant for the entire SDLC.

## Risks and mitigations

Incorrect inference can lead to a poor recommendation; expose source facts, confidence gaps, and a correction path. Overreliance on AI can erode control; preserve explicit confirmation and keep deterministic controls authoritative. A generic experience can hide important exceptions; begin with a constrained archetype and approved pattern library. Sensitive context can create telemetry risk; minimize analytics and exclude source code, secrets, sensitive prompts, and raw policy payloads. Make the downstream review and approval path explicit.

## Success criteria

The first success criterion is learning whether the workflow helps a developer produce a compliant, reviewable infrastructure change faster and with less platform-team intervention, without reducing trust or control. The detailed metrics, event dictionary, funnel, and privacy boundaries are in [success-metrics.md](success-metrics.md). Numerical targets will be set only after a baseline or pilot provides evidence.

## Review package

- [Developer context](docs/mvp-delivery-context.md) — MVP assumptions, authority boundaries, and rollout context for the implementation backlog.
- [Success metrics](success-metrics.md) — P0 measurement and the day-one event contract.
- [Click-through prototype](prototype/index.html) — a self-contained, static workflow using sample data.
- [GitHub Issues](https://github.com/jtreibick/intuitive-assignment/issues?q=is%3Aissue%20state%3Aopen%20sort%3Acreated-asc) — an implementation-first MVP backlog.

### Explore the prototype

Open [`prototype/index.html`](prototype/index.html) directly in a modern browser. It has no framework, backend, network calls, or production integration. Start in **Your applications**, expand `checkout-api`, open its service overview, and choose **Prepare production deployment**. The production-deployment path is the only end-to-end workflow; the floating chat icon opens contextual Compass assistance.

This is a static click-through prototype, not production code or a technical proof of concept. It uses fictional/sample application and policy data, creates no real infrastructure, and ends at a mock pull-request handoff with checks pending.

AI was used as a drafting and prototyping aid. Product framing, scope boundaries, sequencing, and final review were directed to make the tradeoffs explicit and reviewable.
