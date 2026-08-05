# MVP implementation backlog — review draft

This is a private working draft, not a public submission artifact. It assumes a new public service in AWS `us-east-2`, deployed through GitHub and Terraform, using a hardened platform-managed foundation. The MVP does not require an LLM.

## 1. Services — Sign in and create a service

**Objective:** Let an authenticated developer begin in the portal by connecting a GitHub repository, supplying minimal ownership information, and creating or linking a provisional service record plus deployment request.

**User outcome:** A developer signs in with their organization identity and can begin a new-service deployment without first navigating separate catalog and request tools.

**In scope:** Enterprise SSO session; user and group claims; portal entry flow; repository reference; service and infrastructure owner; provisional catalog record; request ID.

**Out of scope:** Local passwords; identity-provider administration; repository creation; team-level opt-in; infrastructure generation.

**Implementation notes:**

- **Entry point:** Developer visits the portal and is redirected to the organization’s existing SSO provider when no session exists; an authenticated developer selects **Create service**.
- **Portal behavior:** Establish a signed-in session that displays the acting developer and uses the same identity in the catalog, audit trail, and downstream handoffs; then collect service name, GitHub repository, service owner, infrastructure owner, and intended production environment.
- **Assumed integration contract:** Enterprise SSO uses OIDC and returns stable user, email, and group claims; the existing service catalog either creates or links a provisional `serviceId`; `DeploymentRequests.create()` creates a portal-owned `deploymentRequestId` linked to that service.
- **State / exit:** Request moves from `draft` to `service-created` and opens the context-collection screen.
- **Failure handling:** An unauthenticated or expired session returns the developer to SSO; a duplicate repository, missing required owner, or catalog error leaves the draft intact and shows a retryable error.

**Dependencies:** None.

**Acceptance criteria:** A developer can create a request with a stable ID; the catalog remains the service system of record; missing required information is shown before the request proceeds.

**Open questions / risks:** What catalog API and minimum ownership fields already exist? Which SSO claims are reliable enough to establish the audit identity and display group membership?

## 2. Services — Collect service context

**Objective:** Gather the service catalog, approved repository, and deployment metadata needed for the supported path.

**User outcome:** The developer sees the trustworthy context used to prepare infrastructure.

**In scope:** Ownership, environment, delivery target, runtime, dependency, and current-deployment signals with field-level source attribution.

**Out of scope:** Unrestricted source-code ingestion; repository writes; silent reconciliation of conflicting inputs.

**Implementation notes:**

- **Entry point:** A request in `service-created` state opens the context-collection screen.
- **Portal behavior:** Show a read-only context summary with known, missing, and conflicting fields clearly marked.
- **Assumed integration contract:** `ServiceCatalog.getService(serviceId)`, `RepositoryContext.getSignals(repositoryId)`, and `DeploymentMetadata.getServiceState(serviceId)` return allowlisted metadata and source references.
- **State / exit:** Persist a versioned `contextSnapshotId`; move to `context-ready` when required reads complete.
- **Failure handling:** Missing data is displayed as unknown; an unavailable source creates a recoverable integration-error state rather than invented context.

**Dependencies:** #1.

**Acceptance criteria:** Each field identifies its source; unavailable and conflicting fields are explicit; only eligible services proceed to the next step.

**Open questions / risks:** Which repository/deployment signals are approved for collection and sufficiently reliable?

## 3. Services — Review and correct service context

**Objective:** Turn collected context into a versioned deployment profile that the developer can inspect and correct.

**User outcome:** The developer starts from known service facts rather than a long infrastructure intake form.

**In scope:** Profile view; developer corrections; field provenance; request revision history.

**Out of scope:** LLM inference; automatic acceptance of inferred values.

**Implementation notes:**

- **Entry point:** Developer opens the context screen after a `contextSnapshotId` is available.
- **Portal behavior:** Present each profile field with its value, source, and status; offer edits only where platform rules permit self-service correction.
- **Assumed integration contract:** `ProfileRules.getFieldPolicy(field)` returns whether a conflict is editable or review-required; `DeploymentRequests.createRevision()` stores corrections.
- **State / exit:** Save a `profileRevisionId`; move to `profile-confirmed` after the developer confirms required fields.
- **Failure handling:** Restricted edits remain visible but route to platform review; unsaved edits persist locally until the revision is saved.

