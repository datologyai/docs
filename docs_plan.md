# Datology public docs plan

This plan defines the first external customer documentation site for the
Mintlify project in `docs/`. It is based on:

- `universe/docs/public-documentation-concepts.md`
- `universe/README.md`
- `universe/cloudformation/README.md`
- `universe/cloudformation/deploy_datology.sh`
- `universe/cloudformation/help_utils.sh`
- `universe/cloudformation/templates/*.yaml`
- `universe/infra/k8s_configs/product_stack_template/scripts/*`
- `universe/src/python/flyte/cli/datology_cli/main.py`
- `universe/src/python/flyte/cli/datology_cli/recipes.py`
- `universe/src/python/flyte/cli/README.md`
- `universe/src/python/flyte/ddb_cli/main.py`
- `universe/src/python/flyte/datology/K8S.md`
- `universe/src/python/flyte/common/datology_flyte_commons/config.py`
- `universe/src/python/pipeline/pipeline/asset_tracking/README.md`

The docs should be written for technical customer operators and ML platform
engineers who deploy Datology into their own AWS account, operate the stack,
register datasets, launch curations, inspect outputs, and debug runs.

## Product story

Datology is a customer-deployed data curation system for large-scale model
training datasets. The public docs should explain a single operational loop:

1. Deploy Datology in your AWS account.
2. Configure access, storage, runtime versions, and the CLI runner.
3. Register input datasets.
4. Choose a supported recipe.
5. Preview, launch, monitor, and cancel curations.
6. Inspect curated outputs and asset lineage.
7. Upgrade runtimes and troubleshoot jobs when needed.

The first version should not document internal research workflows, monorepo
development, unofficial release flows, local developer setup, training jobs,
asset-promotion internals, or Curation Studio UI workflows.

## Canonical customer concepts

Use these terms consistently across every page.

- Recipe: a named, supported curation template, surfaced by `list-curation-recipes`.
- Dataset: customer-provided or Datology-produced data in object storage.
- Curation: a managed workflow run that applies a recipe to input data and writes curated output.
- Asset: a tracked input, output, or intermediate dataset recorded in the asset tracker.
- Workflow entry point: a registered, versioned execution target used by curations.
- Run ID: asset-tracker identifier used for metadata and lineage lookups.
- Curation ID: orchestration execution identifier used by run management and `ddb`.
- Version: runtime image tag plus registered workflow entry point version.

When a page must mention Flyte, Spark, Ray, Kubernetes, or the asset tracker,
frame them as implementation components under these customer-facing concepts.

## Recommended Mintlify structure

Use one primary tab, `Documentation`, with groups organized by customer task.
The starter-kit `API reference`, `essentials`, `agent-ready`, and `ai-tools`
content should be removed or replaced before publishing.

```text
Documentation
  Get started
    index
    quickstart
    concepts
    architecture
  Deploy Datology
    deploy/overview
    deploy/aws-prerequisites
    deploy/network-and-access
    deploy/storage-and-buckets
    deploy/compute-and-scheduling
    deploy/runtime-images-and-versions
    deploy/install
    deploy/cli-runner
    deploy/post-install-validation
    deploy/upgrades
    deploy/security-and-secrets
    deploy/observability
    deploy/troubleshooting
  Curate data
    curation/overview
    curation/recipes
    curation/datasets
    curation/preview
    curation/launch
    curation/monitor-and-cancel
    curation/outputs-and-assets
    curation/dependencies
  Operate
    operate/versioning
    operate/debugging-with-ddb
    operate/job-logs
    operate/common-failures
  Reference
    reference/cli
    reference/datology-config
    reference/deployment-environment-variables
    reference/glossary
  Release notes
    release-notes/changelog
```

If the site needs to stay smaller for the first launch, combine `Curate data`
and `Operate` into a single `CLI` group. Keep the slugs above so pages can be
split later without changing the conceptual model.

## Page catalog

### `index`

Purpose: orient a new customer in one screen.

High-level content:

- What Datology does: customer-operated dataset curation for model training data.
- The four primary tasks: deploy, register datasets, run curations, inspect outputs.
- A simple system diagram: CLI and recipes on top; workflow orchestration, Spark/Ray,
  object storage, and asset tracker underneath.
