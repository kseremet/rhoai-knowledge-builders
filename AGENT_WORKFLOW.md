# Agent operational playbook

This is not part of the wiki schema contract (that's `CLAUDE.md` — read that
first, always). This file is the *how* — the mechanics an agent session
needs to actually run an ingest against this repo efficiently, discovered
across the sessions that built the current wiki. Read this before starting
an ingest if you're a fresh session with no memory of prior runs.

## Priorities (standing decision, 2026-08-25)

- **RHOAI is the main focus.** Prioritize `raw/rhoai/**` over everything else.
- **NVIDIA GPU Operator (`gpu-operator` source) is priority** — RHOAI depends
  on it for GPU support, so its CRDs are worth documenting in depth.
- **General OpenShift (`ocp` source) is deprioritized** except monitoring,
  service mesh, and authentication/authorization. `sources.yaml`'s `ocp`
  entry is already trimmed to those guides with a dated comment explaining
  why — don't re-expand it without re-confirming this decision with the user.
  The `service_mesh` guide slug in that trimmed list is still **unverified**
  (needs network access to docs.redhat.com, which this environment doesn't
  reliably have — see Gotchas below).
- Within `rhoai`, guides have been ingested roughly in "install → manage →
  use" order: getting-started, both install guides (connected +
  disconnected), managing-openshift-ai, configuring-your-model-serving-platform.
  Continuing in that spirit (deploy models → pipelines → MaaS → eval → RAG/OGX
  → distributed inference) keeps cross-references coherent, but isn't a hard
  rule — pick whatever guide the user names.

## The rhkb CLI — gotchas

- Run as `python3 -m rhkb <cmd>` from the repo root (device shell, inside the
  connected folder).
- `rhkb next` / `plan` / `status` operate on *sources* or *tiers*, not
  individual documents — there's no built-in "ingest just this one guide"
  filter. To scope work to a single `raw/` file, call
  `rhkb.units.iter_units()` directly (see next section) and filter by
  `u.doc == '<source>/<version>/<file>.md'`.
- **Default `unit_tokens` is 6000**, read from `sources.yaml`'s
  `unit_tokens` key via `catalog.int_default("unit_tokens", 6000)`
  (`rhkb/cli.py`). If you generate unit IDs with any other `unit_tokens`
  value, `rhkb done <id>` will fail with `no unit '<id>'` for a chunk of
  them — the unit boundaries (and hence IDs) shift with the token size.
  **Always pass `unit_tokens=6000`** when calling `iter_units` yourself, to
  match what the CLI itself uses.
- `iter_units(raw_root: Path, sources: Optional[List[str]], unit_tokens: int)`
  — `sources` is a list of source-id *strings* (e.g. `['rhoai']`), not
  `Source` objects — don't pass `catalog.select(...)` output directly.
- `rhkb done <id1> <id2> ...` / `rhkb skip <id1> ...` do exact-match or
  unique-substring resolution (`_resolve_ids`) — fine for a guide's worth of
  units (dozens), impractical for hundreds. For a bulk operation across many
  units (e.g. the OCP deprioritization), write a script using `State` from
  `rhkb.state` directly instead (`mark_skipped`, `.save()`).
- No reliable outbound network access from this environment — `rhkb guides
  --source X` (which hits docs.redhat.com to list available guides) will
  fail with a `ProxyError`/`403`. Work from the `raw/` files already on disk
  (`device_list_dir` on `raw/<source>/<version>/`) instead of trying to
  query Red Hat's site.

## `device_stage_files` can false-positive on FUSE-mounted sources

Discovered on the `kuadrant-operator` source (2026-08-26): `device_stage_files`
failed on *every* file from that source — brief and raw docs alike — with
`"file is hardlinked (nlink > 1) — refused to avoid reading through a link
alias"`, even after copying the files fresh (`cp`, then `cat >`) to break any
real hardlink; `device_bash stat` confirmed the copies had `nlink=1` and
staging still failed identically. The connected folder is FUSE-mounted
(`stat -f` / `df -T` show `Type: fuseblk`), while the session's own `$HOME` is
real ext4 — the working theory is the FUSE mount misreports `st_nlink` and
trips the staging tool's hardlink safety check as a false positive. Root
cause not fully confirmed.

