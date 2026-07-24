# A Sustainable Public Security Advisory Archive for OpenMRS

**Status: DRAFT for Security Group discussion.** Prepared July 24, 2026. This is a strategy proposal. It argues for a specific architecture and lays out the month-to-month work it would create. This needs to be reviewed and revised by the security squad.

A short glossary of acronyms sits at the end (Section 8). I have tried to keep the body readable without it.

## 1. What we are trying to build, and why

The goal seems clear: a public, permanent record of every security issue found and fixed in OpenMRS, that a hospital IT lead, a researcher, or a scanner can trust, and that stays current without anyone remembering to update it.

Why it matters to us:  we are trying to evolve OpenMRS into a more proactive security posture as a community.  Medical record data and the patients we serve deserve that.  An archive can become the most visible, most checkable way to demonstrate that. Ours today is fragmented, partly outsourced to strangers, and likely out of date.

One principle drives the proposed design: the archive should not be a document someone maintains, it should be a byproduct of work we already do. Every archive that stays current works this way.

## 2. Where we currently stand

Intake works. A report reaches security@openmrs.org, forwards to the private Talk forum, the Security Group and the reporter deliberate, a fix and a CVE result. That pipe is fine.

The problem is the public record afterward, and it splits three ways:

1. **Reached us, published well.** The May 2026 module-upload flaw ([CVE-2026-40076](https://github.com/advisories/GHSA-78fc-9688-w8xw), CVSS 9.4, published from openmrs-core by ibacher) is the model: authored as a GitHub advisory, it automatically produced its CVE, a permanent page, a severity score, affected-version data, credit, and a machine-readable feed. We already do this.
2. **Reached us, recorded thinly.** We fixed it, but it never became a proper advisory, or it hit GitHub and never Talk. All three May 2026 criticals were published on GitHub and never announced on [Talk](https://talk.openmrs.org/tag/security-advisory), against our own process.
3. **Never reached us.** An outsider files a CVE without contacting us. [CVE-2025-25928](https://github.com/advisories/GHSA-7j7m-w4qx-7vmf) is typical: filed against the long-dead 2.4.3, no package, no version data, no fix, its only reference the [finder's personal CVE page](https://github.com/johnchd/CVEs/blob/main/OpenMRS/CVE-2025-25928%20-%20CSRF%20PrivEsc.md). It still carries our name.

One split should focus the discussion. Of the roughly thirty CVEs the world attributes to OpenMRS, only about six were authored or reviewed by us from what is visible online. The other two dozen were filed by outsiders seemingly without coordination, mostly against modules and long-dead versions, and we never adopted, annotated, or disputed them publicly. It's entirely possible that we handled some of these, but in short... we have no authoritative archive of our own. Outsiders are filling a vacuum we left open, and that, more than any stale-list worry, is the argument for what follows.

So: issues that reach us are nearly free to publish well; issues that do not, we cannot prevent but can detect. That drives the design.

## 3. The proposed architecture

Do not build an archive. Let GitHub be the engine, and build a thin layer that turns its raw data into something we own and curate.

GitHub already gives us, free and with no infrastructure: a CVE per advisory (GitHub is an authorized issuer), a permanent page, severity scoring, affected-version data, credit, a private embargoed workspace for the fix, and automatic export to the OSV machine-readable format the tooling world reads. That is most of a best-in-class system, and we are leaving much of it on the table.

What it does not give us is a curated, OpenMRS-owned view. The [github.com/advisories?query=openmrs](https://github.com/advisories?query=openmrs) page is a search, not our archive: we cannot brand it, annotate it, flag the suspected duplicate FHIR2 CVE, mark the dead-version findings as no current risk, dispute the junk, or link our Talk posts. It also blends our handful of reviewed advisories with roughly twenty-seven unreviewed outsider shells, with nothing telling a hospital which is which.

The architecture has three parts, and only the middle one is new work:

1. **Authoring stays on GitHub.** We keep writing our issues up as GitHub advisories, as we did in May 2026, and make that the default deliberation output, so "finish the write-up" and "get the CVE, page, credit and feed" become one action.
2. **A small generator builds the archive.** A scheduled job, a few dozen lines running free on GitHub's automation, pulls every OpenMRS advisory via the API, merges a short hand-kept file for outsider CVEs and our annotations, and produces three things: our branded archive page, an OSV feed, and the raw material for the Talk announcement.
3. **A watchtower catches what never reached us.** The same automation, pointed outward, scans GitHub and the [national database](https://nvd.nist.gov/) daily for new OpenMRS CVEs we did not originate and emails them to security@openmrs.org. It is the only defense for situation 3, and pretty easy to build.

The thesis: GitHub is the system of record, and a thin generator makes it the curated, OpenMRS-owned archive its raw search can never be.

## 4. Free versus build

| Capability | Source | Our effort |
|---|---|---|
| CVE, permanent page, severity, affected-version data, credit | GitHub | None |
| OSV machine-readable feed of our advisories | GitHub | None |
| Single curated OpenMRS archive page | The generator | Build once, then automatic |
| Annotations: duplicate, disputed, end-of-life, fixed-in | Hand-kept file | Minutes per outsider CVE |
| Talk announcement wiring | The generator | Build once |
| Detection of outsider CVEs (situation 3) | The watchtower | Build once, then triage |

The publishing is free or one-time. The only durable human cost is judgment: what an outsider CVE actually is, and whether to claim, enrich, or dispute it.

## 5. The real workload

Standing it up is not a lot of work, and most of it is not code: write the generator and watchtower, fix the broken SECURITY.md files so researchers find the front door, and review the two dozen or so unreviewed historical CVEs already sitting in the draft.

After that the recurring cost is small. Issues we find and fix need no new archive work, since the GitHub advisory produces everything. Each outsider CVE the watchtower surfaces costs a few minutes to annotate or dispute, a handful a year. The one new commitment is discipline, not time: that the Talk announcement actually goes out, which the generator makes nearly automatic. Triaging what outsiders file about us is the real tax of being a named project, and no tooling removes it, but the watchtower makes it cheap and timely rather than expensive and late.

## 6. What this does not do

It is for lookup, not paging. It helps a human or a scanner find what applies to a version; it does not reliably reach every hospital to say "patch now." That is harder and partly structural, since our platform, modules and frontend packages do not all map to the registries that drive automatic alerts. We should claim discoverability, not notification.

We are also skipping the heavier enterprise formats for now. CSAF and VEX are real and rising in health-IT and government procurement, and the top OpenSSF maturity level requires them, but they are more overhead than we can justify today. Keeping our data in OSV means we can add a VEX feed later without starting over.

## 7. Phasing, and what I need from the group

Three phases, each useful on its own. Phase 1: fix SECURITY.md, make authoring-as-GitHub-advisory the default, and backfill the archive. Phase 2: build the generator, publish the archive page and OSV feed. Phase 3: add the watchtower and wire in the Talk announcement.

Four things are the group's call, not mine: 

1. where the canonical page lives (openmrs.org, security-docs, or the wiki)
2. whether we require authoring-as-advisory and commit to claiming or disputing outsider CVEs
3. who owns the few-minutes-a-week triage, so it has a name and not just a hope
4. whether to backfill thoroughly now or incrementally as the watchtower surfaces old entries.

My bias, for what it is worth: Phase 1 is the highest-value and lowest-tech step, and a single honest, current archive page buys more credibility than it looks from inside the project. I am happy to build a working prototype against our real data so the group can see it rather than imagine it.

## 8. Glossary

- **CVE**: Common Vulnerabilities and Exposures. A globally unique ID for one security issue, like CVE-2026-40076.
- **CVE issuer (CNA)**: an organization authorized to hand out CVE numbers. GitHub is one, which is why authoring an advisory there yields a CVE for free.
- **GHSA / GitHub Security Advisory**: GitHub's built-in feature for privately writing up, fixing, and then publishing a security issue on a repository.
- **OSV**: [Open Source Vulnerabilities](https://ossf.github.io/osv-schema/). The standard machine-readable format for describing a vulnerability, endorsed by OpenSSF and read by most scanners. GitHub exports our advisories in it automatically.
- **OpenSSF**: [Open Source Security Foundation](https://openssf.org/). The industry body whose checklists ([Scorecard](https://github.com/ossf/scorecard), the [security baseline](https://baseline.openssf.org/)) define what a well-run open-source project should do.
- **CSAF / VEX**: heavier machine-readable advisory formats used by large enterprise vendors. Out of scope for now, designed for later.
- **Coordinated disclosure / embargo**: keeping an issue private until a fix is ready, so adopters can patch before attackers learn of it. We already do this.
