# Consult-backed OCR delegation

**Status:** Superseded by the approved generic `pragmatic-review-advanced` design in [`Routed Engineering Checks v2`](../specs/routed-engineering-checks-v2.md). The named tools below are private implementation details, not the public skill identity.

## Purpose

Provide a review workflow where Open Code Review (OCR) performs deterministic review preparation while `consult-llm` supplies an independent reviewer model.

## Target users

Agents or engineers who want OCR file selection and rule resolution without configuring OCR with a separate LLM provider.

## Trigger

Use when a user requests a code review and wants the review intelligence to come from `consult-llm` rather than OCR's configured LLM endpoint.

## Boundary

OCR remains the local deterministic component:

- `ocr delegate preview` selects the reviewable files and review mode.
- `ocr delegate rule` resolves applicable rules.

`consult-llm` performs the review analysis from the selected diffs and rules. It still uses its configured model backend, which may require its own credentials or incur provider cost. This workflow does not make review local or free by itself.

## Core output

A coverage-checked review result that accounts for every file returned by OCR, with findings in the existing path, line-range, category, severity, and recommendation structure.

## Verification

- OCR delegation commands return the reviewable file list and rule groups without an OCR LLM endpoint.
- The `consult-llm` review receives the selected diffs and applicable rules.
- The final result accounts for each previewed file as reviewed or skipped with a reason.

## Resolution

The workflow belongs in `pragmatic-engineering` as the manual-only `pragmatic-review-advanced` skill. The public contract remains provider-neutral. The approved design resolves one explicit review model, validates complete file coverage, and requires final `pragmatic-review` evidence before reporting a finding.

## Source

- OCR delegation skill: https://github.com/alibaba/open-code-review/blob/main/plugins/open-code-review/skills/open-code-review-delegate/SKILL.md