- Cards to `quickstart`, `deploy/overview`, `curation/recipes`, and `reference/cli`.

### `quickstart`

Purpose: give a short end-to-end path once the customer has credentials.

High-level content:

- Prerequisites summary with links to full deployment pages.
- Deploy the released stack.
- SSH or connect to the CLI runner.
- Confirm `datology --help`.
- Run `datology list-curation-recipes`.
- Register a small dataset or use the deployment smoke-test dataset.
- Preview a curation.
- Launch the curation.
- Inspect status with `datology show-curation`.
- Find output and lineage with `datology info`.

Open decision: the canonical smoke-test dataset and command need product approval.

### `concepts`

Purpose: create the shared vocabulary for the rest of the docs.

High-level content:

- Define recipe, dataset, curation, asset, workflow entry point, Run ID,
  Curation ID, and version.
- Explain the difference between Run ID and Curation ID.
- Explain where data lives versus where metadata lives.
- Explain that the CLI is the current supported customer interface.

### `architecture`

Purpose: explain what runs after Datology is deployed without exposing internal
repository structure.

High-level content:

- Cloud infrastructure layer: VPC/networking, EKS, IAM, DNS/certificates,
  installer and CLI runner, S3 buckets, ECR access.
- Kubernetes product stack: Flyte, CloudNativePG databases, asset tracker,
  Spark Operator, KubeRay, Karpenter, YuniKorn, Kueue, monitoring, AWS load
  balancer controller.
- Runtime path for a curation: CLI command -> workflow entry point -> Spark/Ray
  jobs -> object storage -> asset tracker metadata.
- Boundaries: Datology runs in the customer AWS account; Datology can help
  scope and troubleshoot, but the customer operates the stack.

### `deploy/overview`

Purpose: explain the install model and what gets created.

High-level content:

- Datology deploys two layers: CloudFormation infrastructure and Kubernetes
  product stack.
- Released deployment path uses `deploy_datology.sh`; local and unofficial
  release flows should not be in public docs.
- Deployment outputs needed later: stack name, EKS cluster name, CLI runner,
  `datology_config.json`, Flyte config, `ASSET_TRACKER_PROD_URI`, storage
  buckets, runtime image tag, and workflow entry point version.
- Link to every deployment page in the expected order.

Source notes:

- `cloudformation/deploy_datology.sh`
- `cloudformation/templates/parent_stack.yaml`
- `infra/k8s_configs/product_stack_template/scripts/_deploy_product_stack.sh`

### `deploy/aws-prerequisites`

Purpose: list what the customer needs before running the installer.

High-level content:

- AWS account and region.
- AWS CLI profile with CloudFormation, EKS, EC2, IAM, S3, ECR, Route53, ACM,
  Secrets Manager, and related permissions.
- Route53 hosted zone or customer subdomain.
- SSH public key via `DEPLOYMENT_PUBLIC_KEY`.
- VPC CIDR choice for new-network installs.
- S3 access to Datology release artifacts.
- Optional existing VPC/EKS information.
- Optional Tailscale auth key, depending on selected access pattern.

### `deploy/network-and-access`

Purpose: let operators choose and validate the correct network pattern.

High-level content:

- New VPC versus existing VPC.
- New EKS cluster versus existing EKS cluster.
- CLI runner public IP versus private-only access.
- `HAS_EXTERNAL_NETWORK_ACCESS=true/false`.
- Tailscale/router access where supported.
- Required CIDR variables: `VPC_SLASH_16_CIDR_BLOCK`, `ALLOWED_SSH_CIDR`,
  `ALLOWED_EKS_API_CIDR`.
- VPC endpoints used for no-external-network installs.
- What changes for customer firewalls, VPNs, and private subnets.

Open decision: decide which access patterns are officially supported for
external docs on day one.

### `deploy/storage-and-buckets`

Purpose: explain Datology's storage layout and customer data responsibilities.

High-level content:

- Asset buckets by domain: development, staging, production.
- Job logs bucket.
- Flyte code artifacts bucket.
- Models/artifacts bucket.
- Customer-provided buckets via `ARTIFACTS_BUCKET_NAME`,
  `ADDITIONAL_READ_ONLY_BUCKET_ARNS`, and `ADDITIONAL_READ_WRITE_BUCKET_ARNS`.
- What the installer creates versus what the customer supplies.
- Data retention expectations and the fact that `delete-dataset` does not
  delete stored objects.

### `deploy/compute-and-scheduling`

Purpose: explain compute shape without turning the page into internal scheduler docs.

High-level content:

- Spark and Ray are the execution frameworks.
- Node pools define available compute.
- Spark uses YuniKorn queues.
- Ray uses Kueue queues.
- Karpenter can provision node pools when enabled; static node groups are also supported.
- Cluster config controls node pools, Spark limits, Ray limits, and queue capacity.
- When customers should ask Datology to adjust compute sizing.

Source notes:

- `cloudformation/configs/README.md`
- `infra/k8s_configs/product_stack_template/scripts/generate_job_queue_config.py`

### `deploy/runtime-images-and-versions`

Purpose: clarify stack version versus runtime version.

High-level content:

- Stack version: CloudFormation templates and install assets.
- Runtime image tag: Spark, Ray, Flyte worker, and DB admin images used by curations.
- Default runtime version is written into generated `datology_config.json`.
- `datology version` shows CLI build metadata.
- `datology upgrade-launchplans` registers workflow entry points for a runtime version.
- How to identify the version a curation used.

### `deploy/install`

Purpose: document the released installation path.

High-level content:

- Download or use released deploy scripts.
- Required environment variables:
  `AWS_PROFILE`, `DEPLOYMENT_PUBLIC_KEY`, `CUSTOMER_NAME_PREFIX`,
  and `VPC_SLASH_16_CIDR_BLOCK` when creating a new VPC.
- Optional environment variables:
  `DATOLOGY_STACK_VERSION`, `RUNTIME_CONTAINER_IMAGE_TAG`,
  `HOSTED_ZONE_NAME`, `CREATE_NEW_NETWORK_STACK`, `CREATE_NEW_EKS_CLUSTER`,
  `CLI_RUNNER_HAS_PUBLIC_IP`, `HAS_EXTERNAL_NETWORK_ACCESS`, `INSTALL_DDB`,
  `DEPLOY_CURATION_STUDIO`, and bucket/permission overrides.
- Run `./deploy_datology.sh`.
- Expected phases: template lookup, network validation, stack creation,
  Helm installer, product stack deployment, CLI runner setup.
- Where install logs are written.

Keep internal sections on unofficial releases and local stack releases out of
this public page.

### `deploy/cli-runner`

Purpose: explain the customer operational host and how the CLI is installed.

High-level content:

- CLI runner is an EC2 instance with Dockerized setup, AWS tools, `kubectl`,
  `flytectl`, `psql`, `datology`, and optional `ddb`.
- `datology` wraps `/home/ubuntu/datology-cli.pex`.
- `ddb` wraps `/home/ubuntu/ddb-cli.pex` when `INSTALL_DDB=true`.
- Generated config paths:
  `/home/ubuntu/.config/datology_config.json`,
  `/home/ubuntu/.config/flyte_config.yaml`,
  `/home/ubuntu/.kube/config`.
- Environment variables added to `/etc/bash.bashrc`:
  `DATOLOGY_CONFIG_PATH`, `FLYTECTL_CONFIG`, `ASSET_TRACKER_PROD_URI`,
  and `KUBECONFIG`.
- How to reconnect to the runner and re-run validation.

### `deploy/post-install-validation`

Purpose: give a checklist to decide whether deployment succeeded.

High-level content:

- CloudFormation stack is complete.
- EKS access works: `kubectl get nodes`.
- Product stack services are present.
- Flyte endpoint is reachable.
- Asset tracker database is reachable.
- `datology --help` works on the CLI runner.
- `datology list-curations` can query orchestration and asset tracker state.
- Workflow entry points are registered.
- Flyte smoke test completed.
- Optional small curation smoke test.

Source notes:

- `infra/k8s_configs/product_stack_template/scripts/run_flyte_smoke_test.sh`
- `infra/k8s_configs/product_stack_template/scripts/_run_cli_operations.sh`