**Dependencies:** #2.

**Acceptance criteria:** Developers can correct fields before proposal generation; self-service conflicts may be resolved by the developer; restricted conflicts route to platform review under platform-maintained rules.

**Open questions / risks:** Which fields are self-service versus review-required in the first policy configuration?

## 4. Deploy — Capture deployment requirements

**Objective:** Ask only for service requirements that approved context cannot resolve and that change the proposed infrastructure.

**User outcome:** The developer supplies business needs, not resource-by-resource configuration.

**In scope:** Expected traffic, data classification, availability/recovery, regionality/residency, and confirmation of detected production dependencies. Options and defaults come from the selected platform blueprint; developers can state an unsupported need in free text.

**Out of scope:** Generic chat; raw Terraform configuration; questions whose answer cannot change the outcome.

**Implementation notes:**

- **Entry point:** A confirmed profile enters the **Determine infrastructure** step in the portal.
- **Portal behavior:** Ask one unresolved, decision-changing requirement at a time; show why it matters and prefill known answers. Developers can select **Request a different setup** rather than forcing an unsuitable answer.
- **Assumed integration contract:** The selected platform blueprint provides a small versioned question/option configuration with defaults and escalation rules; no new generic options API is required for the MVP.
- **State / exit:** Each answer creates a request revision; move to `requirements-confirmed` once all supported required decisions are resolved.
- **Failure handling:** A deferred required answer blocks generation and offers platform review; an unsupported regionality or multi-region need is recorded for review.

**Dependencies:** #3.

**Acceptance criteria:** Known values suppress questions; every question identifies its decision impact; answers are versioned on the request; a multi-region or unsupported residency need is flagged for review.

**Open questions / risks:** Who owns the platform-maintained decision catalog and its allowed values?

## 5. Platform — Define the standard public-service foundation

**Objective:** Identify and version the existing approved Terraform blueprint used for a public containerized HTTP API with PostgreSQL in `us-east-2`.

**User outcome:** A developer receives a secure default path without needing to understand every infrastructure control.

**In scope:** One existing approved Terraform blueprint: managed public edge with TLS, WAF, and rate limiting; managed container runtime; private encrypted PostgreSQL; workload identity; standard telemetry; approved Terraform module versions.

**Out of scope:** Building a generic platform-configuration system; multi-region rollout; bespoke VPC design; public database access; arbitrary cloud services.

**Implementation notes:**

- **Entry point:** This is an existing platform-owned blueprint consumed by proposal matching; it has no developer-authored configuration screen in the MVP.
- **Portal behavior:** Expose the resulting components, defaults, and guardrails in plain language when a proposal is shown.
- **Assumed integration contract:** The existing approved blueprint exposes a version, Terraform module references, supported inputs, policy requirements, and reviewer rules.
- **State / exit:** A versioned `patternId` and `patternVersion` become part of the proposal record.
- **Failure handling:** An inactive or unavailable pattern prevents proposal generation and routes the request to platform review.

**Dependencies:** Platform-owned prerequisite; consumed by #6 and #9.

**Acceptance criteria:** The foundation maps confirmed requirements to versioned modules, defaults, policy requirements, and reviewer rules; concrete resource settings remain platform-owned.

**Open questions / risks:** Which existing Terraform modules meet the hardened baseline, and who approves changes to them?

## 6. Deploy — Select the approved deployment foundation

**Objective:** Deterministically confirm whether the service fits the one MVP foundation and return that blueprint’s configuration.

**User outcome:** The developer receives an approved proposal or a clear explanation of why the path does not fit.

**In scope:** Rule-based eligibility for the public API + PostgreSQL path in `us-east-2`; versioned match result and rationale.

**Out of scope:** LLM selection; multi-region plan generation; architecture design or choosing among arbitrary services.

