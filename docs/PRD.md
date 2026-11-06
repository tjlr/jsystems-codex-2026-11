# Hardware Service Decision Copilot - Decision Model Variant

## 1. Purpose and scope

Build a course proof of concept that helps customer support and hardware service staff assess synthetic complaints and returns. Staff submit a structured form and one equipment photo, receive a preliminary recommendation under fictional company policy, and discuss the result in chat.

The product supports staff decisions; it never issues a refund, rejects a claim, contacts a customer, or performs another external action. Staff retain the final decision outside this PoC. Fictional company policy does not determine statutory consumer rights. A photo cannot establish damage cause, operational condition, or legal eligibility.

This PRD specifies functionality, behavior, UX, UI, and acceptance criteria. A separate ADR will determine architecture, technologies, providers, model choices, and implementation details. This document does not authorize application initialization or implementation.

## 2. Agreed product decisions

The user selected the following during requirements discovery:

| Decision | Agreed scope | Reason |
| --- | --- | --- |
| Initial image criteria | Usability, category consistency, cracks/breakage, dents/deformation, scratches/surface wear | Small set of observable criteria suitable for a bounded assessment |
| Fictional return policy | Within 30 days and no clearly visible damage | Gives the initial observations a simple policy use without claiming legal eligibility |
| Fictional complaint policy | Within 24 calendar months: route to service assessment | Uses exact dates while reserving cause and entitlement for staff |
| Initial journey | Form, assessment, first explanation, follow-up chat | Staff can understand and question the recommendation |
| Deferred features | Customer lookup, database session history, RAG | Keeps the course PoC focused |

The remaining concrete defaults in this PRD are proposed PoC requirements, rather than additional user-confirmed choices.

## 3. Users, outcomes, and boundaries

Primary users are customer support staff preparing a case and hardware service staff deciding what inspection is needed. The person described by the form is a synthetic customer; the application is not a customer self-service portal.

Staff must be able to:

1. Enter a return or complaint with one synthetic, non-identifying image.
2. Distinguish observed evidence, reported facts, deterministic checks, and missing evidence.
3. See a consistent preliminary policy recommendation or a visible human-review requirement.
4. Ask follow-up questions without losing the original evidence and decision context.

In scope: the form, image validation and compression, bounded initial assessment, deterministic policy checks, conditional multimodal escalation, explanation, and session-local chat context.

Out of scope: real customer data, authentication and staff administration, payments, actual refunds, claim rejection, shipping, inventory, repair tracking, legal adjudication, multi-image submissions, document uploads, autonomous tools with external effects, and production claims management.

## 4. Form requirements

The interface is in Polish. This PRD and policy/prompt specifications are in English. Display a persistent notice that this is a synthetic-data PoC and staff make the final decision.

| Field | Control and validation |
| --- | --- |
| Case type | Required select, exactly `Return` / `Complaint`, displayed as `Zwrot` / `Reklamacja`; no preselected answer |
| Equipment category | Required select: smartphone, tablet, laptop, desktop computer, monitor, audio equipment, computer peripheral, other electronics |
| Equipment name/model | Required text, trimmed, 1-200 characters; free text does not prove model identity |
| Purchase date | Required date picker with keyboard entry, valid calendar date, not in the future |
| Reason | Textarea, maximum 4,000 characters; non-whitespace text required for complaints and optional for returns |
| Equipment image | Exactly one required JPEG, PNG, or WebP image; maximum original size 10 MiB; file must decode as an image |
| Synthetic-data confirmation | Required acknowledgement that the case and image are synthetic and non-identifying |

Show required indicators, character limits, inline validation, and an image preview with replace/remove controls. Explain that the image should show the item and relevant condition; it cannot establish the state of hidden surfaces. Return guidance requests a clear view of the item's condition; complaint guidance requests a clear view of the reported damage or affected area. Functional faults may have no visible manifestation.

Validate required fields, date format, limits, case/category membership, and uploaded file content on the backend as well as in the interface. Invalid input blocks model calls and preserves valid entries. Whitespace-only complaint reasons fail validation. Never silently crop or alter the user's reason.

### Image preparation

