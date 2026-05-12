# Documentation project instructions

## About this project

- This is the DatologyAI customer documentation site.
- The site is built on [Mintlify](https://mintlify.com).
- Pages are MDX files with YAML frontmatter.
- Site configuration and navigation live in `docs.json`.
- Run `mint broken-links` after changing links or navigation.
- Run `mint validate` after structural changes, page deletion, or large edits.
- Run `mint dev` to preview locally when visual layout or navigation needs inspection.

## Working conventions

- Read `docs.json` before changing navigation or adding pages.
- Search existing docs before creating a new page.
- Prefer updating an existing page over duplicating similar content.
- Add any new page to `docs.json`; otherwise it will not appear in the sidebar.
- Use root-relative internal links without file extensions, such as `/curation/launch`.
- Keep examples customer-facing and runnable from the CLI runner unless noted.
- Preserve unrelated user changes in the working tree.

## Terminology

- Use `Datology` for the product and `DatologyAI` for the company or docs site.
- Use `customer` or `operator` when describing people running the platform.
- Use `CLI runner` for the EC2 instance used for Datology operations.
- Use `datology` for the customer-facing CLI command.
- Use `recipe` for supported curation templates.
- Avoid customer-facing use of `workflow entry point` or `launch plan` unless documenting low-level support context.
- Use `runtime version` or `version` for the version used to execute a curation.
- Use `dataset`, `curation`, `asset`, `Run ID`, and `Curation ID` consistently with `/concepts`.
- Use `distribution fitting`, not `mining`, in customer-facing recipe descriptions.
- Describe `ddb` as an advanced debug surface intended for use with Datology support.

## Style preferences

- Use active voice and second person.
- Keep sentences concise: one idea per sentence.
- Use sentence case for headings.
- Bold UI labels, for example: Click **Settings**.
- Use code formatting for file names, commands, paths, environment variables, and code references.
- Give every fenced code block a language tag.
- Use descriptive alt text for images and media.
- Avoid marketing language such as "powerful", "seamless", "robust", and "cutting-edge".
- Avoid filler phrases such as "it is important to note", "simply", and "just".

## Content boundaries

- Do not document internal local-release or unofficial deployment flows.
- Do not instruct customers to use `DATOLOGY_CONFIG_JSON`.
- Do not make post-install validation sound like a manual customer step; deployment runs validation and smoke checks.
- Keep runtime image and version details in upgrade or operations pages, not as a standalone customer path.
- Do not present `backfill` as a normal ingestion flow. It is mainly for Datology-supplied artifacts in known internal formats.
- Use dataset registration and `IngestionSource` for customer source data.
- Treat `--input-dataset-path` and `--input-schema` as deprecated for curation launches.
- Mention that `download-hf-dataset` stages files only; staged datasets still need registration or ingestion before use.
- Keep queue internals operator-scoped and describe changes as work done with Datology support.
- Keep Curation Studio admin/internal content hidden until it is publicly documented.