**Implementation notes:**

- **Entry point:** The developer submits confirmed requirements from the determination step.
- **Portal behavior:** Display either the one approved foundation with rationale or a clear no-match result; do not show a speculative architecture.
- **Assumed integration contract:** `PatternMatcher.match(profileRevisionId, requirementsRevisionId)` evaluates platform-owned eligibility rules against the pattern catalog.
- **State / exit:** Persist a `proposalId`, selected pattern/version, and rule evidence; move to `proposal-ready` on a match or `review-required` on no-match.
- **Failure handling:** A matcher error keeps the request at `requirements-confirmed`; unsupported region, multi-region, or prohibited requirement produces a review-ready no-match reason.

**Dependencies:** #4 and #5.

**Acceptance criteria:** A qualifying request returns a traceable pattern and module set; an unsupported requirement does not produce a misleading partial proposal.

**Open questions / risks:** What minimum conditions make a service eligible for this first path?

## 7. Deploy — Review and change the deployment proposal

**Objective:** Let developers review the proposal, revise supported requirements, and regenerate the blueprint-derived change.

**User outcome:** Developers retain control without needing to edit raw infrastructure configuration.

**In scope:** Plain-language component summary; rationale; sources; approved settings surfaced as requirement changes; proposal regeneration.

**Out of scope:** Direct editing of Terraform, individual cloud resources, or the platform blueprint itself.

**Implementation notes:**

- **Entry point:** Developer opens a `proposal-ready` request in the **Infrastructure proposal** screen.
- **Portal behavior:** Show components, guardrails, sources, and blueprint version; developers can return to requirement answers, change permitted values, and regenerate the proposal.
- **Assumed integration contract:** `PatternCatalog.getParameterPolicy(patternVersion)` defines which requirement changes are permitted; `PatternMatcher.match()` regenerates the proposal after a material change.
- **State / exit:** Accepting the proposal creates an accepted `proposalRevisionId`; permitted requirement edits create a new proposal revision and invalidate older preflight results.
- **Failure handling:** A change outside the permitted policy opens the variation path rather than changing Terraform directly.

**Dependencies:** #6.

**Acceptance criteria:** A supported change creates a new proposal revision; the UI explains how the change affects the generated configuration; previous preflight results become stale when a material change occurs.

**Open questions / risks:** Which requirement changes should the first platform configuration permit without review?

## 8. Deploy — Request a nonstandard deployment

**Objective:** Preserve developer momentum when the supported foundation does not meet a requirement.

**User outcome:** The developer creates a context-rich GitHub issue instead of reconstructing the service context in another workflow.

**In scope:** Capture the required outcome and rationale; attach request context, source conflicts, proposal, and unsupported requirement; show review status.

**Out of scope:** Automatic policy exceptions; arbitrary IaC generation; partial multi-region deployment.

**Implementation notes:**

- **Entry point:** Developer selects **Request variation** from a no-match result or the proposal screen.
- **Portal behavior:** Capture a concise required outcome and rationale, then show the linked review status on the request.
- **Assumed integration contract:** `GitHubIssues.create()` creates a platform-review issue with request/profile/proposal references and a sanitized context summary.
- **State / exit:** Persist `platformReviewIssueNumber`; move the request to `review-required` and pause the normal Terraform PR handoff.
- **Failure handling:** A GitHub failure keeps the variation draft and provides retry; no unapproved Terraform is rendered or committed.

**Dependencies:** #6 and #7.

**Acceptance criteria:** Unsupported region, multi-region, bespoke network, or unsupported compliance requirement creates a linked GitHub platform-review issue; the normal PR path is paused until resolution.

**Open questions / risks:** Which platform team owns this issue queue and what response expectation is reasonable?

## 9. Deploy — Generate Terraform for the deployment

**Objective:** Render the Terraform files from the accepted approved blueprint using its exact module versions and confirmed request values.

**User outcome:** The developer gets a reproducible, reviewable infrastructure change rather than an opaque recommendation.

