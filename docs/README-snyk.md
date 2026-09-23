# Snyk — setup & usage guide

Tracks the TODO item "Snyk security scan before cloud deploy" (see
`docs/README-todo.md`). This repo has never been scanned yet — nothing
here reflects actual findings, just how to get started and what to expect
given this codebase's specific shape.

## What Snyk actually checks

Snyk is four separate products under one CLI. Not all of them apply here:

| Product | What it scans | Relevant to this repo? |
|---|---|---|
| **Open Source** (`snyk test`) | Known CVEs in your dependencies (npm, pip) | Yes — `package.json` + `requirements-api.txt` |
| **Code** (`snyk code test`) | Your own source for security bugs (SAST) — injection, hardcoded secrets, etc. | Yes — the FastAPI backend especially |
| **Container** (`snyk container test`) | OS + package vulnerabilities in a Docker image | Not yet — no `Dockerfile` in this repo |
| **IaC** (`snyk iac test`) | Misconfigurations in Terraform/K8s/CloudFormation | Not yet — no infra-as-code here |

For this repo, that means two scans matter: **Open Source** and **Code**.
Add Container/IaC once a `Dockerfile` or cloud infra config actually
exists (see `CLAUDE.md`'s Deployment section — hosting provider isn't
chosen yet, so this is likely still ahead of you).

## Setup

### 1. Install the CLI

No installation needed to try it once — `npx` pulls it on demand:

```bash
npx snyk --version
```

For repeated use, install it globally instead (faster, no re-download
each time):

```bash
npm install -g snyk
snyk --version
# 1.1307.3
```

There's also a standalone binary (no Node required) if you'd rather not
touch global npm packages — see <https://github.com/snyk/cli/releases>
for `snyk-linux` etc. Either way, everything below just uses `snyk ...`;
swap in `npx snyk ...` if you didn't install globally.

### 2. Authenticate

```bash
snyk auth
```

Opens a browser to log in / create a free account and links this machine
to it. Free tier covers everything in this guide (a monthly test-run cap,
not a feature cap) — no credit card needed to get started.

### 3. Sanity-check from the repo root

```bash
cd /home/gongai/projects/digital-duck/conceptbook-app
snyk test --all-projects
# Tested 2 projects, no vulnerable paths were found

```

`--all-projects` matters here specifically: this repo has *two* dependency
manifests side by side (`package.json` for the frontend,
`requirements-api.txt` for the backend) — without it, Snyk only auto-detects
one of them (whichever it finds first walking the directory tree).

## Usage tutorial

### Scan dependencies (Open Source)

```bash
# Both manifests in one pass
snyk test --all-projects


# Just the Python backend
snyk test --file=requirements-api.txt --package-manager=pip > ./tests/snyk/py.txt

# Just the JS frontend
snyk test --file=package.json > ./tests/snyk/js.txt
```

Output for each vulnerable dependency looks like:

```
✗ High severity vulnerability found in some-package
  Description: Prototype Pollution
  Info: https://snyk.io/vuln/SNYK-JS-SOMEPACKAGE-1234567
  Introduced through: some-package@1.2.3
  Fixed in: some-package@1.2.4
```

Fix path is almost always: bump the version Snyk names, or if it's a
*transitive* dependency (something a direct dependency pulls in, not
something in `package.json`/`requirements-api.txt` directly), Snyk tells
you the dependency chain so you know which direct dependency to update to
get the fix.

**`>=` version pins matter here.** `requirements-api.txt` pins everything
with `>=` (e.g. `fastapi>=0.111`), not `==`, so a fresh
`pip install -r requirements-api.txt` always pulls latest-compatible —
meaning a real deploy should pin exact versions (`pip freeze` style, or a
proper lockfile) so Snyk's scan reflects what's *actually* running in
production, not just what a scan run today happens to resolve to.

### Scan your own code (Code / SAST)

```bash
snyk code test  > tests/snyk/code-1.txt

snyk code test  > tests/snyk/code-2.txt

```

#### Enable Snyk Code

```
1. Go to https://app.snyk.io and log in, then make sure the wen.gong.research org is selected (org switcher, top-left).
2. Go to that org's Settings → Snyk Code.
3. Toggle Enable Snyk Code. It'll ask you to accept the data-processing terms — Snyk Code uploads source snippets to their SAST engine for analysis, so this consent is required before it'll scan anything.
4. Once enabled, re-run snyk code test > tests/snyk/code.txt from this same shell.
```


This is the one worth paying attention to given what this backend does —
several routers accept user-supplied strings that become filesystem paths
or subprocess arguments:

