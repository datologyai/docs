# Documentation site plan

Proposed structure for `docs.datologyai.com` (Mintlify).

**Audience:** A technical customer operator or ML platform engineer who deploys Datology themselves in their own AWS account, configures access/storage/networking/runtime versions, launches and inspects curation runs, and decides when to use `datology-cli` vs. `ddb`.

**Scope:** Two outcomes — (1) deploy Datology, (2) use `datology-cli` (and `ddb` for debugging) to curate datasets.

**Source of truth for concepts:** [`universe/docs/public-documentation-concepts.md`](../universe/docs/public-documentation-concepts.md). The structure below maps 1-to-1 to the primitives, deployment expectations, and CLI command groups in that document.

Training, internal HOWTOs, science workflows, per-job internals, and the Curation Studio UI are out of scope for this first milestone.

---

## Public concepts (the canonical list)

These are the only customer-facing primitives. Every page should use this vocabulary; internal terms (e.g. "launch plan") only appear where the user has to type them verbatim.

- **Recipe** — a named, supported curation template. Defines source mixing, budgets, and constraints. Surfaced via `list-curation-recipes`.
- **Dataset** — customer-provided or Datology-produced data at an object-store path.
- **Curation** — a managed workflow run that applies a recipe to input data and writes curated output.
- **Asset** — a tracked output or intermediate dataset produced by a curation. Inspected via the CLI; data itself lives at object-store paths.
- **Workflow entry point** — a registered, versioned target the orchestration engine can execute. Curations launch against an entry point at a chosen version.
- **Run ID** — asset-tracker identifier for a curation run; used for dataset/asset metadata lookups in the CLI.
- **Curation ID** — execution identifier issued at launch by the orchestration engine; accepted by run-management commands and `ddb`. Distinct from Run ID; both are needed for full traceability.
- **Version** — the runtime image and registered workflow entry point version used to execute a curation.

---

## Structure

**One tab, "Documentation."** Groups & pages below.

### Getting started
- `index` — What Datology does (managed data curation for LLMs), who it's for, the deploy → curate flow, and an architecture diagram showing user-facing primitives on top and the system underneath (orchestration engine, Spark/Ray, object storage, asset tracker).
- `quickstart` — End-to-end: deploy a stack, configure the CLI, launch a small curation, inspect its status. Anchors back to the relevant deep pages.
- `concepts` — One page covering every primitive in the canonical list above, plus a short "system underneath" section (orchestration engine, Spark/Ray, object storage, asset tracker, CLI as the user-facing interface).

### Deployment

Answers: "What gets installed, what access is required, how do I validate it, and how do upgrades work?"

- `deploy/overview` — Two-layer install:
  - **Cloud infrastructure layer** — AWS resources (EKS cluster, networking, IAM, installer/runner infrastructure, release assets).
  - **Kubernetes product stack** — runtime services Datology needs (workflow orchestration engine, orchestration + asset-tracker databases, Spark operator, Ray support, scheduling/autoscaling, monitoring).
  - Map of **deployment outputs → CLI inputs**: CLI config file, orchestration-engine endpoint/config, storage locations, asset-tracker connection details, current runtime version. Restated on the CLI install page.
- `deploy/aws-prerequisites` — AWS account, profile, region, IAM permissions, quotas, VPC CIDR selection, customer subdomain.
- `deploy/network-and-access` — Network patterns we support: Tailscale, private access, no-external-network installs, public ingress. Each with when-to-use guidance and the deployment flag/variable that selects it.
- `deploy/storage-and-buckets` — Bucket layout, object-store paths, what each bucket holds, where customer-owned data lives vs. Datology-managed artifacts.
- `deploy/runtime-images-and-versions` — Stack templates and runtime images are versioned **independently**. How to find the deployed stack version, the deployed runtime version, and the version future curations will run against.
- `deploy/install` — Running the install (`deploy_datology.sh`) with the required environment variables; what to expect at each phase. *(source: `cloudformation/README.md`, `deploy_datology.sh`)*
- `deploy/post-install-validation` — Explicit checklist:
  1. Kubernetes access works.
  2. Workflow orchestration engine is reachable.
  3. Workflow entry points are registered.
  4. Asset tracker is reachable.
  5. A small test curation runs successfully.
- `deploy/upgrades` — Two upgrade flavors:
  - **Stack upgrades** change infrastructure or services.
  - **Runtime upgrades** change the images and workflow versions used by curations and are driven by `datology-cli upgrade-launchplans`. Frame as "registering new workflow entry points for a runtime version" — operator/admin action.
- `deploy/security-and-secrets` — Required IAM roles and policies; AWS credentials needed to install vs. operate; where Hugging Face / model-registry tokens live and how to rotate them.
- `deploy/observability` — Default monitoring installed by the stack; how to find Spark/Ray job logs; when to use `datology-cli` vs. `ddb`.
- `deploy/troubleshooting` — Install-time and post-install failure modes.

### CLI

Positions `datology-cli` as the current user-facing operational interface for curations and datasets.

- `cli/overview` — What the CLI is for and the command groups (curation lifecycle, dataset lifecycle, asset operations, recipes, runtime management, dependencies). Links to each section below.
- `cli/install-and-configure` — Where `datology-cli` lives (CLI runner) and what it needs:
  - `DATOLOGY_CONFIG_PATH` → the Datology config file.
  - Workflow orchestration engine config (so the CLI can submit and inspect runs).
  - Asset-tracker DB access for metadata-inspection commands.
  - Kubernetes access for commands that update deployed services.
  - Access to the configured model storage bucket for dependency commands.
  - Each value is a deployment output — links back to `deploy/overview`.