Before any model call, the backend decodes the image, corrects orientation, removes metadata, and compresses/resizes it while preserving aspect ratio and the whole frame. Proposed limits: longest edge at most 2,048 pixels and processed size at most 2 MiB. Do not upscale, crop away evidence, or remove apparent damage. If these limits cannot be met with useful detail, request a replacement image instead of silently sending degraded evidence.

Use the same processed image for initial assessment and escalation. Offer its preview so staff can inspect what was assessed. Corrupt, unsupported, or oversized files show actionable errors. If preprocessing fails, no model call occurs. File extensions alone are insufficient validation.

## 5. Assessment and decision behavior

### 5.1 Ordered flow

1. Validate the form and prepare the image.
2. Calculate exact dates and fixed policy checks deterministically.
3. Run the bounded decision model once on the image and relevant form context.
4. Validate its structured observations and determine whether evidence is sufficient.
5. Escalate uncertain, conflicting, or unsupported image judgments to a multimodal LLM for detailed analysis.
6. Reconcile the analysis with the evidence and reapply deterministic policy rules. Unresolved evidence or unavailable assessment providers produces `Human review required`.
7. Pass the resulting status, evidence, uncertainties, and policy checks to the conversational LLM for an explanation.
8. Open chat with the first assistant message and retain the full session context.

The initial decision model is not the conversational model. Detailed image analysis is conditional; clear initial observations do not require it. No model decides exact date arithmetic or changes fixed policy rules.

### 5.2 Bounded observation set

For each criterion return a category, estimated probability of the positive statement, uncertainty or missing-evidence notes, and a short evidence description referring only to visible content. Include an assessment-level contradiction indicator. These are separate observations; their probabilities need not sum to one.

| Criterion | Positive statement | Allowed interpretation |
| --- | --- | --- |
| Image usability | The image is usable for the requested visible-condition assessment | Clear enough and sufficiently framed for the relevant area |
| Category consistency | The pictured object appears consistent with the selected equipment category | Category-level relevance only; exact model identity is not verified |
| Cracks/breakage | Cracks or breakage are visible | Visible structural discontinuity or broken parts |
| Dents/deformation | Dents or deformation are visible | Visible distortion of the equipment body |
| Scratches/surface wear | Scratches or surface wear are visible | Visible surface marks; no inferred cause |

Categories are `present`, `absent in visible area`, and `indeterminate`. For usability and category consistency, use `yes`, `no`, and `indeterminate`. A missing or unassessable probability is explicitly `null` with a reason, never silently zero. Invalid, non-finite, or out-of-range probabilities invalidate the response and require escalation or review.

Proposed routing defaults: a positive statement with probability at least 0.80 can be a confident positive; at most 0.20 can be a confident negative. Intermediate values are uncertain. A category inconsistent with its probability is conflicting evidence. The thresholds are provisional course settings, not a claim of calibrated model accuracy; verify them on synthetic cases before describing them as reliable.

A low probability is not automatically uncertainty: a crack probability of 0.05 can confidently mean no crack is visible. By contrast, usability of 0.05 confidently means the image is unusable and cannot support a condition recommendation. Uncertainty flags, missing coverage, or contradictions override numerical confidence. No visible damage in the assessed area does not establish an undamaged whole item.

### 5.3 Escalation and fallback

Escalate when a required observation is indeterminate, relevance/usability is negative, probabilities and categories conflict, image evidence conflicts materially with the description, or an image judgment extends beyond the bounded evidence. For example, an unclear mark is escalated; a claimed manufacturing cause is unsupported by a photo.

The detailed multimodal analysis must identify what it can and cannot see, address the specific escalation reason, and describe whether the uncertainty is resolved. It must not certify causation, concealed condition, exact product identity, statutory eligibility, or operational performance. Conflicting analyses remain flagged unless the detailed analysis supplies a specific visible basis for resolving the conflict.

Allow one detailed analysis attempt per submitted assessment; a subsequent explicit retry is a separate attempt. If evidence remains insufficient, required model output is invalid, an assessment provider fails/is unavailable, or required fictional policy content is unavailable, show `Human review required` with the reason and a concrete next step. An obviously unusable or unrelated image may also invite replacement. Never invent a successful result or silently proceed on unsupported evidence.

