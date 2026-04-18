# Threat Model

This project makes two strong claims to the user:

1. **Nothing is collected, stored, or transmitted.** Probe results stay in the visitor's browser.
2. **The interpretation of the data is accurate.** The page tells the user what each API really exposes.

This document enumerates the threats to those claims, the mitigations in place, and the residual risk.

## Assets

| Asset | Owner | Why it matters |
|---|---|---|
| User's probe results (IPs, device names, clipboard, geo) | Visitor's browser | Sensitive. Must never leave the device. |
| User's trust that the page is honest | Project reputation | If the page *did* exfiltrate, the whole premise collapses. |
| The served HTML itself | GitHub Pages | A tampered page could silently exfiltrate to an attacker. |
| The source on GitHub | Repo owner (`niavasha`) | A malicious PR or push could introduce exfil code that Pages then serves. |

## Trust boundaries

```
[ Visitor's machine ]
        |
        | HTTPS (enforced by GitHub Pages)
        v
[ GitHub Pages CDN ]  <-- serves static HTML built from main branch
        |
        | git push over HTTPS (OAuth token)
        v
[ github.com/niavasha/chrome-network-access ]
        |
        | local git
        v
[ Maintainer's workstation ]
```

The only data *leaving* the visitor's machine during normal use is:

- The initial GET for `index.html` (unavoidable)
- Probe fetches **that the user clicks** (to private IPs, localhost, etc. — deliberately)
- `Download JSON` — writes a local file; does not upload

There is **no backend, no analytics, no remote logging, no fonts from CDNs, no third-party scripts**. Open DevTools → Network and you can verify this in real time.

## Threats (STRIDE-style)

### T1. Tampering: malicious push to main (HIGH likelihood over time)

**Scenario:** An attacker gains `niavasha`'s GitHub credentials / OAuth token / session, pushes a version of `index.html` that `fetch()`es probe results to an attacker-controlled server. Visitors to `niavasha.github.io/chrome-network-access/` now silently leak data.

**Mitigations:**
- Repo is owned by a single maintainer — limited blast radius, but also a single point of failure.
- All source is inline and human-readable (single ~700 line file). Auditable in minutes.
- GitHub provides commit signing; this repo does **not currently require signed commits** (TODO).
- No package dependencies — no supply-chain risk via `npm`, `pip`, submodules, or transitive imports.

**Residual risk:** High if maintainer's GitHub session is compromised. Mitigated by visitor's ability to read the served source at any time — but few visitors will actually do this.

**Planned action:** Add branch protection requiring signed commits; add a `SECURITY.md` with a verification recipe (`curl https://niavasha.github.io/chrome-network-access/ | sha256sum` against a pinned hash in README).

### T2. Tampering: GitHub Pages pipeline compromise (LOW likelihood, HIGH impact)

**Scenario:** GitHub itself, or a Pages build-system component, is compromised and serves a modified page to some fraction of visitors.