**In scope:** Versioned module manifest; Terraform rendering; isolated generated workspace; source-to-output provenance.

**Out of scope:** Direct cloud provisioning; GitHub branch creation; arbitrary Terraform generation or template editing.

**Implementation notes:**

- **Entry point:** Developer accepts a `proposal-ready` request with no unresolved variation.
- **Portal behavior:** Render the existing blueprint and return the developer to an inspectable review screen; rendering does not write to GitHub or cloud infrastructure.
- **Assumed integration contract:** The existing Terraform blueprint renderer accepts supported inputs and returns a `workspaceId`, generated-file manifest, and module provenance.
- **State / exit:** Store `workspaceId` and manifest; move to `change-generated` when rendering succeeds.
- **Failure handling:** A rendering failure preserves the accepted proposal and exposes retry or variation review; a generated workspace is isolated and disposable.

**Dependencies:** #5 and #7.

**Acceptance criteria:** Rendering is reproducible for a request revision; generated files identify their source modules; no repository or cloud write occurs during rendering.

**Open questions / risks:** What is the central Terraform repository layout and how are generated files organized by service?

## 10. Deploy — Review the generated Terraform

**Objective:** Present the generated Terraform, change summary, assumptions, and module versions for developer review.

**User outcome:** The developer can understand exactly what will be submitted for review.

**In scope:** File-level preview; plain-language summary; material assumptions; selected module versions; request provenance.

**Out of scope:** A full IDE or in-browser Terraform editor.

**Implementation notes:**

- **Entry point:** A `change-generated` request opens the **Pre-PR review** screen.
- **Portal behavior:** Show generated files, proposed modules, requirement-to-resource rationale, assumptions, and an action to return to the proposal.
- **Assumed integration contract:** `ChangeWorkspaces.get(workspaceId)` returns the generated-file manifest and safe preview content.
- **State / exit:** The developer either returns to proposal editing or initiates preflight validation against the displayed workspace version.
- **Failure handling:** Missing workspace artifacts prevent validation and expose a regenerate action; the preview never implies resources have been deployed.

**Dependencies:** #9.

**Acceptance criteria:** The preview identifies every affected file and the foundation version that produced it; the developer can return to change requirements before validation.

**Open questions / risks:** What preview detail is sufficient for trust without presenting a misleading deployment guarantee?

## 11. Deploy — Run deployment preflight checks

**Objective:** Invoke existing preflight checks against the generated change before the portal creates a draft PR.

**User outcome:** The developer sees security, policy, and cost feedback early while existing GitHub CI remains the merge/deploy authority.

**In scope:** Terraform format/validation/plan; approved-module checks; encryption, identity, network, tagging, observability, and cost checks; required-reviewer determination.

**Out of scope:** Building or waiving checks; policy authoring; replacing GitHub CI.

**Implementation notes:**

- **Entry point:** Developer selects **Run preflight** from the generated-change review screen.
- **Portal behavior:** Separate authoritative check results from explanatory guidance and show pass, fail, or inconclusive status per check.
- **Assumed integration contract:** Existing Terraform, policy, cost, and CI capabilities expose a preflight result with `validationRunId`, check outcomes, remediation references, and reviewer requirements.
- **State / exit:** Store `validationRunId`; a completed run enables draft PR creation with its status attached, while a changed workspace invalidates that run.
- **Failure handling:** Failed or inconclusive checks remain visible; the portal may create a draft PR only with status attached, while GitHub CI remains the mandatory merge/deploy gate.

**Dependencies:** #10.

**Acceptance criteria:** Results are tied to the generated workspace revision; pass/fail/inconclusive is explicit; material proposal changes invalidate prior results; GitHub CI repeats required controls after PR creation.

**Open questions / risks:** Which checks can run synchronously and which must be reported by GitHub CI?

## 12. Deploy — Create a draft infrastructure PR

**Objective:** Commit the generated change to the one configured central Terraform repository and open a draft PR.

**User outcome:** The developer receives a reviewable GitHub handoff without the portal approving, merging, or deploying infrastructure.

