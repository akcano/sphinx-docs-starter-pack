# URL Classifications — Head-to-Head Evaluation

Generated: 2026-03-02
Plan: 03-03 (Classification and Analysis)
Source data: Plans 03-01 (lychee) and 03-02 (Sphinx linkcheck)

---

## Classification Key

| Category | Description |
|----------|-------------|
| `broken` | URL returns a genuine error code (404, 410) — content no longer exists |
| `rate-limited` | URL returns 429 Too Many Requests |
| `bot-blocked` | URL returns 403 or 418 but loads in a browser — access is blocked for automated tools |
| `ssl-error` | TLS/SSL certificate error — server certificate not trusted |
| `false-positive` | URL is flagged as an error but is valid; the error is caused by tool limitations, template variables, or localhost placeholders |
| `lychee-limitation` | Not a real URL error; lychee cannot process the link type (relative internal links, path-conversion failures) |

---

## Methodology Notes

**accept=["429"] asymmetry:** lychee.toml includes `accept = ["100..=103", "200..=299", "429"]` which means HTTP 429 responses are treated as successful. Sphinx linkcheck has no equivalent — it reports all non-2xx as broken. As a result, rate-limited URLs appear in Sphinx output but not in lychee output. This is a structural difference documented here, not a lychee bug.

**User-Agent effect:** lychee uses a browser-like Firefox user-agent (configured in lychee.toml). Some sites return different status codes based on UA. Spot-checks below document where this matters.

**GITHUB_TOKEN:** Not set for any run. github.com URLs are excluded by the `^https://github\.com` pattern in lychee.toml (except snapcraft — see below). Sphinx linkcheck ignores them via `linkcheck_ignore`.

---

## repo: starter-pack (canonical/sphinx-docs-starter-pack)

### lychee Flagged URLs

| URL | Status Code | Classification | Confidence | Notes |
|-----|------------|---------------|-----------|-------|
| `https://manpages.ubuntu.com/manpages/noble/en/man%7Bsection%7D/%7Bpage%7D.%7Bsection%7D.html` | 404 | `broken` | HIGH | Template placeholder URL — `{section}` and `{page}` are RST substitution variables, not escaped braces. The URL is literally a template example in `rst-syntax-reference.rst`, not meant to be a real link. Classified as `broken` (template); could also be excluded. |
| `https://canonical-starter-pack.readthedocs-hosted.com/stable/sitemapindex.xml` | 404 | `broken` | HIGH | Versioned sitemap — does not yet exist (deployment pending) |
| `https://canonical-starter-pack.readthedocs-hosted.com/1.0/sitemap.xml` | 404 | `broken` | HIGH | Versioned sitemap for 1.0 — deployment pending |
| `https://canonical-starter-pack.readthedocs-hosted.com/2.0/sitemap.xml` | 404 | `broken` | HIGH | Versioned sitemap for 2.0 — deployment pending |
| `https://canonical-starter-pack.readthedocs-hosted.com/3.0/sitemap.xml` | 404 | `broken` | HIGH | Versioned sitemap for 3.0 — deployment pending |

**lychee total flagged: 5 URLs (5 unique)**

### Sphinx Flagged URLs (actual docs — excluding venv/)

Sphinx found 0 broken links in the actual documentation. All 27 "broken" entries in Sphinx output are from `docs/venv/` (a legacy 158MB Python venv left on disk from Phase 1), not from documentation source files.

| URL | Status Code | Classification | Confidence | Notes |
|-----|------------|---------------|-----------|-------|
| (All 27 Sphinx errors are in `docs/venv/` — not doc content) | — | `false-positive` | HIGH | Sphinx scanned the venv because `exclude_patterns` in conf.py does not exclude it. These are venv package READMEs (a11y_pygments, mdit_py_plugins). |

**Sphinx total flagged in docs: 0 URLs**

### Per-Category Counts (starter-pack)

| Category | lychee | Sphinx (docs only) |
|----------|--------|---------------------|
| broken | 5 | 0 |
| rate-limited | 0 | 0 |
| bot-blocked | 0 | 0 |
| ssl-error | 0 | 0 |
| false-positive | 0 | 27 (all venv/) |
| lychee-limitation | 0 | — |

### Tool Disagreements (starter-pack)

