# Security Documents OpenMRS Should Publish

**A review of security documentation practices across highly regarded open source projects, with recommendations for OpenMRS**

*Prepared July 2, 2026. This document is the rationale and roadmap for the contents of this repository — see the [README](README.md) for the current status of each planned document.*

---

## 1. Method

Reviewed the published security documentation of: **Kubernetes** (incl. CNCF TAG Security), **Node.js**, **Rust**, **Python/PSF/PyPI**, **Django**, **curl**, **Apache Software Foundation**, **Eclipse Foundation**, **Mozilla**, **GitLab**, the **Linux kernel**, and two healthcare peers — **DHIS2** and **OpenEMR**. Measured against **OpenSSF Best Practices Badge** criteria, the **OpenSSF OSPS Baseline** (Levels 1–3), and the **OpenSSF Scorecard** Security-Policy check. All exemplar URLs were fetched and verified during this review.

## 2. The de facto standard set

Six documents appear in *every* mature project reviewed. They are the floor, and they map directly to OpenSSF Baseline Levels 1–2:

1. A published **vulnerability reporting policy** with a private channel and stated scope
2. A **coordinated disclosure process with timelines** (ack SLA, embargo length, publication order)
3. A **severity classification rubric**
4. A **public advisories archive**, increasingly machine-readable
5. An explicit **"what is NOT a vulnerability"** scope statement (now universal — adopted specifically to control report noise)
6. A **named security team** with documented composition and process

Beyond the floor, the marks of the most mature projects: published threat models, self-hosted third-party audit reports, published internal incident-response runbooks, embargo/distributor lists with public membership criteria, supported-versions matrices, and machine-readable supply-chain artifacts (OSV feeds, SBOMs, signed releases).

## 3. Where OpenMRS stands today