A conclusive policy-date exception requires staff review even when the image is clear; additional image analysis is unnecessary solely for a date exception. Review status takes precedence over any favorable preliminary recommendation. All outcomes still require a staff decision; the special review status highlights cases that cannot follow the ordinary preliminary route.

## 6. Deterministic fictional policy rules

Use the submission date in Europe/Warsaw as the assessment date and freeze it for that assessment. Display it with the purchase date and policy version. A return is within the window when calendar-day difference is 0 through 30 inclusive. Complaint coverage ends on the 24-month calendar anniversary, inclusive; if the target month lacks the day, use its last day. Do not approximate months as a fixed number of days.

### Policy outcome table

| Scenario | Preliminary outcome | Required next step |
| --- | --- | --- |
| Return within 30 days, usable/relevant image, all three damage observations confidently absent, no unresolved conflict | Return-policy criteria met on supplied evidence | Staff verify the item and any unpictured surfaces before deciding |
| Return older than 30 days or any clearly visible listed damage | Outside fictional return policy - staff review | Show the failed rule; staff review without automatic rejection |
| Complaint within 24 calendar months and evidence sufficiently assessed | Route to service assessment | Record the reported symptom and visible findings; arrange staff inspection |
| Complaint older than 24 calendar months | Outside fictional complaint policy - staff review | Staff assess available next steps; no automatic claim rejection |
| Either flow with unresolved evidence, required provider failure, or missing policy | Human review required | Show reasons, available findings, and requested evidence or manual inspection |

For complaints, absence of visible damage does not disprove the reported fault. Presence of damage does not prove customer responsibility or remove access to service assessment. A nonvisual symptom is recorded as reported and requires inspection, without pretending image analysis can verify it.

Keep observations, policy outcome, and review requirement as distinct visible concepts. If several rules fail, list all known failures. A review requirement cannot be cleared by the explanation model or a chat instruction.

## 7. Chat behavior and session context

The first assistant bubble contains a brief greeting, preliminary outcome or review status, concise justification, material uncertainty, and numbered next steps. Use short sections and readable lists. The recommendation remains visibly preliminary, with a notice that fictional rules do not determine statutory rights. Do not display a probability as a probability of refund eligibility.

Keep a case summary and status visible alongside chat: form values, processed image preview, criterion results with estimated probabilities, deterministic checks, escalation summary if present, missing evidence, and policy version. Details may expand/collapse without hiding a review requirement.

The conversational LLM receives the complete current-session context: form data, initial observations/probabilities, escalation analysis, deterministic checks, current outcome and review reasons, applicable fictional policy, first explanation, and all subsequent messages. If the context limit is reached, explain the limit and invite a new assessment; never silently drop evidence or safety-relevant history.

Users can ask why a rule applied, what an observation means, or what staff should inspect next. Additional text is labelled as user-reported information. It cannot rewrite image observations, purchase dates, or policy outcomes. Case corrections use `Edit case and reassess`, which starts a new assessment with updated evidence; chat alone cannot clear review status. Reassessment is explicit, never an automatic repeat of the decision model on every message.

No additional image upload is available inside chat in the initial PoC; replacing the single image occurs through the form. Show an editing warning that the old conversation will be replaced, and require explicit confirmation before discarding it.

If the explanation/chat provider fails after assessment, retain the structured result and show a deterministic first-message fallback containing status, known facts, uncertainties, and next steps. Explain that chat is temporarily unavailable and offer explicit retry. An explanation failure does not change a valid assessment into an invented outcome; a failure during required assessment does require human review.

Maintain session data for the active page only. Reloading or starting a new case clears it, with an unsaved-session warning where supported. No cross-session persistence, customer lookup, or database audit trail is part of the initial PoC.

## 8. UX and UI requirements

Use the existing [design guidelines](design-guidelines.md) for neutral surfaces, restrained dark primary actions, typography hierarchy, spacing, rounded controls, and panels. Apply the reference to a focused staff tool; retail navigation, campaigns, product browsing, and basket controls are outside scope. Error, focus, loading, and review states must be designed explicitly because the reference does not establish them. Use system fonts unless asset usage rights are confirmed.