### `deploy/upgrades`

Purpose: separate infrastructure upgrades from runtime upgrades.

High-level content:

- Stack upgrades change CloudFormation resources or Kubernetes services.
- Runtime upgrades change image tags and registered workflow entry points.
- `datology upgrade-launchplans --version <version>` is the customer-facing
  operator action for runtime registration.
- Include required access: asset tracker DB URI, Kubernetes access, ECR access.
- Explain reactivating cron launch plans at the new version.
- Call out Curation Studio image upgrades as optional/admin until the UI is public.

### `deploy/security-and-secrets`

Purpose: document minimum security posture and where credentials live.

High-level content:

- IAM roles and broad permission categories for installer, Helm installer,
  CLI runner, Spark jobs, Ray jobs, Flyte workers, and indexed jobs.
- Permission boundary support.
- Additional bucket access configuration.
- Secrets Manager usage for database credentials.
- Hugging Face/model registry token storage and rotation.
- SSH access and allowed CIDRs.
- No-external-network considerations.

Open decision: identify the exact customer-facing secret names and rotation
procedure before publishing.

### `deploy/observability`

Purpose: show operators where to see system health and run-level evidence.

High-level content:

- Monitoring stack and Grafana entry point.
- Spark History endpoint.
- Spark driver and executor logs.
- Ray dashboard for running Ray jobs.
- S3 job log locations.
- Asset tracker metadata for run/task state.
- When to use `datology show-curation`, `datology info`, `ddb`, `kubectl logs`,
  and Grafana.

### `deploy/troubleshooting`

Purpose: give first-response diagnosis for install and validation failures.

High-level content:

- S3 release bucket access failure.
- Hosted zone not found.
- VPC CIDR conflict.
- EKS access denied.
- CLI runner setup failure.
- Helm installer failure.
- Product stack service not ready.
- Asset tracker URI unavailable.
- Launch plans missing for a runtime version.
- No-external-network failures caused by missing endpoints.
- Links to log locations and `SETUP_ONLY` resume flow.

### `curation/overview`

Purpose: introduce the task-oriented curation workflow.

High-level content:

- Register or reference data.
- Choose a recipe.
- Preview.
- Launch.
- Monitor.
- Inspect assets and lineage.
- Debug when needed.
- Explain that all task pages use the `datology` wrapper on the CLI runner,
  even though the executable package is `datology-cli`.

### `curation/recipes`

Purpose: make recipes the primary way customers choose a curation.

High-level content:

- What a recipe is.
- How recipes map to workflow entry points.
- `datology list-curation-recipes`.
- Public recipes from current CLI:
  `web_multilingual_stew` and `stew_001`.
- For each recipe: intended use, launch plan, token range, mixing sources,
  rephrasing models, required inputs, outputs, and version compatibility.
- Guidance on choosing a recipe.

Source notes:

- `src/python/flyte/cli/datology_cli/recipes.py`

### `curation/datasets`

Purpose: document dataset registration and lifecycle.

High-level content:

- What qualifies as a dataset.
- Required object storage path permissions.
- `datology add-dataset`.
- `add-dataset` launches a workflow run; it is not a synchronous metadata write.
- `datology list-datasets`.
- `datology describe-dataset <run_id>`.
- `datology delete-dataset <run_id>`.
- Domain filters: `web` and `target`.
- Dataset status and ingestion summary.
- Dataset types shown by the CLI: Parquet, JSONL, Unknown.

### `curation/preview`

Purpose: encourage validation before launching costly runs.

High-level content:

- `--pretty-preview` for human-readable validation.
- `--full-json-preview` for full resolved workflow parameters.
- Required and optional fields for preview.
- Version and recipe validation.
- How extra curation args are merged from file and JSON string.
- What preview checks do and do not guarantee.

### `curation/launch`

Purpose: show the supported launch path for curations.

High-level content:

- `datology new-curation`.
- Required output path.
- Input dataset path and input schema.
- Supported input schemas: `dclm`, `dclm_parquet`, `c4`.
- Output formats: `parquet`, `mds`, `jsonl`, `lit`.
- `--curation-ratio`.
- `--version`.
- `--launch-plan` only where needed; prefer recipe-first examples when possible.
- One curation at a time by default.
- `ALLOW_PARALLEL_CURATION_RUNS=true` as an operator override.
- Returned Curation ID and output folder.

