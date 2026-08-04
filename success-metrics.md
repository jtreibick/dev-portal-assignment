# Day-one success metrics

## Learning question

Can the product help a developer produce a compliant, reviewable infrastructure change faster and with less platform-team intervention, without reducing trust or control?

All measures below are for a defined pilot cohort and application archetype. Establish a baseline or pilot distribution before setting numerical targets; this package deliberately does not invent improvement percentages.

## P0 metrics

| Metric | Definition | Why it matters |
| --- | --- | --- |
| Time to reviewable infrastructure change | Median elapsed time from repository selection to mock or real PR creation, reported with the distribution and cohort. | Tests whether the workflow reduces coordination time. |
| Workflow completion rate | Share of started workflows that reach PR creation. | Shows whether the full journey is usable. |
| Inference acceptance rate | Share of application profiles confirmed without edits. | Tests the usefulness and accuracy of inferred context. |
| Recommendation decision rate | Share of recommendations accepted, modified, or rejected; report each outcome separately. | Reveals whether recommendations lead to a decision, not just a view. |
| First-pass policy validation rate | Share of proposed changes that pass all required deterministic checks on the first run. | Indicates how well the workflow prepares compliant changes. |
| Platform-help request rate | Share of workflows with an escalation or assistance request before PR creation. | Tests whether routine work becomes more self-service. |

## Guardrails

These are diagnostic guardrails, not vanity metrics:

- Recommendation rejection and modification rate, segmented by stated reason.
- Policy failures by rule category.
- Workflow abandonment by last completed step.
- Developer-reported confidence and trust after handoff, using a short optional prompt.

## Compact event dictionary

| Event | Fires when | Essential properties |
| --- | --- | --- |
| `repository_selected` | A developer selects a repository to begin a workflow. | Anonymized `workflow_id`, anonymized `application_id`, timestamp, step, repository archetype. |
| `analysis_completed` | Inference has produced the application profile and clarification state. | `workflow_id`, `application_id`, timestamp, step, duration, unanswered-question count. |
| `application_profile_confirmed` | The developer confirms the profile. | `workflow_id`, `application_id`, timestamp, step, duration, edited flag. |
| `recommendation_decided` | The developer accepts, modifies, or rejects a recommendation. | `workflow_id`, `application_id`, timestamp, step, decision, duration, reason when supplied. |
| `policy_validation_completed` | Required deterministic checks finish. | `workflow_id`, `application_id`, timestamp, step, result, duration, failed-rule category count. |
| `pull_request_created` | The workflow creates or hands off a mock/real PR. | `workflow_id`, `application_id`, timestamp, step, duration, handoff type. |
| `workflow_abandoned` | A started workflow ends without PR creation after a defined inactivity window or explicit exit. | `workflow_id`, `application_id`, timestamp, last step, duration, reason when supplied. |
| `platform_help_requested` | The developer explicitly requests platform help or triggers an approved escalation. | `workflow_id`, `application_id`, timestamp, step, reason category. |

Use a consistent schema version and generate workflow and application identifiers that cannot be reversed into repository names. Where duration is included, record elapsed workflow time rather than detailed interaction traces.

## Privacy boundaries

Product analytics must not receive source code, secrets, prompts containing sensitive content, raw policy payloads, or personally identifying repository content. Collect only the minimum structured event properties needed for the learning question. Retention, access, and aggregation rules should be reviewed with the organization’s privacy and security owners before a pilot.

## Day-one dashboard and funnel

The initial dashboard follows this funnel:

`repository selected` → `analysis completed` → `profile confirmed` → `recommendation decided` → `policy validation completed` → `PR created`

Show counts and conversion at each stage, median time from selection to PR creation, recommendation decisions by outcome and reason, first-pass validation by rule category, help-request rate, and abandonment by step. Segment only by agreed pilot archetype or environment when the sample is large enough to protect anonymity and support interpretation.

Review the dashboard alongside lightweight qualitative evidence: developer confidence after handoff, correction reasons, and platform-team notes on escalations. The result should guide an expand, iterate, or stop decision rather than be used to claim a predetermined business impact.