**Mitigations:**
- Out of scope — there is no practical defence against a compromised CDN from a static site.
- Visitors who want strong assurance should clone the repo and open `index.html` locally (`file://` — note LNA prompt won't fire but no exfil is possible either).

**Residual risk:** Accepted.

### T3. Spoofing: lookalike domain (MEDIUM likelihood)

**Scenario:** An attacker clones the repo, deploys it at `chrome-network-access.com` (or similar), adds exfil, and promotes it as "the tool". Visitors type the wrong URL or follow a phishing link.

**Mitigations:**
- The canonical URL is published in the README and linked from the page itself (`github.com/niavasha/chrome-network-access`).
- MIT licence permits forks, so some forks will exist — not all forks are malicious.

**Residual risk:** Medium. Detection, not prevention. Rely on visitors verifying the URL.

### T4. Information disclosure: self-XSS via user-typed input (MEDIUM likelihood, LOW impact)

**Scenario:** The LNA probe accepts comma-separated URLs typed by the user. If those strings were interpolated into `innerHTML` without escaping, a user who pastes a crafted URL (e.g. from a tutorial or someone's Discord) could execute arbitrary JavaScript in their own browser against this page's origin. On GitHub Pages that means no cross-origin cookie access, but an attacker's code could still:

- Silently exfil their probe results to a remote server
- Pretend the page is reporting different findings than it actually is
- Steal the origin's `localStorage` (currently empty, but a future feature might add it)

**Mitigations implemented:**
- All user-controlled strings (LNA URLs, clipboard preview, device names, error messages) are now passed through a dedicated `esc()` function before being written to `innerHTML`.
- Port numbers are passed through `parseInt()` so they are numeric, not strings.
- Interpreters that accept untrusted error-object fields (e.g. `e.message`) escape them.

**Residual risk:** Low, but the page uses `innerHTML` for interpretation — a future edit that forgets to escape a new field would regress. A Content Security Policy is recommended as a defence-in-depth layer (see T7).

### T5. Information disclosure: malicious browser extension (LOW likelihood for most users)

**Scenario:** The visitor has a malicious extension with `"<all_urls>"` permissions that reads page content and exfils it.

**Mitigations:**
- Out of scope — an extension that can read the page can also read your bank.
- The page itself provides no extension-privileged APIs.

**Residual risk:** Accepted. The visitor's device security is their responsibility.

### T6. Repudiation / false accuracy of interpretation (MEDIUM likelihood)

**Scenario:** The page tells a user "your IPs are not leaking" when they actually are, or vice versa. A user relies on a wrong interpretation and makes a bad privacy decision.

**Historical precedent:** Already happened three times in development (port-scan miscounting closed ports as live devices; colour-coding without stating a lens; port 22 labelled "closed" when it was actually browser-blocked). Each was caught by user review.

**Mitigations:**
- Raw JSON is always visible (collapsible `<details>`) alongside every interpretation. Users can verify.
- Interpretation logic is pure and unit-testable (though no tests currently exist — TODO).
- README explicitly positions the tool as a transparency aid, not a security certification.

**Residual risk:** Medium. This document should be updated whenever an interpretation is changed.

### T7. Missing CSP (LOW impact, easy fix)

**Scenario:** If a T4-style XSS slips through, an attacker's injected code could `fetch()` to any origin for exfil.

**Mitigations:** None currently. A Content Security Policy `<meta>` tag can restrict outbound `connect-src` but is complicated by the fact that the page's *legitimate* function is to fetch arbitrary private IPs. A CSP like `connect-src 'self' http://127.0.0.1:* http://192.168.0.0/16 …` is too restrictive to cover all possible user-typed LNA targets.

**Planned action:** Add `default-src 'self' 'unsafe-inline'; script-src 'unsafe-inline'; object-src 'none'; base-uri 'none'; form-action 'none';` — restricts most vectors while leaving fetch unconstrained (since that's the whole point of the tool). Accept that a successful XSS could still exfil via `fetch()`.

### T8. Denial of service (LOW impact)

**Scenario:** GitHub Pages outage.

**Mitigations:** None; clones work offline.

**Residual risk:** Accepted.

## Out of scope

- Browser bugs in Chrome/Firefox/Safari that bypass same-origin policy or leak beyond documented APIs. The tool can surface such bugs but cannot defend against them.
- Physical attacks on the visitor's machine.
- Nation-state adversaries with the ability to compromise GitHub infrastructure.
- Side-channel timing attacks beyond the port-scan demo itself.

## Summary

| Threat | Likelihood | Impact | Status |
|---|---|---|---|
| Malicious push to main | Medium | High | Partially mitigated; signed commits recommended |
| GitHub Pages compromise | Low | High | Accepted |
| Lookalike domain | Medium | Medium | Link canonical URL from README + page |
| Self-XSS via user input | Medium | Low | **Mitigated** (escaping applied) |
| Malicious extension | Variable | High | Out of scope |
| Wrong interpretation | Medium | Medium | Mitigated via raw JSON + open review |
| Missing CSP | Low | Low | Planned |
| DoS | Low | Low | Accepted |

Found a gap? Open an issue at https://github.com/niavasha/chrome-network-access/issues.
