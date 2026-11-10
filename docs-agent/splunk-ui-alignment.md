# Splunk UI alignment for the Python SDK docs POC

## Goal

Reuse documentation build and packaging ideas from the Splunk UI docs system where they fit, so this POC complements the sister team's work. Keep the POC's distinct objective: propose and draft useful conceptual and procedural documentation from engineering evidence, with human review for intent and unsupported claims.

## Candidate reuse

- **Build-time alternate output:** Evaluate a branch-only export of the existing Python SDK reference and reviewed narrative drafts as Markdown for LLM consumption, alongside the existing Sphinx HTML output.
- **Discovery index:** Evaluate emitting an `llms.txt` index that points to the Markdown pages and describes the docs sections. Treat this as navigation metadata, not generated guidance.
- **Source-controlled selection/configuration:** Keep any inclusion list, page metadata, and export options in the repo so reviewers can see exactly what enters the LLM bundle.
- **Same source, multiple renderings:** Prefer deriving HTML and LLM-friendly output from the same approved source pages where practical; do not let the LLM export become a second, drifting copy of content.
- **Reviewable build artifacts:** Produce the export only on this POC branch initially. Inspect its links, completeness, freshness, and whether Sphinx source or generated outputs are the safer export input.

## Fit boundary

The Splunk UI system's React component parser and React docs page components are specific to React packages. The Python SDK already has a Sphinx-based reference build, so adopting those React-specific parts would add a parallel parser and site framework without solving the POC's narrative-generation problem.

Before adding a new exporter, inspect the current Sphinx build and its existing extensions or outputs for reusable structured data and Markdown conversion options. If a suitable Splunk UI docs-gen helper is package-independent and can consume Python/Sphinx content without replacing the authoritative build, evaluate it with the Splunk UI maintainers. Otherwise, implement only the minimal adapter needed for this POC and document why.

## Acceptance checks for the experiment

1. Existing Sphinx HTML/reference output remains the authoritative SDK reference.
2. The optional LLM bundle is generated from source-controlled, approved docs inputs and includes a clear index; it does not publish automatically.
3. Every exported page has stable source links or provenance back to its source page.
4. Generated narrative drafts enter the bundle only after human review; draft status is visible.
5. The export does not copy secrets, internal-only material, or unapproved content.
6. Record what was reused, what was evaluated and rejected, and the evidence for those decisions so the approach can be explained to the Splunk UI partner team.

## Not in scope

This is a compatibility and packaging investigation, not a proposal to replace Splunk UI's docs system or the Python SDK's Sphinx reference generator. It does not assume that the Splunk UI team owns or supports a Python documentation exporter.