Open decision: decide whether public examples should use raw launch plans or a
recipe-first command shape.

### `curation/monitor-and-cancel`

Purpose: document normal run management.

High-level content:

- `datology list-curations`.
- `--limit` and `--show-cron-jobs`.
- `datology show-curation <curation_id>`.
- Overall status values: pending, running, completed, failed, cancelled, not found.
- Task-level progress values: not started, in progress, completed, reused, failed.
- `datology cancel-curation <curation_id>`.
- What can and cannot be cancelled.

### `curation/outputs-and-assets`

Purpose: explain what was produced and how to trace it.

High-level content:

- Output path from `new-curation`.
- Asset tracker record for produced assets.
- `datology info <s3_path>`.
- `--verbose` to show workflow inputs.
- `datology backfill` for existing customer-owned object-store paths.
- Relationship between asset location, Run ID, Curation ID, and execution version.
- Reuse and lineage at a conceptual level.

### `curation/dependencies`

Purpose: document staging external dependencies into customer-accessible storage.

High-level content:

- Why models, tokenizers, and Hugging Face datasets need staging.
- `datology register-tokenizer`.
- `datology register-model`.
- `datology download-hf-dataset`.
- Models bucket default from `datology_config.json`.
- Supported synthetic/rephrasing models:
  `olmo2-7b`, `phi4`, `qwen3-8b`, `mistral-7b-v3`.
- Hugging Face token requirements and rotation link.

### `operate/versioning`

Purpose: give operators a single place for runtime-version behavior.

High-level content:

- `default_version` in `datology_config.json`.
- `--version` on commands.
- `latest` resolution behavior.
- Missing launch plan behavior and prompted registration.
- `--auto-upgrade` on `new-curation`.
- `datology upgrade-launchplans`.
- How to verify a recipe is registered at the requested version.
- How to find the runtime used by a completed curation.

### `operate/debugging-with-ddb`

Purpose: explain when to leave normal CLI output and use the debugger.

High-level content:

- `ddb` is the Datology Debugger.
- Input is the Curation ID / orchestration execution ID.
- Options: `--flyte-domain`, `--spark-ui-local-port`, `--db-uri`, `--logs-prefix`.
- Use it when `show-curation` lacks enough task, Spark, Ray, or log detail.
- Running jobs: open Spark UI or Ray dashboard with port forwarding.
- Completed jobs: inspect S3 log artifacts and pod logs.
- Point to `ddb --help` for command details.

### `operate/job-logs`

Purpose: explain where runtime evidence lives.

High-level content:

- Install logs: `/var/log/helm-installer.log`,
  `/var/log/cli-runner.log`, `/var/log/cli-runner-docker-install.log`.
- Logs copied to `s3://<environment>-datologyai-job-logs/...`.
- Spark driver/executor logs.
- Ray worker/head logs.
- Kubernetes pod logs.
- How `ddb` discovers related resources.

### `operate/common-failures`

Purpose: document run-time failure patterns separately from install failures.

High-level content:

- Missing or inaccessible input S3 path.
- Unsupported input schema or output format.
- Recipe not registered for selected version.
- Asset tracker connection failure.
- Flyte endpoint/config failure.
- Kubernetes access failure for admin actions.
- Spark scheduling failure.
- Ray resource exhaustion.
- Existing curation blocks launch due to one-at-a-time policy.
- How to collect enough context for Datology support.

### `reference/cli`

Purpose: provide complete command inventory with public/admin labels.

High-level content:

- `new-curation`
- `add-dataset`
- `list-curations`
- `list-datasets`
- `list-curation-recipes`
- `show-curation`
- `describe-dataset`
- `info`
- `cancel-curation`
- `delete-dataset`
- `backfill`
- `download-hf-dataset`
- `version`
- `upgrade-launchplans`
- `upgrade-curation-studio` as hidden/admin until Curation Studio is public
- `register-tokenizer`
- `register-model`

