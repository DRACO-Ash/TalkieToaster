# Talkie Toaster - Flight plan

**Owner:** Ash Higgins, Technical Director, Bluestaq Limited
**Classification:** Unclassified (confirmed by the owner, 31 October 2026)
**Status:** Draft 4. Classification, resource ceiling, latency budget and provider egress confirmed. External model API adopted on owner decision, with a local-first extractive path retained for resilience. Remaining thin areas marked "TBC, re-verify".

## Vision and single job

● In one sentence: Talkie Toaster is a conversational, locally hosted custodian of the JCO Performance Work Statement (PWS) corpus and its supporting standardisation documents, which answers requirement questions out loud and in a written record, and holds who is on contract against which requirements.
● Single job: answer "what does the requirement actually say, and who is obliged to meet it" from the authoritative document set, with a citation, without a human searching the documents.
● Why now: the requirement knowledge is tribal and scattered across PWS, standardisation documents and contract records. NATO adoption is on the horizon and will roughly double the corpus, so the separation of US mission and NATO mission requirements has to be designed in rather than retrofitted.

## Users and experience

● Primary user: JCO staff who need a requirement answer fast, in the flow of work. Context: operational, time-poor, not document archivists. Pain today: knowing the answer exists somewhere in a PWS or a standardisation document and having to go and find it.
● Secondary users: those drafting new PWS constructs for new capabilities (draft proposals only, the authority writes the real thing), and those building an Initial Capability Test (ICT) for an incoming vendor.
● The one thing the interface must make effortless: asking a plain-language question and getting a trustworthy, cited answer, spoken and written, in a single move.
● Look and feel: fun, interactive and deliberately characterful. A Talkie Toaster persona with an animated sprite assistant in the spirit of the Office paperclip, built on the Hugo assistant skill in the repository (renamed Talkie). Exquisite front end, graphical presentation of answers where a chart beats a paragraph. Worthy of use inside JCO, which means the fun layer must never obscure the citation.
● Feel reference and "done looks like": TBC, re-verify (owner: Ash). Needed: one reference image or site, plus the acceptance line. Candidate acceptance: "a JCO staff officer shows it to a colleague unprompted, and the colleague can still see the source document reference without asking."

## Outcomes and success measures

● Transformation: from "I think the PWS says something about that, give me an hour" to "here is the threshold and the objective, here is the paragraph, here is which on-contract vendor it binds."
● Observable success measures:
  ● A fixed evaluation set of requirement questions with known-correct answers is answered correctly and with the correct citation, at or above an agreed pass rate (threshold TBC, re-verify, owner: Ash).
  ● Every answer carries a document, paragraph and threshold-or-objective reference; answers without a citation are refused rather than guessed.
  ● A new PWS dropped into the store is searchable without any manual indexing step.
  ● Vendor-to-requirement questions ("do our fusion providers have to meet a confidence level before verifying a post-manoeuvre state?") are answered in one turn.
  ● Answer returned within **10 seconds** end to end for a spoken question, measured at the 95th percentile on the deployed container, not on a workstation.
● Counter-measure: the rate of answers the system declines because the corpus does not support them. A system that never declines is fabricating.

## Scope and journeys

Core journeys, in order:
1. **Ask and answer.** Speak or type a question, get a spoken answer plus a written, cited record generated as a document.
2. **Ingest.** Drop a new PWS, standardisation document or amendment into the document store; it is parsed, chunked, indexed and available, with its corpus (US or NATO) and its own document marking recorded.
3. **Vendor register.** Record a vendor as on contract, against which contract and which PWS requirements, as structured data.
4. **Cross-query.** Ask across corpus and register together: which on-contract providers carry requirement X, and at threshold or objective.
5. **Draft a PWS construct.** Propose draft requirement language for a new capability, clearly marked as a Bluestaq or JCO draft proposal, never as authority text.
6. **Generate an ICT.** Produce an Initial Capability Test plan for a named vendor from the testing-process template in the repository.
7. **Corpus separation.** US mission and NATO mission PWS held apart, with deliberate cross-walk where requirements overlap.

Out of scope for the first version:
● Integration with the separate vendor market-scanning application (name TBC, re-verify, owner: Ash). Interface only, later.
● Multi-tenant or NATO-hosted deployment; the NATO corpus is supported, a NATO instance is not.
● Authoring final PWS text. The authority owns that; Talkie Toaster drafts proposals.
● Any write-back to a contracts system of record.

Smallest genuinely useful version: ingest the existing PWS set, ask a typed question, get a cited answer plus a generated document. Voice, sprite, vendor register and ICT generation layer on after that.

## Archetype and stack

