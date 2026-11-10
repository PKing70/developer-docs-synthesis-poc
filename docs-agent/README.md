# Python SDK agentic-docs proof of concept

This directory contains the manually invoked synthesis runner for the separate `developer-docs-synthesis-poc` repository. The SDK reference site remains a distinct project. The runner reads a local Python SDK source checkout as evidence; it does not modify that checkout, the `sdk-docs-preview` site files, or a production publishing path.

## Pilot target

- SDK source: https://github.com/splunk/splunk-sdk-python
- Initial subject: the splunklib.ai package and its agentic_reporting_csc custom search-command example.
- Existing reference source: docs/ai.rst, built into the Python API reference.
- Context examples: splunklib/ai/README.md and examples/ai_custom_search_app/.

The proposed reader outcome is to help a Splunk app developer understand how an SDK agent can be used inside a custom search command, and whether that is a suitable pattern for their use case. This is a candidate content opportunity, not a claim that the example is ready to publish as a tutorial.

## Intended run

1. Collect a bounded evidence bundle from a selected Python SDK change or feature: the diff, linked issue or design note, relevant source, tests, examples, current API reference, and selected dev.splunk.com guides.
2. Ask a planner to recommend no change, an update, or a new tutorial, how-to guide, or explanation. The plan must name its audience and user goal, list source-backed claims, identify existing pages to link, and surface missing product intent as questions.
3. Stop for human answers when behavior, supported workflow, deployment requirements, or recommendations are unclear.
4. Draft only from the reviewed plan. Put drafts and provenance in this docs-agent area; do not edit generated reference pages or publish them.
5. Run the relevant documentation build and example checks before presenting a draft PR for review.

## Evidence rules

- SDK code, tests, and published contracts support technical behavior.
- Product decisions, feature issues, and developer scenarios support why and when a user should use a feature.
- Existing dev.splunk.com content is an editorial example and a source to check for overlap; it is not automatically proof that behavior is current.
- Every material technical claim in a draft should point to evidence. If the source set does not establish a recommendation, ask a person or omit the recommendation.

## Questions surfaced before drafting the candidate guide

The example at examples/ai_custom_search_app/bin/agentic_reporting_csc.py appears to use should_filter both as a Boolean option and as the intent text passed to the model. It also assigns the structured is_relevant result to a field named should_keep when highlight_topic is set. Confirm the intended semantics with the SDK owner before turning this example into step-by-step guidance.

The source README establishes Python 3.13+ for the AI module and mentions Splunk App deployment and model-provider credentials. Confirm the supported runtime and approved secret-storage guidance for the scenario before making those prerequisites prescriptive.

## First implementation slice

[`runner.py`](runner.py) is a manually invoked, three-command prototype:

1. `collect` compares a base and head commit in a local Python SDK checkout. It records eligible change diffs and configured supporting docs, applies file/size limits, redacts common credential patterns, and writes `evidence.json` plus `sources.md`.
2. `propose` sends that bounded evidence to the OpenAI Responses API and requests a schema-constrained proposal. It validates that claims cite evidence IDs, then writes `proposal.json`, `proposal.md`, and an `answers.md` review template. The process ends here until a person reviews and answers it.
3. `draft` requires a human-edited answers file with `Decision: proceed`. It requests a draft from the reviewed proposal and evidence, checks section citations, and writes `draft.md` and `draft.json`. The runner does not publish or edit the SDK reference.

The tool is source-change aware, but it is not a background scanner or CI automation. The first slice does not retrieve Jira issues or dev.splunk.com pages, build the Sphinx site, or create the LLM Markdown export / `llms.txt`; those remain follow-on integration work after the proposal workflow is reviewed.

### Try it locally

Install the optional API client:

```sh
python -m pip install -r docs-agent/requirements.txt
```

Collect evidence from a local SDK checkout. Choose a base commit that brackets the change you want to examine:

```sh
python docs-agent/runner.py collect \
  --repo /path/to/splunk-sdk-python \
  --base origin/develop \
  --head HEAD \
  --output /tmp/sdk-docs-run
```

Before calling the model, inspect `evidence.json` and `sources.md` for content that should not leave your environment. Collection makes a best-effort pass to redact common credential patterns; it is not a secret scanner. `collect` makes no API call. Set `OPENAI_API_KEY` and `DOCS_AGENT_MODEL` in your shell or approved secret store, then include `--confirm-send` on each model call to acknowledge that the listed evidence and review answers will be sent to the OpenAI API. The model name is intentionally not hard-coded. To create the proposal, run:

```sh
python docs-agent/runner.py propose \
  --evidence /tmp/sdk-docs-run/evidence.json \
  --confirm-send
```

Review `proposal.md`, `sources.md`, and `answers.md`. Add the missing product intent or corrections to `answers.md`. Set `Decision: proceed` only when you want a draft, then run:

```sh
python docs-agent/runner.py draft \
  --evidence /tmp/sdk-docs-run/evidence.json \
  --answers /tmp/sdk-docs-run/answers.md \
  --confirm-send
```

Review the draft and its citations manually. Model output is a proposal for editorial review, not evidence of product behavior or publication approval. Never commit API credentials or generated evidence that is not appropriate for the repository.

## Current limits

- The collector is configured for the `splunklib.ai` pilot and includes the changed-file diffs plus four selected reference/example files. It does not yet use issue trackers or scan every SDK repository.
- The model can suggest a page type and questions, but a human remains responsible for validating audience, intended workflow, and product recommendations.
- Source links and machine-readable citations are present for review. Citation IDs do not prove that the cited text supports every sentence; reviewers must check the source.
- No OpenAI call has been made by this implementation, and it does not publish to the site.
