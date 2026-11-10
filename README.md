# Developer docs synthesis POC

This is an independent repository for a proposal-first pipeline that turns software engineering changes into reviewable developer documentation drafts. It does not modify or publish the SDK reference site.

## Project boundaries

- This repository contains the agent runner, its configuration, workflow notes, and generated review artifacts when you choose to keep them locally.
- `sdk-docs-preview` remains its own project for SDK reference site proofs. Its main checkout and current site publishing path are independent of this repository.
- The Python SDK source checkout is an input only. Pass its local path to the collector; the runner reads source files and Git history and does not write to that checkout.
- The first pilot targets `splunklib.ai` and the `agentic_reporting_csc` example in `splunk/splunk-sdk-python`.
- Nothing in this POC publishes content, changes a deployed site, or updates generated reference files.

## Current workflow

1. Collect a bounded evidence bundle for a base-to-head source change.
2. Ask a model for a structured content proposal with claims, evidence citations, suggested page type, and questions for a human.
3. Review the proposal and fill in `answers.md`.
4. Generate a cited draft only after the human explicitly sets `Decision: proceed`.

The current runner is manually invoked. It does not yet poll commits, connect Jira or dev.splunk.com, run in GitHub Actions, or publish. See [`docs-agent/README.md`](docs-agent/README.md) for setup and commands, and [`docs-agent/splunk-ui-alignment.md`](docs-agent/splunk-ui-alignment.md) for where the POC can reuse Splunk UI documentation practices.

## Try it locally

Use a separate local clone of the Python SDK source. Install the optional API client in this repository's environment:

```sh
python -m venv .venv
source .venv/bin/activate
python -m pip install -r docs-agent/requirements.txt
```

Collect evidence (replace the checkout path and refs with the SDK change you want to examine):

```sh
python docs-agent/runner.py collect \
  --repo /path/to/splunk-sdk-python \
  --base origin/develop \
  --head HEAD \
  --output /tmp/sdk-docs-run
```

Review the evidence bundle before sending it to a model. Model calls require `OPENAI_API_KEY`, `DOCS_AGENT_MODEL`, and an explicit `--confirm-send` flag. Follow the remaining proposal and draft steps in the runner guide.

## Safety and review

The runner can send selected source diffs and context to the configured model provider. Inspect those files and the resulting `evidence.json` before confirming a model call. Collection uses best-effort credential-pattern redaction, not a full secret scanner. Treat model drafts as unverified editorial proposals. Human review remains required for product intent, technical correctness, examples, and release decisions.