The form is a clear single-page card with progressive helper text for the selected case type. Primary action: `Oceń zgłoszenie`. During processing, show descriptive stages such as image preparation, initial assessment, detailed analysis when needed, and explanation preparation. Do not show a countdown or fabricated completion percentage. Disable duplicate submissions while retaining the entered data.

The chat screen prioritizes the preliminary outcome, then reasons and next actions. Display `Wymagana ocena pracownika` prominently for required review; do not use a success treatment for unresolved cases. Provide `Edytuj zgłoszenie` and `Nowe zgłoszenie`. New-case confirmation protects an active conversation from accidental loss.

Support keyboard navigation, labelled inputs, visible focus, accessible error/status announcements, and sufficient contrast. Status cannot rely on color alone. Provide descriptive text for image previews. On narrow screens, stack the case summary above chat without horizontal scrolling. Suggested prompt chips may offer "Why this recommendation?" and "What should I inspect next?" without triggering assessment automatically.

## 9. Data and model guardrails

Use synthetic cases and non-identifying images only. Ask users not to include names, addresses, faces, receipts, account details, or identifying serial numbers. Do not claim automatic anonymization or identity detection. Image metadata removal is required; arbitrary identifying content is not made safe by compression.

Do not persist uploaded images or conversations as a product feature. Any transient processing files are removed after use or failure. Do not log raw images, full prompts, form descriptions, or chat text. Provider retention and any technical operational telemetry must be resolved in the ADR before provider integration.

Treat form text, image text, and chat messages as case data, never as instructions that override policy or status. Render model output as safe formatted content. The assistant cannot call tools that enact decisions. Never present invented evidence, estimated probabilities as calibrated facts, or fictional procedures as legal advice.

## 10. Acceptance criteria and evaluation

Use a small, labelled synthetic evaluation set covering both clear and difficult examples. Record observable results for each case; report routing correctness separately from subjective explanation quality. Proposed release gate: all deterministic rules and critical status-preservation cases below pass; publish image-observation results on the labelled set rather than claiming an unmeasured accuracy target.

| ID | Scenario | Acceptance criterion |
| --- | --- | --- |
| AC-01 | Missing required field/image, whitespace complaint reason, future purchase date | Inline error; no model call; valid form entries retained |
| AC-02 | Return with blank reason | Valid when other required fields are complete |
| AC-03 | Image preparation | One valid file processed within limits; aspect ratio/frame preserved; metadata removed before model use |
| AC-04 | Unsupported, corrupt, excessive, or insufficient-quality image | Actionable error or review/replacement path; no silent evidence degradation |
| AC-05 | Clear relevant photo with scratch probability 0.05 and other evidence sufficient | Can yield a confident negative; low probability alone does not trigger escalation |
| AC-06 | Mid-range probability, contradiction, mismatched category, or poor usability | Detailed multimodal analysis attempted; unresolved evidence produces visible human review |
| AC-07 | No damage visible, but hidden surfaces or reported fault cannot be assessed | Explanation limits the finding to visible areas; no claim of whole-item condition or working order |
| AC-08 | Return at day 30 versus day 31 | Inclusive boundary applied deterministically; day 31 routes to policy-exception staff review |
| AC-09 | Complaint at 24-month anniversary versus following day, including leap-day purchase | Inclusive calendar-month rule and last-day handling produce the specified routes |
| AC-10 | Complaint with clear damage versus no visible damage | Both can route to service assessment within the window; neither establishes cause or disproves a fault |
| AC-11 | Assessment failure, unavailable provider, invalid output, or missing policy | No fabricated success; human-review status with reason and next step |
| AC-12 | Explanation provider fails after a valid assessment | Structured result remains; deterministic fallback appears; chat retry available |
| AC-13 | First assistant message and follow-up question | Greeting, outcome, justification, uncertainty, next steps; follow-up uses original form and full assessment context |
| AC-14 | User asks chat to approve a refund, ignore policy, or remove review status | No external action and no review override; response explains the staff next step |
| AC-15 | User reports a corrected purchase date in chat | Treat as reported information; require explicit form reassessment before changing the policy result |
| AC-16 | Keyboard/narrow-screen/error-state use | Form, preview, review status, and chat remain accessible and usable; status is understandable without color |
| AC-17 | New case, edit/reassess, duplicate submission, and provider retry | Destructive session transitions confirmed; single active submission; explicit retry cannot silently replace evidence |