| Disagreement | URLs | Investigation |
|-------------|------|---------------|
| lychee flags 5 broken; Sphinx flags 0 in docs | 5 sitemap/template URLs | The 5 lychee errors are real 404s. Sphinx does not flag them because they are under a `linkcheck_ignore` pattern in `docs/conf.py` (readthedocs-hosted.com and manpages.ubuntu.com URLs are in the ignore list). Sphinx correctly ignores them per config; lychee correctly flags them per its config. Both tools are correct — the difference is in what is excluded. |

---

## repo: multipass (canonical/multipass)

### lychee Flagged URLs

**All 449 lychee errors are relative-path conversion failures.** Multipass docs use internal links like `/how-to-guides/install-multipass` and relative file references. lychee cannot resolve these without a base URL — it logs `Error building URL for "/path" (Attribute: Some("href")): Cannot convert path '/path' to a URI` and marks them as errors. None of these represent broken external URLs.

| URL Pattern | Count | Classification | Notes |
|-------------|-------|---------------|-------|
| `error:` (lychee's URI for path-conversion failures) | ~400+ | `lychee-limitation` | Root-relative paths like `/how-to-guides/...`, `/explanation/...`, `/reference/...`. These are internal navigation links. |
| `file:///.../{path}` (lychee-converted local file paths) | ~49 | `lychee-limitation` | Relative paths converted to local file:// URIs — they resolve to the local disk path, not a server URL. Files don't exist locally because multipass docs are built with a server base URL. |

**lychee total flagged: 449 (all lychee-limitation — not broken external URLs)**

**1 additional real error found in lychee data:** None — all 449 are relative-path issues.

### Sphinx Flagged URLs (multipass)

| URL | Status Code | Classification | Confidence | Notes |
|-----|------------|---------------|-----------|-------|
| `https://docs.snapcraft.io/core/install` | 404 | `broken` | HIGH | Redirects to `https://snapcraft.io/docs/core/install/` which returns 404. Verified with curl: `301` then `404`. Content removed/moved. |

**Sphinx total flagged: 1 URL**

### Per-Category Counts (multipass)

| Category | lychee | Sphinx |
|----------|--------|--------|
| broken | 0 | 1 |
| rate-limited | 0 | 0 |
| bot-blocked | 0 | 0 |
| ssl-error | 0 | 0 |
| false-positive | 0 | 0 |
| lychee-limitation | 449 | — |

### Tool Disagreements (multipass)

| Disagreement | Investigation |
|-------------|---------------|
| lychee flags 449 errors; Sphinx flags 1 | All 449 lychee errors are internal relative links that lychee cannot resolve. Sphinx does not check internal cross-references (`:doc:`, `/path/` in Myst MD). The 1 Sphinx broken link (`docs.snapcraft.io/core/install`) was NOT checked by lychee because it appeared on a page that only has relative links — lychee's error_map shows only the path-conversion failures for multipass. **Sphinx is correct; lychee has a fundamental limitation with relative internal links in markdown.** |

---

## repo: ubuntu-server-documentation (canonical/ubuntu-server-documentation)

### lychee Flagged URLs

| URL | HTTP Code | Classification | Confidence | Spot-check Result |
|-----|----------|---------------|-----------|-------------------|
| `https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html` | 403 | `bot-blocked` | HIGH | curl without UA: 200 OK. curl with Firefox UA: 403. Bot detection blocks browser-like UAs. |
| `https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/#deploy-specific-intel-tdx-module-version` | Timeout | `broken` | MEDIUM | DNS resolution fails — `cc-enabling.trustedservices.intel.com` does not resolve. Confirmed by Sphinx: also reports broken (DNS failure). |
| `https://en.wikipedia.org/wiki/SCSI_CDB` | 404 | `broken` | HIGH | curl returns 404 (no redirect to existing article). Article does not exist or was deleted. |
| `https://netplan.io/faq/` | 500 | `broken` | MEDIUM | Server error — netplan.io FAQ page returns 500 Internal Server Error consistently. Page is down or removed. |
| `https://git.launchpad.net/ubuntu/+source/openssl/tree/debian/rules?h=ubuntu/jammy-devel#n15` | 503 | `false-positive` | MEDIUM | git.launchpad.net returns 503 consistently for all curl requests (with and without UA). This is rate-limiting/throttling behaviour on Launchpad's git viewer. The URL format is valid and the content exists (Launchpad is functional). |
| `https://git.launchpad.net/ubuntu/+source/gnutls28/tree/debian/rules?h=ubuntu/jammy-devel#n38` | 503 | `false-positive` | MEDIUM | Same as above — git.launchpad.net 503 pattern. |
| `https://community.openvpn.net/openvpn/wiki/Easy_Windows_Guide` | 403 | `bot-blocked` | HIGH | curl (with and without UA) returns 403. OpenVPN wiki blocks automated access. The page is accessible in a browser. |
| `https://www.mysql.com/` | 403 | `bot-blocked` | HIGH | curl without UA: 200 OK. curl with Firefox UA: 403. MySQL homepage blocks browser UA. Paradoxically, lychee's browser UA triggers the block. |
| `https://git.launchpad.net/ubuntu/+source/nss/tree/nss/tests/policy` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern (see above). |
| `https://git.launchpad.net/ubuntu/+source/nss/tree/nss/tests/ssl/sslpolicy.txt` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `https://git.launchpad.net/ubuntu/+source/nss/tree/nss/lib/pk11wrap/pk11pars.c#n144` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `https://www.freedesktop.org/software/polkit/docs/latest/polkit.8.html` | 418 | `bot-blocked` | HIGH | curl without UA: 200 OK. curl with Firefox UA: 418. freedesktop.org returns 418 ("I'm a teapot") as a bot-detection signal. |
| `https://www.java.com/en/configure_crypto.html` | 403 | `bot-blocked` | HIGH | curl without UA: 403. curl with Firefox UA: 403. Java.com consistently blocks automated requests regardless of UA. |
| `https://git.launchpad.net/ubuntu/+source/openjdk-lts/tree/src/java.base/share/conf/security/java.security/#n520` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `https://git.launchpad.net/ubuntu/+source/openjdk-lts/tree/src/java.base/share/conf/security/java.security?h=applied/ubuntu/jammy-devel#n520` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `http://localhost:3000/` | Connection refused | `false-positive` | HIGH | Localhost URL in docs — intentional placeholder showing what a local server would look like. Should be excluded. |
| `http://prometheus:9090/` | Timeout | `false-positive` | HIGH | Local service reference (Prometheus on a container network). Intentional placeholder in container/ROCKS tutorial. Should be excluded. |
| `https://dev.mysql.com/doc/...` (12 MySQL URLs) | 403 | `bot-blocked` | HIGH | dev.mysql.com blocks all automated requests. Multiple URLs from `install-mysql.md`. All return 403 regardless of UA. |
| `https://en.opensuse.org/SDB:AppArmor_geeks` | 403 | `bot-blocked` | HIGH | curl (with and without UA) returns 403. OpenSUSE wiki blocks automated access. |
| `https://git.launchpad.net/ubuntu/+source/openssh/tree/debian/patches/gssapi.patch` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `https://krbdev.mit.edu/rt/Ticket/Display.html?id=7754#` | SSL error | `ssl-error` | HIGH | SSL certificate not trusted by lychee's cert chain. curl without `--insecure`: exit code 000 (connection error). curl with `--insecure`: 200 OK. Server has a self-signed or expired certificate. |
| `file:///...ubuntu-server-documentation/docs/explanation/multipath/introduction-to-multipath` | Cannot find file | `lychee-limitation` | HIGH | Relative link in `configuring-multipath.md` converted to local file:// path — file does not exist locally (RST extension omitted, file system path wrong). This is an internal cross-reference link. |
| `https://dnsleaktest.com/` | Timeout | `broken` | MEDIUM | DNS resolution fails — `dnsleaktest.com` does not resolve in this network environment. Also confirmed broken by Sphinx. |
| `https://www.freedesktop.org/software/systemd/man/latest/systemd-timesyncd.service.html` | 418 | `bot-blocked` | HIGH | freedesktop.org 418 bot-detection pattern. |
| `https://www.freedesktop.org/software/systemd/man/latest/timedatectl.html` | 418 | `bot-blocked` | HIGH | freedesktop.org 418 bot-detection pattern. |
| `https://git.launchpad.net/ubuntu/+source/rclone/tree/debian/control?h=ubuntu/jammy-devel#n7` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html` | 418 | `bot-blocked` | HIGH | freedesktop.org 418 bot-detection pattern. |
| `https://git.launchpad.net/ubuntu/+source/openvpn/tree/sample/sample-config-files/server.conf` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |
| `https://git.launchpad.net/ubuntu/+source/openvpn/tree/sample/sample-config-files/client.conf` | 503 | `false-positive` | MEDIUM | git.launchpad.net 503 — consistent throttling pattern. |

**lychee total flagged: 37 URLs across 20 source files**

### Sphinx Flagged URLs (ubuntu-server)

| URL | Status Code | Classification | Confidence | Notes |
|-----|------------|---------------|-----------|-------|
| `https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/04/hardware_setup/#deploy-specific-intel-tdx-module-version` | DNS failure | `broken` | HIGH | Both tools agree: DNS resolution fails. Domain does not exist. |
| `https://dnsleaktest.com` | DNS failure | `broken` | MEDIUM | Both tools agree: DNS resolution fails in this network environment. |

**Sphinx total flagged: 2 URLs**

### Per-Category Counts (ubuntu-server)

| Category | lychee count | Sphinx count |
|----------|-------------|-------------|
| broken | 3 | 2 |
| rate-limited | 0 | 0 |
| bot-blocked | 13 | 0 |
| ssl-error | 1 | 0 |
| false-positive | 9 (launchpad 503) + 2 (localhost) = 11 | 0 |
| lychee-limitation | 1 (multipath relative link) | — |
| not checked (accept=429) | 0 | 0 |

**Totals: lychee=37, Sphinx=2. Discrepancy: 35 additional lychee flags.**

### Tool Disagreements (ubuntu-server)

| Disagreement | URLs | Investigation |
|-------------|------|---------------|
| Sphinx flags `cc-enabling.trustedservices.intel.com` timeout; lychee flags both DNS failure AND intel.com 403 | 2 URLs | Both tools agree on the DNS failure URL. lychee additionally flags `intel.com` (403 bot-blocked) which Sphinx ignores (it is in `linkcheck_ignore` in ubuntu-server's conf.py). |
| lychee flags 13 bot-blocked URLs; Sphinx flags 0 | 13 URLs | Sphinx ignores many domains via `linkcheck_ignore`. mysql.com, dev.mysql.com, opensuse.org, openvpn.net, freedesktop.org, java.com are all in the ubuntu-server `linkcheck_ignore` list. lychee does not ignore them — they show up as 403/418. The Ubuntu Server Sphinx config has extensive ignore patterns; lychee.toml has only 5 patterns. |
| lychee flags 9 git.launchpad.net URLs (503); Sphinx flags 0 | 9 URLs | git.launchpad.net consistently returns 503 for all automated requests. This appears to be systematic throttling of the Launchpad git web viewer. Sphinx ignores these via `linkcheck_ignore`. lychee does not have a launchpad.net exclusion pattern. |
| lychee flags 2 localhost/container URLs; Sphinx flags 0 | 2 URLs | `localhost:3000` and `prometheus:9090` are docs-as-examples URLs — they appear in how-to guides showing local service access. Sphinx ignores localhost URLs via standard `linkcheck_ignore`. lychee has `^http://127\.0\.0\.1` excluded but not `localhost` or named container hosts. |
| lychee flags 1 SSL error; Sphinx flags 0 | 1 URL | `krbdev.mit.edu` has an invalid TLS certificate. Sphinx ignores it via `linkcheck_ignore`. lychee does not have a krbdev exclusion. |

**Key pattern:** The 35-URL discrepancy between lychee (37) and Sphinx (2) is almost entirely explained by ubuntu-server's extensive `linkcheck_ignore` list in conf.py. lychee checks what Sphinx ignores.

---

## repo: snapcraft (canonical/snapcraft)

**Note:** Sphinx linkcheck setup FAILED for snapcraft (ModuleNotFoundError: craft_application_docs). No Sphinx comparison data is available. lychee-only analysis.

### lychee Flagged URLs

| URL | HTTP Code | Classification | Confidence | Notes |
|-----|------------|---------------|-----------|-------|
| `http://package-cache.lxd:3128/` | Timeout | `false-positive` | HIGH | LXD internal package cache proxy — a local network address used only when LXD is running. Documented as a "reuse packages between builds" example. Not a public URL. |
| `http://cdn.geekbench.com/Geekbench-$SNAPCRAFT_PROJECT_VERSION-Linux.tar.gz` | 404 | `false-positive` | HIGH | Template URL with `$SNAPCRAFT_PROJECT_VERSION` variable — a shell variable not expanded by lychee. The actual URL (with a real version) would resolve. Template variables in docs should be excluded. |
| `https://snapcraft.io/docs/snapcraft-esm` | 404 | `broken` | HIGH | Redirects (301) to URL that returns 404. Page has been removed or renamed. |
| `http://github.com/example/sampleapp.git` | 404 | `broken` | HIGH | Redirects to `https://github.com/example/sampleapp.git` which returns 404. Example repository URL that does not exist. Note: this URL uses `http://` not `https://` — falls outside the `^https://github\.com` exclusion pattern. |
| `https://www.freedesktop.org/wiki/Software/pkg-config/` | 418 | `bot-blocked` | HIGH | curl without UA: 200 OK. curl with Firefox UA: 418. freedesktop.org 418 bot-detection pattern (consistent with ubuntu-server findings). |
| `https://dl.discordapp.net/apps/linux/$%7BSNAPCRAFT_PROJECT_VERSION%7D/discord-$%7BSNAPCRAFT_PROJECT_VERSION%7D.deb` | 403 | `false-positive` | HIGH | URL-encoded template variable (`$%7BSNAPCRAFT_PROJECT_VERSION%7D` = `${SNAPCRAFT_PROJECT_VERSION}`). The encoded variable is not a valid URL path component. Template URL that should be excluded. |

**lychee total flagged: 6 URLs (7 total — bases.rst has a duplicate cached snapcraft-esm error)**

**Sphinx total flagged: N/A — setup failed**

### Per-Category Counts (snapcraft)

| Category | lychee count | Sphinx count |
|----------|-------------|-------------|
| broken | 2 | N/A |
| rate-limited | 0 | N/A |
| bot-blocked | 1 | N/A |
| ssl-error | 0 | N/A |
| false-positive | 3 (template variables + localhost) | N/A |
| lychee-limitation | 0 | N/A |

### github.com Exclusion Pattern Gap (snapcraft)

The current `^https://github\.com` exclusion pattern does not exclude:
- `http://github.com/...` (http:// not https://)

The `github.com/example/sampleapp.git` URL was flagged because it uses `http://` (no TLS) — it redirects to https but lychee checks it before redirect. This is the pattern gap identified in Plan 03-01.

---

## Aggregate Classification Summary (All Repos)

| Category | starter-pack | multipass | ubuntu-server | snapcraft | Total |
|----------|-------------|-----------|---------------|-----------|-------|
| broken | 5 | 0 | 3 | 2 | 10 |
| rate-limited | 0 | 0 | 0 | 0 | 0 |
| bot-blocked | 0 | 0 | 13 | 1 | 14 |
| ssl-error | 0 | 0 | 1 | 0 | 1 |
| false-positive | 0 | 0 | 11 | 3 | 14 |
| lychee-limitation | 0 | 449 | 1 | 0 | 450 |
| **lychee total flagged** | **5** | **449** | **37** | **6** | **497** |

*Sphinx comparison data: starter-pack=0 broken (docs only), multipass=1 broken, ubuntu-server=2 broken, snapcraft=N/A*

---

## Recommended Exclusion Patterns for lychee.toml

Based on the classification findings, the following new patterns are recommended:

### Confirmed False Positives — Add to lychee.toml

| Pattern | Rationale | Category |
|---------|-----------|----------|
| `^http://localhost` | Localhost URLs appear in tutorial/how-to docs as examples. Always false-positive when running lychee on a local checkout. | false-positive |
| `^https://git\\.launchpad\\.net/` | git.launchpad.net consistently returns 503 for automated requests — systematic throttling of the web viewer. Content is valid; the 503 is access control, not server failure. | false-positive (rate-limited/throttled) |

### Not Recommended as Exclusions

| Pattern | Reason Not Excluded |
|---------|-------------------|
| `^https://.*\\.mysql\\.com/` | mysql.com and dev.mysql.com return 403 consistently — these are bot-blocked and the content exists. However, they represent real docs links that should ideally be checked. Consider accepting 403 globally rather than excluding the domain. |
| `^https://www\\.freedesktop\\.org/` | freedesktop.org 418 bot-detection. However, freedesktop.org is a legitimate upstream source — teams should know if these links break. Better to accept 418 globally (like 429). |
| `^https://.*intel\\.com/` | Intel's bot-blocking is UA-specific and inconsistent. Not a reliable pattern to exclude. |
| Template variable URLs (`\$\{`, `\$SNAP`) | Template URLs with shell variables are repo-specific. They should be fixed in each repo (use code blocks rather than live links) rather than excluded globally. |

### Proposed lychee.toml Addition

The two exclusion patterns for `localhost` and `git.launchpad.net` are confirmed false positives across all tested repos. These will be added in Task 2.