The wiki policy layer is real — [Security 101](https://openmrs.atlassian.net/wiki/spaces/docs/pages/25477259/OpenMRS+Security+101+Policies+Vulnerability+Management) covers reporting, severity/SLA, and CVE workflow, and four new secure-development pages appeared March–June 2026. But the audit found serious gaps:

- **The public front door is broken.** openmrs-core's SECURITY.md is three lines; openmrs-esm-core and openmrs-sdk have none; openmrs.org has no security page at all. A researcher landing anywhere but the wiki finds nothing.
- **Documented process and observable practice diverge.** The May 2026 critical CVEs (including CVE-2026-40076, CVSS 9.4, public PoC per CISA) were never announced on Talk per the wiki's own Step 3.
- **Third parties own OpenMRS's CVE narrative.** ~30 CVEs since 2017, many filed by VulDB/MITRE without org review; four March 2025 CVEs were never acknowledged.
- **No threat model, no published audit report** (the Jan 2025 pentest exists only as advisory prose; the only full assessment is a 2016 HIPAA PDF), **no OpenSSF badge, no supported-versions security matrix**.
- **Three overlapping, partly orphaned hardening guides** on the wiki with no canonical one.
- **The cvss-scanning harness is invisible** — not linked from any community-facing security doc.

## 4. Recommended documents

Prioritized into three tiers. Each entry gives the exemplar to copy and why it matters for OpenMRS specifically.

### Tier 1 — The front door (highest leverage, lowest effort)

**1.1 Real SECURITY.md in every flagship repo** (openmrs-core, openmrs-esm-core, openmrs-sdk, or an org-level default in `openmrs/.github`)
Contents: private reporting channels (GitHub PVR + security@openmrs.org with PGP key), ack SLA, link to full policy, supported versions. Exemplar: [Node.js SECURITY.md](https://github.com/nodejs/node/blob/main/SECURITY.md); template: [OpenSSF templates](https://github.com/ossf/oss-vulnerability-guide/tree/main/templates/security_policies). The [Scorecard check](https://github.com/ossf/scorecard/blob/main/docs/checks.md#security-policy) scores exactly this. *This is the single cheapest fix with the biggest external-signal payoff.*

**1.2 A Trust Center page on openmrs.org** (`openmrs.org/security` + `/.well-known/security.txt`)
One navigable hub linking policy, supported versions, advisories, hall of fame, and hardening guidance. Exemplar: [DHIS2 Trust Center](https://dhis2.org/trust/) — OpenMRS's closest peer (donor-funded, ministry deployers, PHI-adjacent), and its health-context-justified embargo language ("many implementations cannot patch quickly") is directly reusable. security.txt exemplar: [curl](https://curl.se/.well-known/security.txt) (RFC 9116).

**1.3 Supported-versions and security-backport policy**
A table: which platform/O3 versions receive security fixes, for how long, and the release cadence deployers can plan around. Exemplars: [DHIS2](https://dhis2.org/trust/) (latest 3 majors, patch each), [GitLab maintenance policy](https://docs.gitlab.com/policy/maintenance/). This is the document ASF conspicuously lacks and that slow-upgrading health facilities need most. No exemplar reviewed serves deployers on unreliable connectivity with multi-year upgrade cycles — OpenMRS should say explicitly what happens to them.

**1.4 "What is NOT a vulnerability" scope statement + AI-report policy**
Pre-reject noise: documented-dangerous-by-design features, DoS thresholds, out-of-scope deployments. Add a formal policy on AI/LLM-generated reports — [Django's](https://docs.djangoproject.com/en/dev/internals/security/) (unverified LLM output closed without response) and [Python's](https://devguide.python.org/security/policy/) are the models, and curl's July 2026 intake pause from report-volume burnout is the cautionary tale. *Note the mirror-image obligation: OpenMRS's own cvss-scanning harness output must be human-triaged before it becomes reports or advisories, or the project violates the norm it's asking researchers to follow.*

### Tier 2 — Process integrity (makes the response repeatable)

**2.1 Security response runbook (committer-facing, numbered steps)**
Report → acknowledge → severity → CVE → private fix → advisory → announce → retrospective, with "work in private" rules (no public issues, sanitized commit messages). Exemplars: [ASF committers process](https://www.apache.org/security/committers.html) (the best template for making volunteer maintainers behave consistently), [Kubernetes security-release-process](https://github.com/kubernetes/committee-security-response/blob/main/security-release-process.md). The May 2026 advisories skipping Talk shows the current process isn't operationalized — a checklist is the fix.

**2.2 Security team charter**
Membership criteria, term/rotation, employer-diversity cap, confidentiality obligations, decision authority. Exemplars: [Kubernetes SRC](https://github.com/kubernetes/committee-security-response/blob/main/security-release-process.md) (max 10 members, ≤half per org), [Node.js security-wg governance](https://github.com/nodejs/security-wg/blob/main/GOVERNANCE.md). Security 101's "as of 2022-02" staffing references show why this needs to be a maintained charter, not prose.

**2.3 Severity rubric with OpenMRS-specific judgment layer**
CVSS 4.0 as input, not verdict — with an explicit PHI-confidentiality modifier (a "moderate" info-leak in a generic web app is severe in an EMR). Exemplars: [Kubernetes severity-ratings](https://github.com/kubernetes/committee-security-response/blob/main/severity-ratings.md), [curl's severity levels and anti-CVSS rationale](https://curl.se/dev/vuln-disclosure.html#severity-levels). Also revisit the current SLA table: 180 days for High in an EMR is out of step with peers (Django targets 90 days overall).

**2.4 Advisory authoring guide + canonical advisories index**
Template (affected versions, CVSS vector, workarounds, credits), where advisories are published, and in what order (GHSA → Talk → announce list). Exemplars: [curl's advisory guide](https://curl.se/dev/advisory.html), [Django's full historical security archive](https://docs.djangoproject.com/en/dev/releases/security/) — a single permanent page of every issue ever disclosed. This also closes the gap of third-party CVEs going unacknowledged: adopt a stated policy of reviewing/annotating every CVE filed against OpenMRS, even to dispute it.

**2.5 Pre-notification / trusted-implementer list governance**
Criteria for who gets embargoed advance notice (large implementers, ministries, hosting partners), obligations, and removal for leaks. Exemplar: [Kubernetes private-distributors-list](https://github.com/kubernetes/committee-security-response/blob/main/private-distributors-list.md) (8 criteria, embargo-break removal). For OpenMRS this matters more than for most projects: a public critical advisory races hostile actors to unpatched clinic servers holding PHI.

### Tier 3 — Maturity differentiators (where OpenMRS can lead its peer group)

**3.1 Shared-responsibility / compliance-posture statement (HIPAA, GDPR, data-protection laws)**
What the software provides (audit logging, RBAC, encryption support) vs. what deployers must do. Exemplar to improve on: [DHIS2 security considerations](https://docs.dhis2.org/en/implement/implementing-dhis2/security-considerations.html); cautionary counterexample: OpenEMR publishes nothing and cedes the topic to commercial vendors. **Every healthcare project reviewed does this weakly or not at all — the clearest open niche for OpenMRS to lead.** The 2016 HIPAA assessment PDF on Talk is a decade old and concluded confidentiality protection was weak; leaving it as the last word is itself a liability.

**3.2 Consolidated deployment hardening guide (one canonical doc)**
Merge the three overlapping wiki guides (Securing an Implementation / MBSS / Comprehensive Guide) into one versioned, maintained document; redirect the rest. Exemplars: [Kubernetes securing-a-cluster](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/), [Django deployment checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/) (enforced by tooling — an idea worth stealing for the SDK).

**3.3 Threat model**
Start lightweight: [Mozilla Rapid Risk Assessment](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html) (30–60 min per module) rather than a monolith — sustainable for volunteers, and it directly prioritizes which modules the cvss-scanning harness targets first. Grow toward the [CNCF self-assessment template](https://github.com/cncf/tag-security/blob/main/community/assessments/guide/self-assessment.md). Consider Node's pattern of *two* models: one for the product, one for [maintainer infrastructure/insider threat](https://github.com/nodejs/security-wg/blob/main/MAINTAINERS_THREAT_MODEL.md). This is OSPS Baseline Level 3 territory; no healthcare EMR peer has one.

**3.4 Published security assessment reports (turn the pentest + harness into artifacts)**
The Jan 2025 pentest deserves a methodology/scope/findings report, self-hosted like [curl's audits page](https://curl.se/docs/audits.html) or [Kubernetes' audit archive](https://github.com/kubernetes/sig-security/tree/main/sig-security-external-audit) (which publishes even the RFPs). The cvss-scanning harness should get a public page: what it scans, how findings are triaged, results cadence — GitLab's [reproducible-vulnerabilities page](https://handbook.gitlab.com/handbook/security/product-security/application-security/reproducible-vulnerabilities/) is the manual precursor to exactly this. This converts the NSF Safe-OSE investment into visible community assets.

**3.5 Incident response plan (beyond single-vuln handling)**
Playbooks for premature disclosure, zero-day exploitation in the wild, compromised committer account, malicious module in the module repository. Exemplars: [Node.js incident response plan](https://github.com/nodejs/security-wg/blob/main/INCIDENT_RESPONSE_PLAN.md) (a genuinely rare artifact), [GitLab's public PSIRT runbooks](https://handbook.gitlab.com/handbook/security/product-security/psirt/runbooks/unintended-vuln-disclosure/). Notably, *almost no project reviewed* publishes a committer-account-compromise procedure — a one-pager here exceeds peer practice.

**3.6 Release integrity and supply-chain documentation**
How to verify OpenMRS artifacts (signatures, checksums), SBOM availability per release, dependency-update policy. Exemplars: [curl's verify.html](https://curl.se/docs/verify.html) (the strongest single "why trust our releases" page anywhere), [kernel signature page](https://www.kernel.org/signature.html). Especially relevant given CVE-2026-40076 was a malicious-module-upload RCE — module distribution *is* OpenMRS's supply chain.

**3.7 Annual security transparency report**
Reports received, CVEs issued, median time-to-fix, harness statistics. Exemplar: [ASF annual security reports](https://news.apache.org/foundation/entry/apache-software-foundation-security-report2). Doubles as NSF Safe-OSE reporting material.

**3.8 OpenSSF Best Practices badge + security-insights.yml**
Not documents per se, but the external scoreboard: pursue the [Best Practices badge](https://www.bestpractices.dev) (passing level is achievable once Tier 1 lands) and publish [security-insights.yml](https://github.com/ossf/security-insights). Target [OSPS Baseline](https://baseline.openssf.org/) Level 2 explicitly, Level 3 (threat model, VEX) as the Safe-OSE horizon.

## 5. What to skip (deliberate non-recommendations)

- **Becoming a CNA.** Kubernetes, PSF, curl, and the kernel did it to control CVE quality, but it carries real ongoing obligations. At ~5 CVEs/year, GitHub's CNA via GHSA (the DHIS2/OpenEMR path) is sufficient — *if* paired with the Tier 2.4 policy of reviewing every third-party CVE.
- **A paid bug bounty.** curl killed its bounty in 2026 and paused intake from burnout; Node's ended with IBB. DHIS2's hall-of-fame model is the right fit and effectively free.
- **Foundation-wide CVSS worship.** Follow curl and Django: use CVSS as an input, publish a rubric with named severity classes and OpenMRS-specific reasoning.

## 6. Suggested sequencing

**Quarter 1:** Tier 1 complete (SECURITY.md everywhere, openmrs.org/security + security.txt, supported-versions table, scope statement). Also: retroactively publish Talk advisories for the May 2026 CVEs — the process must be seen to work before it's rewritten.
**Quarter 2–3:** Tier 2 (runbook, charter, rubric, advisory guide, pre-notification list). Apply for the OpenSSF badge.
**Quarter 3+:** Tier 3, sequenced by Safe-OSE deliverables — shared-responsibility statement and consolidated hardening guide first (deployer-facing, highest health-context value), then RRA-style threat models feeding the cvss-scanning harness's target list, then the transparency report.

---

## Appendix: exemplar quick-reference

| Document | Best exemplar |
|---|---|
| SECURITY.md | [Node.js](https://github.com/nodejs/node/blob/main/SECURITY.md) |
| Trust Center hub | [DHIS2](https://dhis2.org/trust/) |
| Disclosure policy (single doc) | [Django](https://docs.djangoproject.com/en/dev/internals/security/) |
| Committer response process | [ASF](https://www.apache.org/security/committers.html) |
| Security release process | [Kubernetes SRC](https://github.com/kubernetes/committee-security-response/blob/main/security-release-process.md) |
| Severity rubric | [Kubernetes](https://github.com/kubernetes/committee-security-response/blob/main/severity-ratings.md) / [curl](https://curl.se/dev/vuln-disclosure.html#severity-levels) |
| Distributor/embargo list | [Kubernetes](https://github.com/kubernetes/committee-security-response/blob/main/private-distributors-list.md) |
| Advisory archive | [Django](https://docs.djangoproject.com/en/dev/releases/security/) / [curl vuln.json](https://curl.se/docs/vuln.json) |
| Incident response plan | [Node.js](https://github.com/nodejs/security-wg/blob/main/INCIDENT_RESPONSE_PLAN.md) |
| Infra/insider threat model | [Node.js](https://github.com/nodejs/security-wg/blob/main/MAINTAINERS_THREAT_MODEL.md) |
| Lightweight threat modeling | [Mozilla RRA](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html) |
| Self-assessment template | [CNCF TAG Security](https://github.com/cncf/tag-security/blob/main/community/assessments/guide/self-assessment.md) |
| Release verification | [curl verify.html](https://curl.se/docs/verify.html) |
| Audit publication | [curl audits](https://curl.se/docs/audits.html) / [Kubernetes](https://github.com/kubernetes/sig-security/tree/main/sig-security-external-audit) |
| Hardening guide | [Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/) / [DHIS2](https://docs.dhis2.org/en/implement/implementing-dhis2/security-considerations.html) |
| Transparency report | [ASF](https://news.apache.org/foundation/entry/apache-software-foundation-security-report2) |
| Meta-standards | [OpenSSF Baseline](https://baseline.openssf.org/) / [Best Practices badge](https://www.bestpractices.dev/en/criteria/0) / [OpenSSF templates](https://github.com/ossf/oss-vulnerability-guide) |