● Archetype: **server**, because a model, a retrieval index and an ingest pipeline all run at runtime.
● Language and stack: Python. FastAPI plus a single-file SPA front end, matching the PSIRENS pattern that is already proven against the App Store quality gate. Document parsing for DOCX, PDF and PPTX. Local embedding model plus a local vector index; relational storage for the vendor and contract register.
● Model hosting: **external API, owner decision taken 31 October 2026**, reversing the earlier no-external-calls constraint. Assumption, stated and proceeding: Claude via the Anthropic API. The corpus is Unclassified, so this is permissible; the owner has accepted that unclassified but commercially sensitive content leaves the boundary.
● Envelope, no longer binding on the model: 4Gi ceiling, CPU only, no GPU. Tenant budget at time of writing showed 4352Mi of 8192Mi memory and 2200m of 6000m CPU committed. Without a local generative model the application fits comfortably; set the request nearer 1Gi and leave the rest of the tenant budget alone. Answer latency budget: 10 seconds, easily met.
● **Hybrid by design, local-first for the common case.** Retrieval, the governing paragraph and its citation run in-container against a small local embedding model and index. That path is sub-second, cannot fabricate, costs nothing per query, and **works with no network at all**. The external model is called only for synthesis, cross-corpus questions, drafted PWS constructs and ICT generation. Citations are rendered by the interface from structured data, never written by the model.
● Degraded mode is a first-class requirement, not an error state: with no egress the application still answers extractive requirement questions and says plainly that synthesis is unavailable.
● Voice: browser speech recognition and synthesis are now permissible. Local in-container engines remain an option if the degraded mode should also be spoken. Choice made at build time.
● Deployment target: Bluestaq App Store, container archetype (Dockerfile auto-detected), per CONTEXT-001 Section 7. Reads `PORT` and binds 0.0.0.0, runs as non-root uid 10001, `GET /`, `/healthz` and `/readyz` return 200 unauthenticated, resources set in the Configuration tab, two requirements files (`requirements.txt` carrying all test tooling, `requirements-runtime.txt` lean).
● App Store submission facts:
  ● Slug: `talkie-toaster` proposed, lowercase single-hyphen. Uniqueness in the store TBC, re-verify (owner: Ash).
  ● Display name: Talkie Toaster.
  ● Category and visibility: TBC, re-verify (owner: Ash).
  ● Resource envelope: ceiling **4Gi memory, CPU only, no GPU** (owner-confirmed). With no local generative model, request nearer 1Gi and measure. Set in the Configuration tab, not the Dockerfile.
  ● Environment variables: secret, the model provider API key, injected by the platform and never in the repository. Non-secret, the platform-injected `PORT`, the model identifier, and the storage path. Whether an encrypted secret is delivered differently from a plain-text variable, and whether saving one restarts the pod, is TBC, re-verify.
  ● Storage add-on: **required**. Document store, vector index and register must survive a restart. Add-on and volume write permissions for the non-root user TBC, re-verify (owner: Ash with the platform team).
  ● Rollback target: previous published build. Confirmed as available on the platform; specific target set at first publish.
  ● Runtime deployment contract items still to confirm: whether saving a configuration value restarts the pod, whether a signed-in identity is passed to the app and via which header, and the health-probe timeouts. TBC, re-verify.

## Structure

● **Ingest pipeline.** Watched directory on the persistent volume, then parse, chunk, embed, index. Records corpus (US or NATO), document type, version and the document's own marking as read from the document, never asserted by the application.
● **Retrieval and generation service.** Retrieval with a mandatory corpus filter. Extractive answers are returned directly with their reference; only synthesis and cross-corpus questions are passed to the generative model, constrained to a small retrieved context. Citations are returned as structured data alongside the prose in both paths.
● **Register.** Vendors, contracts, and the requirement identifiers each contract binds. Relational, queryable, editable through the interface.
● **Document generator.** Produces the written record for every answer, plus drafted PWS constructs and generated ICTs.
● **Interaction layer.** SPA, voice in and out, the Talkie sprite and persona, graphical answer rendering.
● Data and state live on the persistent volume (documents, index) and in the relational store (register, answer history). Nothing of consequence lives in the container filesystem.

## Data and integrations

● Data held: JCO PWS documents, JCO standardisation documents, the ICT testing-process template, vendor and contract records entered by users, and the answer history.
● Sources: the GitHub repository zip of PWS and requirement files, the Hugo assistant skill zip, the testing-process PowerPoint, and drag-and-drop additions by users thereafter.
● External integrations at runtime: **the model provider API** (assumed Anthropic), called only on the synthesis path. Retrieved document text and the user's question are sent; the vendor register is sent only when the question requires it. A future interface to the vendor market-scanning application is out of scope for version 1.
● Sensitivity: the work and the whole intended corpus are **Unclassified**, confirmed by the owner. Vendor contract status remains commercially sensitive in aggregate even though no document is marked. The application still displays each document's own marking as recorded, and never asserts a marking of its own, so that a marked document arriving later is handled correctly rather than silently mislabelled.

## Security and classification