- `cli/datasets` — `add-dataset`, `list-datasets`, `describe-dataset`, `delete-dataset`.
  - **Important:** `add-dataset` is itself a workflow run (with a Run ID and possible failure modes), not a synchronous record write. Customers must wait for and inspect the registration run before the dataset is curatable.
  - `delete-dataset` marks a dataset run as purged and can terminate an in-flight workflow run, but does **not** delete stored data.
  - Supported input schemas enumerated here: `dclm`, `dclm_parquet`, `c4`.
- `cli/curations` — `new-curation`, `list-curations`, `show-curation`, `cancel-curation`.
  - `new-curation` launches a registered workflow entry point and returns a Curation ID + output location.
  - `--version` selects the workflow + runtime image version; defaults to the configured default version.
  - `--pretty-preview` and `--full-json-preview` validate inputs without launching.
  - Extra workflow arguments via `--extra-curation-args-file` / `--extra-curation-args-string`.
  - One-curation-at-a-time policy; `ALLOW_PARALLEL_CURATION_RUNS` to override.
  - `show-curation` displays overall status and task-level progress.
  - Supported output formats enumerated here: `parquet`, `mds`, `jsonl`, `lit`.
- `cli/recipes` — Standalone short page:
  - What a recipe is and why it's the primary unit of curation.
  - The publicly supported recipes (`web_multilingual_stew`, `stew_001`) with inputs, outputs, and intended use.
  - How to choose between recipes; how `list-curation-recipes` surfaces them.
  - Recipe / runtime-version compatibility notes.
  *(source: `cli/datology_cli/recipes.py`)*
- `cli/inspect-datasets-and-assets` — Asset-side commands:
  - `info <s3_path>` — look up the run that produced an asset.
  - `backfill` — register an existing customer-owned object-store path as a known asset so subsequent curations can reuse or reference it.
- `cli/versioning` — `--version` flag behavior, configured default version, `latest` resolution. What happens when a recipe isn't registered at the requested version. `upgrade-launchplans` as an operator/admin action that registers workflow entry points for a runtime version (cross-link to `deploy/upgrades`).
- `cli/dependencies` — Staging models, tokenizers, and Hugging Face datasets into customer-accessible storage:
  - `register-tokenizer`
  - `register-model`
  - `download-hf-dataset`
- `cli/debugging-with-ddb` — Short page:
  - What `ddb` (Datology Debugger) is for: introspecting curation runs, tasks, and assets when `show-curation` isn't enough.
  - When to reach for it vs. `datology-cli show-curation`.
  - Pointer to its built-in `--help` for full reference.

### Reference
- `reference/cli` — Full inventory of every `datology-cli` command and every flag, with **public** vs. **admin/operator** labels. Generated from `cli/datology_cli/main.py`.
- `reference/datology-config` — Every field in `datology_config.json` with type, default, and effect.
- `reference/glossary` — The canonical concept list, with cross-links into the guides.

### Release notes
- `release-notes/changelog`

---

## Out of scope (for this milestone)

- Anything training-related (no `train/`, no checkpoint conversion, no eval framework).
- Internal monorepo dev workflow: PR bot, lint, PyCharm, `lcp.sh`, `git_package_overrides.txt`, science repos.
- Internal-only deploy flows: unofficial stack/runtime releases, `deploy_parent_stack.sh` from local changes, dev AWS account assumptions.
- Per-job and per-workflow internals (`pipeline/jobs/*`, `flyte/datology/workflows/*`).
- Curation Studio UI documentation — deferred; the concepts doc treats the CLI as the current user-facing interface.

---

## Open questions to resolve before publishing

Carried forward from `public-documentation-concepts.md`:

**Deployment**
- Which deployment paths should be public — released stack only, or also a simplified customer-only flow?
- Which network patterns should be documented — Tailscale, private access, no-external-network installs, public ingress, or all of them?
- What exact customer-facing validation command should be the canonical smoke test?

**CLI**
- Which commands are customer-facing, and which should be hidden in an operator/admin reference?
- Which workflow entry points should appear in public examples?
- Should public examples use raw `new-curation` commands or recipe-first commands?
- What is the canonical way for customers to obtain the config files and credentials produced by deployment?

---

## Implementation approach

Aligned to the "Suggested First Docs Milestone" in the concepts doc:

1. **Update `docs.json`** to the tab/group/page tree above.
2. **Delete the Mintlify starter content:** `essentials/`, `agent-ready/`, `ai-tools/`, `api-reference/`.
3. **Write the four anchor pages first** to lock voice + components (Cards, Tabs, Accordions, CodeGroups):
   - `index`
   - `concepts`
   - `deploy/overview` + `deploy/install`
   - `cli/install-and-configure` + `cli/curations`
4. **Backfill the rest in milestone order:**
   1. Overview (`index`, `concepts`).
   2. Deployment pages.
   3. CLI quickstart (`quickstart`, `cli/install-and-configure`).
   4. Dataset guide (`cli/datasets`).
   5. Curation guide (`cli/curations`).
   6. Recipes guide (`cli/recipes`).
   7. Versioning guide (`cli/versioning`, plus `deploy/runtime-images-and-versions`).
   8. Debugging with `ddb` (`cli/debugging-with-ddb`).
   9. CLI reference (`reference/cli`).
5. **Generate the CLI reference page** by enumerating every `@cli.command(...)` in `cli/datology_cli/main.py` and labeling each command + flag as public or admin/operator. (One-time scrape now; later we can wire up an autogen script.)
6. **Wire `snippets/`** for the env-var preamble that appears before nearly every CLI invocation:
   ```bash
   export DATOLOGY_CONFIG_PATH=config/config.json
   export FLYTECTL_CONFIG=flytectl-remote-config.yaml
   ```