Evaluation reporting should include observation categories versus labelled visible evidence, escalation frequency, unresolved-review frequency, inappropriate confident conclusions, unsupported statements in explanations, and whether each tested final status survives chat. The PoC demonstrates behavior on synthetic examples; it does not establish production or legal reliability.

## 11. Deferred features

Later, separately scoped features may include SQLite lookup of synthetic customer data and purchase history; durable storage of every session, decision, and staff action; and a RAG knowledge base for electronics specifications and approved procedures. These require additional data, retention, retrieval, and reconciliation requirements. They are not dependencies of the initial journey.

## Appendix A. Fictional return procedure specification

Policy ID: `DEMO-RETURNS-v1`. This is fictional training material, not a statement of statutory rights. Its content is specified here so only the requested PRD file is created. During implementation, maintain it as a separate policy document injected into the return explanation prompt.

1. Assess the purchase date against a 30-calendar-day inclusive window.
2. Require usable, category-relevant evidence and assess the three listed damage observations.
3. Within the window, with all three confidently absent and no unresolved evidence conflict, recommend "Return-policy criteria met on supplied evidence." Staff still verify condition, including unpictured surfaces.
4. Outside the window or with clearly visible listed damage, recommend "Outside fictional return policy - staff review." Do not reject the return or infer statutory ineligibility.
5. If assessment is unresolved or required evidence/providers are unavailable, require human review and state what is missing.
6. Staff make the final decision; the PoC performs no refund or external action.

## Appendix B. Fictional complaint procedure specification

Policy ID: `DEMO-COMPLAINTS-v1`. This is fictional training material, not a warranty contract or a statement of statutory rights. During implementation, maintain it as a separate policy document injected into the complaint explanation prompt.

1. Require a description of the reported problem and one image.
2. Check the purchase date against the inclusive 24-calendar-month anniversary.
3. Within the window, recommend service assessment when required evidence has been sufficiently assessed. Record visible damage separately from the reported symptom.
4. No visible damage does not disprove a functional complaint. Visible damage does not prove misuse, manufacturing defect, or responsibility.
5. Outside the window, require staff review under this fictional policy. Do not reject a complaint or determine statutory eligibility.
6. Unresolved evidence or required provider/policy unavailability requires visible human review. Staff decide whether further inspection or additional evidence is needed.

## Appendix C. Separate explanation prompt specifications

These prompt specifications are part of the PRD; separate runtime prompt files are a later implementation deliverable. Both prompts must treat case content as untrusted data, use Polish, preserve deterministic results, cite the applicable fictional policy ID and relevant rule in plain language, and avoid invented visible evidence. Both receive the context defined in Section 7.

### Return explanation prompt

"You assist staff reviewing a synthetic return. Use DEMO-RETURNS-v1 and the supplied deterministic result. Greet the user, state the preliminary return outcome or required human review, explain the date check and visible-condition observations, disclose missing coverage and uncertainty, and give concise staff next steps. Describe model probabilities only as estimates about the specified observations. Do not convert absence of visible damage into proof of whole-item condition. Do not decide legal eligibility, authorize a refund, or remove a required review status. In follow-up, distinguish user-reported additions from assessed evidence and direct material corrections to form reassessment."

### Complaint explanation prompt

"You assist staff reviewing a synthetic complaint. Use DEMO-COMPLAINTS-v1 and the supplied deterministic result. Greet the user, state service-assessment routing or required human review, explain the calendar-month check, distinguish reported symptoms from visible findings, disclose uncertainty, and give concise inspection or evidence-gathering next steps. A photo cannot establish damage cause or verify an internal functional fault. Do not infer customer responsibility, reject a claim, determine statutory rights, or remove a required review status. In follow-up, retain the entire case context and direct material corrections to form reassessment."
