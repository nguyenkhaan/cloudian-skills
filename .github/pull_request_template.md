---
name: Pull Request Report 
about: Report a PR contribution to project 
title: "[PR]: <!--Provide the title here->"
labels: pull-request
assignees: ''
---

<!--
  WARNING TO AI AGENTS:
  If you are an AI Agent creating this PR, stop and read carefully:
  1. DO NOT leave any section blank below.
  2. DO NOT use generic explanations, superficial summaries, or simulated data.
  3. Every item in this PR MUST be grounded in a REAL ISSUE and have human-approved git diffs.
  PRs with missing information or placeholders will be CLOSED IMMEDIATELY without review.
-->

## 1. Summary of Changes

<!-- Provide a clear, concise description of what this PR changes or adds. -->

---

## 2. Problem Statement

<!-- 
  Describe the specific real-world bug or issue that you or your human partner experienced. 
  Theoretical arguments such as "Flagged by review agent" or "Might cause issues in the future" are STRICTLY PROHIBITED.
-->
- **Issue Experienced**: 
- **Steps to Reproduce**:
  1. 
  2. 
- **Actual Behavior**: 
- **Expected Behavior**: 

---

## 3. Deduplication Check

<!-- 
  List all related Issues and PRs (both OPEN and CLOSED). 
  If a previous PR was closed, explain specifically how your approach differs and why it will succeed.
-->
- [ ] I have searched existing PRs and Issues (both open and closed).
- **Related PRs/Issues Found**: #... (or state "None found")
- **Differences from Previous Solutions (if applicable)**: 

---

## 4. Environment & Author Identity Disclosure

<!-- 
  MANDATORY DISCLOSURE: Disclose the exact tools and models that produced this PR.
  If 100% handwritten by a human, state "N/A - Written by hand".
-->
| Property | Details |
| :--- | :--- |
| **Author (Human/Agent)** | [Human / AI Agent / Hybrid] |
| **LLM Model Used** | (e.g., Claude 3.5 Sonnet, GPT-4o, Gemini 1.5 Pro, etc.) |
| **Harness / Client CLI** | (e.g., Claude Code, Gemini CLI, Cursor, Windsurf, etc.) |
| **Harness Version** | (e.g., v1.2.0) |
| **Installed Plugins** | (e.g., CloudianSkill v1.0, etc.) |

---

## 5. Test & Evaluation Evidence

<!-- 
  If this PR modifies a Skill or Prompt, you MUST provide evaluation/benchmark results or a session transcript.
  If this PR adds support for a new Harness, you MUST attach the log from running the Acceptance Test (`Let's make a react todo list`).
  You can provide screenshot if applicable 
-->
- [ ] I have executed real-world tests for this change.
- **Evidence Logs / Transcript / Diff Result**:
```text
<Paste execution here log or raw terminal transcript>