- `api/services/executor.py`, `api/services/ingest_svc.py` — spawn
  subprocesses (`spl3`, `cb_pipeline`, `sync_from_press.py`) with
  request-derived arguments (`domain`, `target`, `level`, `language`,
  `model`, `pdf_path`, `prefix`).
- `api/services/manage_svc.py` — shells out to `concept_graph.py` with a
  request-derived `graph.yaml` path.
- `api/services/path_safety.py` — the existing guardrail
  (`safe_segment`/`safe_optional_segment`/`assert_within`) every router is
  *supposed* to run these values through before they touch a path or
  command.

A SAST tool like Snyk Code generally can't tell that `path_safety.py` is
being called upstream of a given subprocess call (that requires tracing
call graphs it doesn't always follow) — expect it to flag some of these
as potential path/command injection even where `path_safety.py` already
covers it. That's the main category of finding to triage by hand rather
than blindly "fix": check whether the flagged value actually passes
through `safe_segment`/`assert_within` first. If it does, it's a
resolvable false positive (see Suppressing findings below); if it
doesn't, it's a real gap — pass it through `path_safety.py` like every
other router does.

**Content injection is a related but separate risk Snyk Code won't catch
at all** — this pipeline renders third-party PDF text through an LLM into
static HTML, which is a content-provenance risk, not a code pattern any
generic SAST tool recognizes. Already found and fixed by hand (see
`docs/README-todo.md`'s "Content-injection fix in generated HTML" and
"Prompt-injection hardening for PDF extraction" items,
`spl/tools.py`'s `_md_to_html`/`_inline_md`, and the new
`scripts/sanitize_html_content.py` for scanning/fixing already-generated
files) — worth knowing about so a clean Snyk Code run isn't mistaken for
"this class of risk is covered."

### Continuous monitoring

```bash
snyk monitor --all-projects
```

Unlike `snyk test` (one-off, prints results, exits — nothing sent
anywhere), `snyk monitor` uploads a dependency snapshot to your Snyk
account's dashboard and watches it going forward: if a *new* CVE gets
disclosed next month for a dependency already in this repo, Snyk emails
you even though nothing in the repo changed. Run this once after the
first clean `snyk test` pass, and again after any dependency bump.

### CI integration (once there's a pipeline)

This repo has no `.github/workflows/` yet. When one exists, the standard
pattern is a Snyk GitHub Action that fails the build on new
high/critical-severity findings:

```yaml
# .github/workflows/security.yml (example — not yet added to this repo)
- uses: snyk/actions/node@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  with:
    args: --all-projects --severity-threshold=high
```

`SNYK_TOKEN` comes from your account's API token (Account Settings on
snyk.io) — store it as a GitHub Actions secret, never commit it.

## Key CLI flags worth knowing

| Flag | What it does |
|---|---|
| `--all-projects` | Scan every manifest found in the repo, not just the first one (needed here — see Setup) |
| `--severity-threshold=high` | Only fail/report at high+critical — useful once low-severity noise is triaged and you just want new regressions to block something |
| `--json` | Machine-readable output — pipe to a file for later diffing between scans |
| `--sarif-file-output=results.sarif` | Emits SARIF, the format GitHub's own code-scanning UI understands (upload via `github/codeql-action/upload-sarif` in CI) |
| `-d` | Debug output — verbose, useful if a scan errors out on manifest resolution |
| `snyk test --dev` | Include devDependencies (Snyk skips them by default for `package.json`) — relevant here since `vite`/`playwright`/`gh-pages` are all `devDependencies` |

## Suppressing a finding (false positives, accepted risk)

```bash
snyk ignore --id=<vulnerability-id> --reason="Reviewed: path_safety.py validates this before it reaches subprocess" --expiry=2026-12-31
```

Writes to a `.snyk` policy file at the repo root, which `snyk test` reads
on every future run — the finding still shows up but is marked
suppressed with your reason attached, rather than silently disappearing.
Always set `--expiry` (a re-review date) rather than ignoring forever —
an ignored finding is a decision made with today's code; a later change
nearby could invalidate the reasoning.

## Suggested first pass for this repo

1. `snyk auth`
2. `snyk test --all-projects` — fix every dependency CVE it finds (bump
   versions; for `requirements-api.txt`'s loose `>=` pins, also consider
   whether to start pinning exact versions for reproducible deploys).
3. `snyk code test` — triage each finding against the path-safety
   question above; fix real gaps, `snyk ignore` documented false
   positives.
4. `snyk monitor --all-projects` — so new CVEs disclosed later surface
   automatically instead of requiring a manual re-scan to notice.
5. Re-run `snyk test --all-projects --severity-threshold=high` right
   before the actual cloud deploy as a final gate, per the TODO item.