**In scope:** Scoped GitHub App access; branch and commit creation; draft PR; generated-change summary; validation status; service and platform reviewers for one configured delivery target.

**Out of scope:** Merge; deployment; per-service delivery routing; organization-wide mandatory PR gating.

**Implementation notes:**

- **Entry point:** Developer selects **Create draft PR** after preflight on a current generated workspace.
- **Portal behavior:** Create the handoff, show PR URL/number, validation status, requested reviewers, and explain that GitHub CI controls merge/deploy.
- **Assumed integration contract:** `DeliveryTarget.get(serviceId)` returns the central Terraform repository; `GitHubApp.createBranchCommitAndDraftPr()` returns branch, commit SHA, and PR metadata.
- **State / exit:** Persist branch, commit SHA, PR number/URL, and reviewer list; move to `pr-created`.
- **Failure handling:** Branch/PR errors preserve the request and generated workspace for retry; idempotency prevents duplicate commits or PRs.

**Dependencies:** #11.

**Acceptance criteria:** The PR records request, proposal, and validation versions; service and platform owners are requested by default, with additional reviewers determined by rules; GitHub CI remains authoritative for merge/deploy.

**Open questions / risks:** What code-owner and reviewer-routing conventions apply to the central Terraform repository?

## 13. Platform — Audit and recover deployment requests

**Objective:** Retain request status and an audit trail across portal, platform-review, preflight, and GitHub handoff actions.

**User outcome:** Developers can resume work after interruption, and operators can understand what happened without recording sensitive payloads.

**In scope:** Request state transitions; request/profile/proposal versions; actor and timestamp audit events; recovery and retry behavior.

**Out of scope:** Building a general workflow engine; analytics collection of source code, secrets, raw prompts, or raw policy payloads.

**Implementation notes:**

- **Entry point:** This capability is invoked by every request mutation and external handoff; developers see it through resume and status behavior.
- **Portal behavior:** Resume the request at the last completed state and show the current owner/status for a failed or review-required step.
- **Assumed integration contract:** `DeploymentRequests.transition()` enforces valid states and idempotency keys; `AuditEvents.append()` records actor, timestamp, action, version references, and result.
- **State / exit:** Every material action records a durable transition and audit event; no separate terminal state is required.
- **Failure handling:** Retriable external actions use their idempotency key; non-retriable failures preserve all preceding request revisions.

**Dependencies:** #1; used by #2–#12.

**Acceptance criteria:** A developer can resume a request at the last completed state; external writes are idempotent; material actions create queryable audit records.

**Open questions / risks:** Which audit retention and access requirements apply to deployment requests?

## 14. Platform — Enable early access for eligible services

**Objective:** Enable constrained early access for eligible new services and measure whether the secure path works before broader release.

**User outcome:** The organization can release the portal safely and decide what to expand next.

**In scope:** A constrained release rule for eligible new services; workflow timing, preflight outcomes, PR creation, platform-review requests, abandonment, and operational failures.

**Out of scope:** LLM capability; existing-service migration; multi-region rendering; mandatory organization-wide adoption.

**Implementation notes:**

- **Entry point:** Events are emitted by the MVP flow; the release control is evaluated when a new service begins a request.
- **Portal behavior:** Show only the developer-facing status required to complete or resume a request; operational reporting is for the product/platform team.
- **Assumed integration contract:** `Analytics.track()` accepts allowlisted event properties; feature access is determined by service eligibility rather than team-level opt-in.
- **State / exit:** Capture request-started, context-ready, requirements-confirmed, proposal-ready, preflight-completed, variation-requested, and PR-created events; release outcome is recorded after the constrained launch.
- **Failure handling:** Analytics failure must not block a deployment request; sensitive code, secrets, raw prompts, and raw policy payloads are never sent.

**Dependencies:** #1–#13.

**Acceptance criteria:** The team can measure secure reviewable PR creation, first-pass preflight outcome, help/variation rate, and integration reliability; broader rollout is blocked until the existing-service adoption path is defined.

**Open questions / risks:** How will existing services be catalog-linked or imported before wider release?