● Worth defending: the aggregated view of what every vendor is contractually obliged to deliver. That aggregation is more sensitive than any single document.
● Realistic attackers, in order: a curious internal user seeing more than their role warrants; a vendor-supplied document carrying prompt-injection text that steers the model; accidental exposure through an over-permissive App Store visibility setting.
● **Prompt injection is a first-class threat here**, because the corpus is fed by dropped documents. Retrieved document text must be treated as data, never as instruction, and the generation prompt must be structured accordingly. This is a build requirement, not a nice-to-have.
● Secrets: the model provider API key. Lives in a platform environment variable, never in the repository, never logged, never returned to the client. The browser never calls the provider directly; all model calls are server-side.
● Egress: `api.anthropic.com` is reachable, confirmed by the owner, 31 October 2026. Re-verify on any platform network change, since an egress policy can tighten without notice. The local-first extractive path is retained regardless, because it also serves users on restricted networks and cuts per-query cost.
● Provider data handling: what the provider retains, and for how long, is an owner decision to confirm against the contract terms before the first production ingest. TBC, re-verify (owner: Ash).
● Sign-in: whether the App Store identity header is treated as a real boundary, and whether the register is role-restricted, is TBC, re-verify (owner: Ash). Until decided, assume every authenticated user sees everything and say so in the interface.
● Classification: **Unclassified**, confirmed by the owner. Not assumed.

## Code standards

● Build mode per the Principal Python Engineer project instruction: full toolchain (Ruff, Black, MyPy, Pyright, Bandit, pytest), typed throughout, structured logging, no pseudocode.
● Cognitive complexity capped at 15 per function (`python:S3776`), and the SonarQube local-proxy gaps listed in CONTEXT-001 Section 7 pre-empted by grep gates in `simulate-pipeline.sh`.
● Language and spelling: **UK English**, Bluestaq house style. No em-dashes. Bullets as ● and ■. Acronyms expanded on first use.
● Must never appear: a hardcoded credential, a client-side-only gate, or a fabricated requirement, citation or paragraph reference. An unsupported answer is refused, not approximated. This is the product's core integrity rule as well as a code rule.

## Quality and testing bar

● Engineering gate: pytest with a coverage floor of 80%, the five SonarQube conditions on overall code (violations 0, coverage at least 80%, security A, reliability A, security review A), `simulate-pipeline.sh` green before upload.
● **Retrieval and answer gate:** a fixed evaluation set of requirement questions with known-correct answers and citations, run as a test. A release that regresses the evaluation set does not ship. Set size and pass threshold TBC, re-verify (owner: Ash). Measured evidence over asserted acceptance criteria.
● Ingest gate: a known document set ingests, indexes and is retrievable end to end, including corpus separation (a US question must not return NATO text).
● Performance gate: 95th-percentile answer latency at or under 10 seconds on the deployed container, and resident memory within the configured request during an ingest. Measured, not asserted.
● Degraded-mode gate: with egress blocked, the extractive path still answers and the interface says so clearly. Tested, not assumed.
● Security gate: Bandit clean, image scan clean, prompt-injection test cases in the evaluation set.

## Constraints and ownership

● Deadline: TBC, re-verify (owner: Ash).
● Budget: TBC, re-verify (owner: Ash). Relevant mainly if the local model needs hardware beyond the standard App Store envelope.
● Authority over PWS content: JCO. The application drafts proposals; it does not speak for the authority.
● Decision owner for the build, scope and publish: Ash Higgins. Irreversible actions (publish, visibility setting) confirmed by the owner before execution.

## The one creative risk

The Talkie Toaster persona itself: a voiced, animated, deliberately entertaining character as the primary interface to contractual requirements in a defence organisation. It is worth it because adoption is the whole game; a dull search box gets used twice. It is kept from leaking by confining boldness to the interaction layer. The retrieval, citation, corpus-separation and register layers stay dull, verifiable and humourless. The persona may be charming about the answer; it may never be charming instead of the citation.

## Definition of done

● The local loop is green: tests pass, coverage at or above 80%, `simulate-pipeline.sh` clean.
● The SonarQube quality gate passes on all five conditions at first upload.
● The evaluation set passes at the agreed threshold, including corpus-separation and prompt-injection cases.
● Deployed to the Bluestaq App Store, healthy on `/`, `/healthz` and `/readyz`, with the storage add-on persisting documents, index and register across a restart (verified by restarting it).
● A user can drop in a new PWS and get a cited answer about it without any manual step.
● Rollback to the previous build is available and has been exercised once.
● Every remaining item below is closed or consciously carried.

## Open questions (TBC, re-verify)

● Confirm the exact model identifier. Provider settled: Claude via the Anthropic API, `api.anthropic.com` reachable from the container (owner-confirmed, 31 October 2026). (Owner: Ash.)
● Provider data-retention terms, confirmed before the first production ingest. (Owner: Ash.)
● Per-query cost ceiling and any monthly budget cap, and what the application does when it is reached. (Owner: Ash.)
● Which local embedding model and index for the offline extractive path. Settled by measurement at build time. (Owner: Ash.)
● Feel reference and the "done looks like" acceptance line for the Talkie persona. (Owner: Ash.)
● Is sign-in a real boundary, and is the vendor register role-restricted? (Owner: Ash.)
● Evaluation set size and pass threshold. (Owner: Ash.)
● App Store slug uniqueness, category, visibility, resource envelope, storage add-on and volume write permissions. (Owner: Ash, with the platform team.)
● Deadline and budget. (Owner: Ash.)
● Name and interface of the future vendor market-scanning application. (Owner: Ash.)
● Where the authoritative source of vendor and contract truth lives, and who keeps it current. (Owner: Ash.)