Each command should include purpose, syntax, options, examples, required
configuration, and whether it is everyday, operator/admin, or support/debug.

### `reference/datology-config`

Purpose: document customer-visible configuration fields.

High-level content:

- `default_version`.
- `storage_config`: domain buckets, job logs bucket, code artifacts bucket,
  models bucket.
- `image_config`: ECR base URL, registry account ID, Spark image, Flyte worker
  image, Ray image.
- `kubernetes_cluster_config`: local cluster name, Spark config, Ray config.
- `scheduler_config`: job queue mapping and job queues at a conceptual level.
- Config resolution order:
  `DATOLOGY_CONFIG_JSON` internal wire state, `DATOLOGY_CONFIG_PATH`, then
  `./config/config.json`.
- Public docs should tell users to set `DATOLOGY_CONFIG_PATH`, not
  `DATOLOGY_CONFIG_JSON`.

### `reference/deployment-environment-variables`

Purpose: provide an operator reference for install-time variables.

High-level content:

- Required variables.
- Optional variables.
- Existing network variables.
- Existing EKS variables.
- Bucket access variables.
- Runtime image variables.
- Curation Studio variables marked optional/admin.
- Defaults and allowed values from `deploy_datology.sh` and `help_utils.sh`.

### `reference/glossary`

Purpose: short definitions and cross-links.

High-level content:

- Canonical concepts from this plan.
- Selected implementation terms customers will see in logs:
  Flyte, Spark, Ray, Karpenter, YuniKorn, Kueue, EKS, asset tracker, CLI runner,
  launch plan/workflow entry point.

### `release-notes/changelog`

Purpose: give customers a place to see meaningful changes.

High-level content:

- Stack release changes.
- Runtime release changes.
- CLI changes.
- Known breaking changes.
- Required operator actions after upgrade.

## First publishing milestone

The smallest useful external docs set is:

1. `index`
2. `quickstart`
3. `concepts`
4. `deploy/overview`
5. `deploy/aws-prerequisites`
6. `deploy/install`
7. `deploy/cli-runner`
8. `deploy/post-install-validation`
9. `curation/recipes`
10. `curation/datasets`
11. `curation/preview`
12. `curation/launch`
13. `curation/monitor-and-cancel`
14. `curation/outputs-and-assets`
15. `operate/versioning`
16. `operate/debugging-with-ddb`
17. `reference/cli`
18. `reference/deployment-environment-variables`
19. `reference/glossary`

The remaining pages can be added as soon as the primary flows are accurate.

## Suggested implementation sequence

1. Replace the starter `docs.json` navigation with the structure above.
2. Remove starter-only pages from navigation first; delete files after real
   replacements exist.
3. Create the core pages in this order:
   `index`, `concepts`, `quickstart`, `deploy/overview`, `curation/overview`.
4. Write deployment pages from released deployment scripts only.
5. Write curation pages from `datology_cli/main.py` and `recipes.py`.
6. Generate or manually scrape the first `reference/cli` page from Click
   decorators, then mark commands public/admin/support.
7. Add snippets for repeated CLI environment context:

```bash
export DATOLOGY_CONFIG_PATH=/home/ubuntu/.config/datology_config.json
export FLYTECTL_CONFIG=/home/ubuntu/.config/flyte_config.yaml
export ASSET_TRACKER_PROD_URI="<from deployment>"
export KUBECONFIG=/home/ubuntu/.kube/config
```

8. Add a release-note page once there is a public stack/runtime versioning
   policy for customers.

## Open publication decisions

- Which network patterns are supported publicly on day one: Tailscale,
  private access, no-external-network installs, public ingress, or a subset.
- The canonical smoke-test dataset and curation command.
- Whether public examples should show raw `new-curation --launch-plan ...`
  or a recipe-first command shape.
- Which commands are public, operator/admin, or hidden.
- Exact customer procedure for getting and rotating Hugging Face/model tokens.
- Whether Curation Studio should be documented now, hidden as admin-only, or
  deferred entirely.
- Whether `datology` or `datology-cli` should be the primary command name in
  examples. The CLI package is `datology-cli`; deployed customers appear to
  use the `datology` wrapper on the CLI runner.