**Workaround, not a fix:** skip `device_stage_files` for the affected source
entirely. Dispatch the drafting subagent with explicit instructions to read
the brief and raw files directly via `mcp__remote-devices__device_bash`
(`cat`, `sed -n`, `grep -n`) instead of relying on staged copies — subagents
inherit the same `mcp__remote-devices__*` tool access as the main session, so
this works without any special plumbing. This is exactly what unblocked the
first Kuadrant/inference-gateway ingest batch. If a future source hits the
same error, try this workaround before spending time on further staging
diagnosis.

## Standard ingest workflow (used successfully across ~6 guides)

1. **Generate a "brief"** — the raw file's units, concatenated with headers,
   via a small script run through `device_bash`:
   ```python
   from rhkb.units import iter_units
   from pathlib import Path
   units = list(iter_units(Path('raw'), ['rhoai'], unit_tokens=6000))
   target = [u for u in units if u.doc == 'rhoai/3.5/<file>.md']
   with open('.ingest-tmp/brief-<name>.md', 'w') as f:
       for u in target:
           f.write(f"=== UNIT ID: {u.id} ===\nTITLE: {u.title}\n"
                    f"BREADCRUMB: {u.breadcrumb}\nTOKENS: {u.tokens}\n{u.body}\n\n")
   ```
   Print `len(target)` and total tokens so you know the scale before
   committing to a drafting approach.

