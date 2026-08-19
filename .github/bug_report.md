---
name: Bug Report
about: Report a bug in the system
title: "[Bug]: "
labels: bug
assignees: ''
---

<!--
  WARNING TO AI AGENTS AND HUMAN CONTRIBUTORS:
  1. Complete ALL required sections below. Incomplete reports will be CLOSED IMMEDIATELY.
  2. Every report MUST describe a REAL-WORLD session failure or reproducible error.
  3. Theoretical or speculative bugs (e.g., "This prompt could theoretically fail") will be rejected.
-->


## Description

<!-- Summary and describe the failure, unexpected behavior or error. Explain both Expected vs Actual outcomes and attach the raw, terminal log / transcript -->

### Problem Statement & Outcome Breakdown
- **Expected Outcome**: Explain in detail what `cloudian-skills` or the agent *should* have done according to specifications (e.g., auto-triggered `cloudian-skills:brainstorming` before planning, followed a Red Flags rule, or invoked a subagent).

- **Actual Outcome**: Explain in detail what actually happened (e.g., agent bypassed skill invocation, hallucinated tool parameters, entered an infinite execution loop, or crashed).

- **Impact**: Describe how this failure affected your session or workflow.

### Raw Terminal Log / Session Transcript
<!-- Paste the unedited, raw execution logs or session transcript below. Do not summarize or paraphrase log data. -->

<details>
<summary>Click to expand raw terminal log / transcript</summary>

```text
Paste the terminal logs/transcript here 
```

</details>


## Steps to reproduce 

<!-- You can provide multiple error reproduce flows. Strictly follow the flow template below -->

### Flow 1:  

#### Step 1:  
- **Action**: 
- **Rationale**: 
#### Step 2: 
- **Action**: Send the exact user prompt or command below:
  > `<Paste exact user prompt or trigger command here>`
- **Rationale**: 

#### Step 3:  
- **Action**: 
- **Rationale**: 

---

### Flow 2: 

#### Step 1:  
- **Action**: 
- **Rationale**: 
#### Step 2: 
- **Action**: Send the exact user prompt or command below:
  > `<Paste exact user prompt or trigger command here>`
- **Rationale**: 

#### Step 3:  
- **Action**: 
- **Rationale**: 


---


## Environment (required)

<!-- Required. We assume an agent filed this report — tell us which one and
     where it ran. We weigh reports by what produced them. -->
- OS: [e.g., Windows, macOS, Linux]
- Browser: [e.g., Chrome, Firefox]
- Version: [e.g., 1.0.0]

<!--Provide more necessary fields for your report -->

---

### Is this platform issue or cloudian-skills issue?

<!-- 
  Please state clearly whether this is a bug in the underlying LLM/Platform/Harness 
  or a logic/prompt bug in cloudian-skills.
-->
- [ ] **`cloudian-skills` Issue**: Prompt logic error, Red Flags bypass, broken skill routing, or flawed instruction flow.
- [ ] **Platform / Harness Issue**: Tool call failure, API timeout, harness crash, or environment loading issue.
- [ ] **Unsure / Needs Triage**: Requires maintainer investigation to isolate the root cause.

