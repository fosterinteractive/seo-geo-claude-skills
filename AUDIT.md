# AUDIT.md — Foster Interactive fork of `seo-geo-claude-skills`

**This file records the security audits performed on this fork. Keep it in
the repo root. Re-audit and append a new dated entry whenever the pinned
version is bumped or upstream changes are merged.**

---

## Checked — 2026-10-31 (no bump)

- **Pin:** unchanged, tag `audited-2026-07-18` = `f594142` (upstream v9.9.12 plus FI's audit record).
- **Upstream now:** `main` at `3a1578a` ("sync: umbrella v20.1.0"), 8 commits since the July check.
  - `main` is now a signpost. The standalone code is gone from it (177 files changed, about 19,000 lines removed),
    and SKILL.md tombstones point to the old skill paths.
  - Development continues in the successor repository `aaron-marketing-skills`. The standalone line stays frozen at
    tag v9.9.12.
- **Decision:** no material change to adopt. Nothing in the delta replaces or fixes code in the pinned line. The
  successor repository is a different product and was not reviewed; adopting it would be a new intake.
- **Staleness:** the frozen line gets no further fixes or guidance updates, so its SEO and GEO guidance drifts from
  Google's current documentation. Expected stale by about Jan 2027. **Re-audit or retire by 2027-01-31.**
- **Recommendation:** check whether the three skills FI uses from this line (`meta-tags-optimizer`,
  `schema-markup-generator`, `geo-content-optimizer`) are covered by `claude-seo` 2.5.0 (`seo-schema`, `seo-geo`,
  pinned `audited-2026-10-31`). If they are, retire this pin instead of re-auditing it.
- **Checked by:** Nick (Foster Interactive), with Claude Code.

---

## Formalization & pin verification — 2026-07-18 (no bump)

- **What this entry is:** the 2026-06-30 audit below was originally recorded
  **only in the message of annotated tag `audited-2026-06-30`** — a
  non-conformance with the fork-and-pin workflow, which requires `AUDIT.md`
  at the fork root. This entry formalizes that record into this file,
  re-verifies its findings against the pinned commit, and refreshes the
  staleness assessment. **The pin's content baseline is unchanged**: commit
  `1608176f6c18de6aec62a9abf6a2074bf82c9f67` (v9.9.12). Tag
  `audited-2026-07-18` marks the commit that adds this file on top of it.
- **By:** Aidan Foster (Foster Interactive), performed with Claude Code
  (Fable 5), live-testing `skill-audit-manage-skills` Mode B.

### Verification performed (spot-check, not a new full audit)

- **Pin integrity:** annotated tag `audited-2026-06-30` (tag object
  `e65c92da98d5501ae4cf70603f28f16308e60d44`) resolves to commit
  `1608176f6c18de6aec62a9abf6a2074bf82c9f67`, which is identical to release
  tag `v9.9.12` and to the fork's `main` HEAD before this commit. Content
  matches the v9.9.12 layout (20 skills, `scripts/connectors/`, `hooks/`,
  `build/`).
- **Finding 1 — private-IP SSRF (re-verified, still accurate):**
  `scripts/connectors/_http.py:37` states "Private/loopback IPs are
  intentionally NOT blocked so users can audit staging hosts." The
  `ALLOWED_SCHEMES` allowlist (`_http.py:38`) blocks `file://`/`ftp://` on
  requested URLs; private/loopback targets remain reachable by design. Same
  LOW characterization as recorded.
- **Finding 2 — unofficial Google autocomplete endpoint (re-verified, still
  accurate):** `scripts/connectors/suggest.py:39` hardcodes
  `https://suggestqueries.google.com/complete/search`; a WARNING about the
  unofficial/undocumented endpoint is in the docstring and emitted to stderr
  at runtime (`suggest.py:41-44, 145`). It is the only unofficial host in the
  connectors' egress set; all others are documented public APIs
  (web.archive.org CDX, Google PageSpeed API, Wikidata, Open PageRank,
  Google News RSS, Wikidata reconciliation on wmcloud.org).
- **Finding 3 — non-defused stdlib XML (re-verified, still accurate):**
  `scripts/connectors/sitemap.py:35` and
  `scripts/connectors/rss_monitor.py:30` use `xml.etree.ElementTree`;
  `defusedxml` is absent by design (repo policy: stdlib only, no third-party
  deps). Exposure is DoS-class from hostile XML, bounded by the 5 MB response
  cap in `_http.py`. Same LOW characterization as recorded.
- **Nothing-worse sweep:** no `subprocess`, `os.system`, `eval`, `exec`,
  `__import__`, `base64` blobs, or `pickle` in `scripts/` or `hooks/`. The
  SessionStart/PostToolUse hook (`hooks/claude-hook.sh`, 44 lines) matches
  the tag's characterization: symlink rejection and path confinement on every
  file it reads, a directive-redaction sanitizer on injected content, no
  network calls, no writes. **The spot-check confirms the recorded LOW
  verdict; nothing contradicting it was found.**

### Upstream freeze & history rewrite

- Upstream `aaron-he-zhu/seo-geo-claude-skills` **froze on 2026-07-05**
  (commits `82625fa` "docs: freeze notice" and `16920b7` "convert to
  signpost", both 2026-07-05): the repo is now a signpost stating the
  standalone 20-skill line is preserved at tag v9.9.12 and development
  continues in successor repo **`aaron-he-zhu/aaron-marketing-skills`**
  (120 skills across 7 disciplines, Apache-2.0, actively pushed as of
  2026-07-18). **Correction (2026-07-18, same-day re-check):** the freeze
  applies to *releases*, not the branch. After the freeze notice, upstream
  `main` kept receiving "sync: umbrella vN" commits mirroring the successor
  project's content: as of 2026-07-18 it is 7 commits ahead of the audited
  baseline `1608176f`, touching ~180 files — including the three FI-used
  skills under `build/`, every `scripts/connectors/*.py`, and `hooks/` —
  with **no new tag or release** (v9.9.12 remains the only one, ever). An
  earlier draft of this entry called the post-freeze delta "docs only";
  a file-level compare disproved that. Decision unchanged: **no pin bump** —
  adopting `main` would be an unplanned intake of the successor project's
  restructured content, not a maintenance update of the audited v9.9.12
  line; it belongs to the successor-repo intake decision (below).
- The tag's history-rewrite warning verifies: the originally-claimed pin
  `5643f5a` / `audited-2026-05-04` does not exist in upstream history
  (`git cat-file -e 5643f5a^{commit}` fails after fetching upstream), and
  upstream retains exactly **one** tag (v9.9.12) — earlier tags were wiped.
  This is precisely the upstream-hygiene failure the fork-and-pin invariant
  guards against: the fork, not upstream, is the durable record.

### Release cadence & staleness

- **Upstream release history:** effectively unmeasurable — the history
  rewrite left a single surviving tag/release (v9.9.12, 2026-06-28), and
  upstream froze 2026-07-05. `release_cadence.py` on the fork reports 2 tags
  (v9.9.12 + the audit tag), span 2 days: no meaningful cadence signal.
- **Domain velocity:** practice-based (AEO/SEO guidance; quarters, not
  weeks) — but per the staleness guidance, a frozen upstream overrides the
  class: this pin will never receive upstream fixes or improvements.
  Staleness risk is therefore "upstream dead; plan a replacement," not
  "re-audit later."
- **Expected to go stale by:** no calendar-driven rot — the pinned guidance
  stays sound on the practice-based horizon (~6 months, i.e. ~Jan 2027), and
  nothing upstream will ever change under the pin. Staleness is
  **event-driven**: a CVE affecting the stdlib usage patterns, a breaking
  change in a connector's public API endpoint, or FI adopting the successor.
- **Re-audit by:** event-driven re-audit only for this fork. Concrete
  decision point: **evaluate successor repo `aaron-he-zhu/aaron-marketing-skills`
  (fresh Mode A intake) by ~2027-01-15** and either migrate to it or
  affirm continued use of this frozen pin.

### Operating conditions (reaffirmed + one new)

- Install only from this fork at the pinned tag — never from upstream, which
  is frozen and has rewritten history before.
- **Installed copies in client workspaces predate the audited commit** (per
  the 2026-06-30 tag warning: skills moved to `build/`, contents drifted).
  Re-sync every installed copy from tag `audited-2026-07-18` (content
  baseline v9.9.12) before trusting the pin, and drop/refresh `.skill-pin`
  files accordingly.

---

## Audit — 2026-06-30

*Transcribed 2026-07-18 from the message of annotated tag
`audited-2026-06-30` (tag object `e65c92da98d5501ae4cf70603f28f16308e60d44`);
the tag message was the only record of this audit until this file was
created. Structure adapted to the AUDIT.md template; bracketed notes are
transcription annotations, not part of the original record.*

- **What this is:** Foster Interactive's public fork of
  [`aaron-he-zhu/seo-geo-claude-skills`](https://github.com/aaron-he-zhu/seo-geo-claude-skills)
  — a 20-skill SEO/GEO (Generative Engine Optimization) library for Claude
  Code: Markdown skills + zero-dependency Python-stdlib connector helpers
  and a session hook.
- **Upstream reviewed at:** commit
  `1608176f6c18de6aec62a9abf6a2074bf82c9f67` — release **v9.9.12** (HEAD at
  audit time). This fork is pinned to that reviewed commit
  (`audited-2026-06-30`); do not run un-reviewed upstream commits.
- **Date audited:** 2026-06-30
- **Audited by:** Foster Interactive, performed with Claude (Opus 4.8).
- **Why audited:** Audit-before-install governance (`skills-management`
  decision #1). Blast radius: connector scripts make outbound HTTP from dev
  machines; the hook injects repo content into Claude sessions.
- **License:** Apache-2.0.

### Scope
Scripts, connectors, and hooks at the pinned commit (v9.9.12 HEAD).

### Findings (from the tag message)
- Stdlib-only connectors to documented public APIs; credentials confined to
  their own vendors; the auto-run hook is read-only/in-repo with symlink
  rejection and injection sanitization.
- **Minor (accepted caveats):**
  1. Private-IP SSRF not blocked on user-supplied URLs.
  2. One unofficial Google autocomplete endpoint
     (`suggestqueries.google.com`).
  3. Non-defused stdlib XML parsing.
- **Warning (recorded in the tag):** the originally-claimed pin `5643f5a` /
  `audited-2026-05-04` never existed upstream (history rewritten). The
  currently-installed markdown subset predates this commit and does NOT
  match it (skills moved to `build/`, contents drifted) — re-sync installs
  from this tag before trusting the pin.

### Verdict
**LOW** — adopt at the pinned commit, with the three minor findings recorded
as accepted caveats.

### Accepted caveats
- **Private-IP SSRF:** connector fetches don't block private/loopback IPs on
  user URLs (upstream documents this as intentional, for auditing staging
  hosts). Runtime reminder: only pass URLs for hosts you intend to fetch;
  don't run connectors against attacker-chosen URL lists on
  credentialed/internal networks.
- **Unofficial autocomplete endpoint:** `suggest.py` uses an undocumented
  Google endpoint that may break, rate-limit, or conflict with Google ToS.
  Runtime reminder: the script itself warns on stderr; nothing load-bearing
  may depend on it.
- **Non-defused stdlib XML:** `sitemap.py`/`rss_monitor.py` parse fetched
  XML with `xml.etree`. Runtime reminder: treat parsed feed/sitemap content
  as untrusted data; DoS-class exposure only, bounded by the 5 MB fetch cap.

### Operating conditions
- Run only from this fork at the pinned tag; review any upstream changes
  before merging.
- [Non-conformances at the time of this audit, remediated 2026-07-18: no
  `AUDIT.md` existed at the fork root — the record lived only in the tag —
  and no cadence/staleness block was produced.]

<!-- Append future audits above the oldest entry, newest first. -->