2. **Stage the brief + the raw file** into the cloud container via
   `device_stage_files` (both — the agent doing the drafting needs real line
   numbers from the raw file, not just the brief's unit boundaries).

3. **Dispatch drafting to an isolated subagent** (the `Agent` tool), not
   inline in the main session — this is what keeps the coordinating
   session's context budget free across many guides. Give the subagent:
   - The full `CLAUDE.md` contract, pasted verbatim (subagents don't share
     this session's context or tool access to read it themselves reliably —
     paste it in).
   - The current `wiki/index.md` content (paste inline — **do not** rely on
     a `{{PLACEHOLDER}}` substitution pattern in the prompt string; past runs
     have had this silently fail to interpolate, leaving the literal
     placeholder text in what the subagent received. Paste the real content
     directly into the prompt you send.)
   - For any existing pages plausibly in the new guide's territory (rule 5 —
     search before creating): dump their full current content and paste it
     in too, or explicitly tell the subagent it does NOT have that dump this
     round and must flag uncertainty rather than guess (better to under-merge
     and flag than silently duplicate).
   - The brief file's path and the raw file's path (both under
     `/mnt/user-data/uploads/rhoai-knowledge-builders/...` once staged).
   - Exact output instructions: new pages to `/tmp/wiki_build<N>/pages/`,
     updated existing pages as full files (same filename) to the same
     directory, `index-additions.md` (new pages only, grouped by `##
     <type>`), `log-entry.md` (a full `## [date] ingest | ...` entry per
     CLAUDE.md's log format), and a final plain-text summary line:
     `NEW_PAGES: ... UPDATED_PAGES: ... UNIT_IDS: ...` (all unit IDs from the
     brief, space-separated — needed for step 6).
   - Note: if the subagent reports a blocking problem (missing file, bad
     placeholder), use `SendMessage` to resume the *same* agent with the fix
     rather than starting a fresh one — it keeps all the work it already did
     digesting the brief.

4. **Verify citations** before shipping: `grep -n '^#{1,4} '` against the
   raw file to get the real heading line numbers, and cross-check every
   `sources:` `#L<start>-<end>` range in the subagent's output lines up with
   real heading boundaries (a citation should start at a heading line and
   end at the line before the next heading, or an intentional narrower/wider
   range with a documented reason). Fix any mismatches with a `sed`/Python
   pass before delivering.

5. **Merge `index-additions.md` into `wiki/index.md`** — insert each new
   entry under the matching `## <type>` section (create the section if it's
   the first page of that type). Watch for missing blank lines between
   sections after a merge — CommonMark needs a blank line before a heading
   for the file to render correctly elsewhere.

6. **Append the log entry** to `wiki/log.md` via `device_bash`:
   ```
   cat >> "$HOME/mnt/rhoai-knowledge-builders/wiki/log.md" << 'EOF_LOGENTRY'
   <log entry text>
   EOF_LOGENTRY
   ```
   (Strip any trailing `UNIT_IDS:` line from the subagent's log-entry
   output first — that's bookkeeping for step 8, not part of the log.)

7. **Deliver and commit the files**: `SendUserFile` on every new/updated
   page plus the updated `index.md`, then `mcp__remote-devices__device_commit_files`
   with each returned `file_uuid` mapped to its real path under
   `wiki/pages/<slug>.md` (or `wiki/index.md`). If the bridge reports "device
   not connected," don't loop retrying — tell the user and wait; retry once
   when they confirm it's back.

8. **Mark units done**: `python3 -m rhkb done <all unit IDs from step 3,
   space-separated>` — use the *canonical* IDs (regenerate via `iter_units`
   with `unit_tokens=6000` if the brief was built with a different value; see
   Gotchas above). Confirm the count matches (`marked N unit(s) done`).

9. **Clean up the scratch brief file.** `device_bash` runs with delete
   permission off by default — `rm`/`rmdir` fail with "Operation not
   permitted." Don't fight this: `mkdir -p _to_delete && mv
   .ingest-tmp/brief-<name>.md _to_delete/` and mention to the user that
   folder can be deleted manually. (If `device_request_delete_permission`
   is available and the user grants it, real deletion becomes possible —
   worth trying once per session if the `_to_delete/` folder is getting
   large.)

10. **Verify**: `python3 -m rhkb plan -s rhoai` (or the relevant source) —
    outstanding unit count should have dropped by exactly the number marked
    done in step 8.

## On running multiple guides via separate conversations

CLAUDE.md rule: **"Ingest one guide at a time — parallel ingests create
duplicate pages under near-identical names."** This is a real risk if
separate conversations (e.g. one per guide — MaaS, EvalHub, inference) run
*concurrently*: two sessions could each decide a page like
`model-serving-platforms-overview` or `evaluating-ai-systems-overview`
doesn't exist yet (because neither has seen the other's in-flight, uncommitted
work) and both create it under slightly different names, or two sessions
could both try to append to `wiki/log.md` / `wiki/index.md` around the same
time and race.

Safest pattern for multiple conversations against this repo:
- **Finish and commit one guide's ingest fully (through step 10 above)
  before starting the next**, even across separate conversations — treat
  "ingest in progress" as a lock, not something to hold open while you go
  start another.
- If truly running two in parallel is unavoidable, pick guides with minimal
  topical overlap (e.g. MaaS and a GPU-driver update are unlikely to collide;
  MaaS and inference/serving are likely to — both will want to touch
  `kserve-model-serving-crds` and friends), and do a manual reconciliation
  pass on `index.md` afterward.
- Any new session should start by reading `CLAUDE.md` (the contract) and
  this file (the mechanics), then `wiki/index.md` (current state) before
  touching anything — that's enough to pick this up cold with no memory of
  prior sessions.

## OpenShift templates (`templates/`) — a third layer, added 2026-08-25

CLAUDE.md defines two layers (`raw/`, `wiki/`). There's a third now:
`templates/` — buildable `template.openshift.io/v1` Template objects (or
Kustomize bases, whichever fits the source material better) for setting up
real components on a cluster: Inference Gateway, MaaS, EvalHub, and whatever
follows. This is a *build artifact*, not documentation — it doesn't replace
a wiki page, it's downstream of one.

**Rules for this layer** (same spirit as the wiki's, adapted):

1. **Generate templates only after the wiki pages for that topic exist.**
   The wiki page is where "what this CRD means and why" lives; the template
   is just the runnable shape. Don't skip straight to templates without the
   documentation pass — you'll end up re-deriving field meanings from raw
   YAML with no citation trail.
2. **Prefer adapting a real sample over inventing one.** Several sources
   already ship working example manifests — `raw/maas/docs/samples/maas-system/*`
   (per-tier AuthPolicy/Model/Subscription YAML), `raw/maas/docs/samples/models/*`
   (per-model YAML), `raw/kuadrant-operator/examples/toystore/*` and
   `examples/x509-authentication/*` (Gateway/AuthPolicy/RateLimitPolicy
   YAML). Parameterize these (swap hardcoded model names, namespaces, tiers,
   hostnames for `${PARAM}`-style template parameters) rather than writing
   the CR shape from scratch — it's both faster and more trustworthy, since
   the shape is proven to actually apply.
3. **Cite the source in a header comment** on every template file: which
   `raw/` sample it was adapted from, and which `wiki/pages/*.md` page(s)
   explain the fields being parameterized. Same cite-or-delete spirit as the
   wiki, just as a comment instead of frontmatter (Templates don't have a
   frontmatter slot for it).
4. **One directory per topic**: `templates/inference-gateway/`,
   `templates/maas/`, `templates/evalhub/`, etc. Each gets its own README.md
   inside explaining what the template deploys, its parameters, and
   prerequisites (what must already be installed — e.g. MaaS templates
   assume Kuadrant + a Gateway already exist).
5. **Version-qualify the same as wiki pages** — a template built against
   `maas` docs fetched at a point in time should say so (a comment noting
   the fetch date / raw source version), since MaaS's CRDs are under active
   development (see the `tier-to-subscription` and `upgrade-to-3.5` migration
   docs in `raw/maas/docs/content/migration/` — the object model has changed
   between versions before).

## Topic → raw source map (for the MaaS / EvalHub / Inference Gateway work)

- **Inference Gateway** → `raw/kuadrant-operator/` (Gateway API policy
  engine: AuthPolicy, RateLimitPolicy, TokenRateLimitPolicy, DNSPolicy,
  TLSPolicy — CRD references under `doc/reference/*`, walkthroughs under
  `doc/user-guides/*`, working samples under `examples/toystore/*` and
  `examples/x509-authentication/*`) plus `raw/rhai-inference/3.5/` (Red Hat's
  own distributed-inference-with-llm-d guides, the inference-serving side of
  the same gateway).
- **MaaS** → `raw/maas/docs/` — install docs under `content/install/*`
  (`platform-setup`, `maas-setup`, `model-setup`, `multi-tenant-setup`),
  CRD references under `content/reference/crds/*` (`tenant`, `ai-tenant`,
  `external-model`, `maas-auth-policy`, `maas-model-ref`,
  `maas-subscription`), and real deployable sample manifests under
  `docs/samples/maas-system/*` and `docs/samples/models/*`.
- **EvalHub** → inside `raw/rhoai/3.5/evaluating-ai-systems.md`, specifically
  Chapter 2 ("Evaluate LLMs with EvalHub") and Chapter 3 ("Use EvalHub with
  AI coding agents"). The same file's Chapter 4 (LM-Eval) and Chapter 5
  (automated safety/risk assessment) are related but distinct tools — don't
  conflate them with EvalHub itself when scoping an ingest to "just EvalHub."

Recommended order: **Inference Gateway first** (MaaS's own AuthPolicy/
RateLimitPolicy samples sit on top of a Gateway — documenting Kuadrant's CRDs
first lets MaaS pages link out instead of re-explaining rate-limiting), then
MaaS, then EvalHub (the most independent of the three — fine to do anytime,
even in parallel with the others since it shares little territory with
Gateway/MaaS).

## Known open follow-ups (as of 2026-08-25)

- `configuring-your-model-serving-platform.md`'s ingest didn't have a
  verified dump of `installing-rhoai-components` /
  `configuring-kserve-oauth-proxy-sidecar-resources` to check against —
  flagged as unconfirmed non-duplication in that ingest's log entry. Worth a
  quick reconciliation pass before ingesting `deploying-models.md` or any
  MaaS/inference guide, since they'll all touch the same KServe territory.
- `sources.yaml`'s `ocp.service_mesh` guide slug is unverified (no network
  access to confirm against docs.redhat.com).
- Several Kueue/Gateway CRDs (`ResourceFlavor`, `ClusterQueue`,
  `LocalQueue`, `GatewayConfig`) are richly described across existing
  procedure pages but don't have their own `component` pages yet — flagged
  as a future-ingest candidate, not urgent.

## State snapshot (2026-08-26, for orientation only — always re-check `rhkb plan`)

`rhoai` source: 431 units total, 345 outstanding (~86 done). Guides fully
ingested: `getting-started-with-red-hat-openshift-ai-self-managed`,
`installing-and-uninstalling-openshift-ai-self-managed` (+ its disconnected
sibling), `managing-openshift-ai`, `configuring-your-model-serving-platform`.
`gpu-operator` source: ~20 component pages from an early ingest (ClusterPolicy
CRD family). `ocp` source: trimmed to 5 guides, mostly still pending.

`kuadrant-operator` source: 221 units total, 190 outstanding (31 done as of
2026-08-26 — the first inference-gateway slice: `Kuadrant` CR, `AuthPolicy`,
`RateLimitPolicy`, `TokenRateLimitPolicy` + their 3 concept overviews + 2
walkthrough/tutorial procedures + the 9 toystore example manifests folded in
as citations). Explicitly deferred to a follow-up slice: DNS (`DNSPolicy`),
TLS (`TLSPolicy`), mTLS/x509 client-cert auth, OIDC federation,
observability/tracing, CEL extensions, the extension SDK, and egress
credential injection. New pages are `products: [kuadrant]` (upstream, no
Red Hat version) — see the topic → raw source map above for how this feeds
into MaaS/RHOAI inference-gateway work.
