# Case Study 01: Streaming Markdown Parser Bug in a Large Language Model Interface

**Author:** Julian Gonzalo Cutini  
**Date:** January 20, 2026  
**Platform Tested:** Z.ai — GLM-4.6  
**Environment:** Chrome Browser / Windows 10  
**Session:** [Shared Session](https://chat.z.ai/c/7ee87ac7-92f2-45fb-6d5d-9137d1de9c99)

---

## Executive Summary

During a live interaction session with the GLM-4.6 model on the Z.ai platform, a reproducible bug was identified, isolated, and documented in real time. The bug caused long AI responses to render with corrupted markdown and truncated text during streaming. The root cause was determined to be a **synchronization failure between the token stream and the markdown renderer** in the frontend — not a backend generation error. The bug was reproduced multiple times, documented with screenshots and video, and a diagnostic methodology was applied without prior formal QA training.

---

## Problem

Long or structurally complex responses from the GLM-4.6 model appeared broken when first rendered on screen. Symptoms included:

- Text truncated mid-sentence
- Markdown tokens (`**`, `#`) appearing unprocessed inline with plain text
- Words rendered as isolated fragments disconnected from their context
- Headings broken across lines (e.g., `H**prueba de reproducibilidad**`)
- List items cut off mid-word (e.g., `"Hipótesis: El e"`, `"Experimento: Pedi"`)

**Screenshot showing the bug active:**

![Bug Screenshot](Z_IA2.png)

**Screenshot showing normal rendering (baseline):**

![Normal Screenshot](Z_IA1.png)

---

## Hypothesis

The error is not in the backend (the model generates correct text). It is a **frontend rendering failure** that occurs specifically during the real-time streaming of long responses. When the page is refreshed after the response has fully loaded, the content renders perfectly — confirming the HTML is correct, but the rendering pipeline breaks under streaming conditions.

---

## Methodology

The bug was identified through a structured, self-designed experimental process:

### Step 1 — Initial Observation
The first occurrence of truncated text was noticed during a regular interaction. Rather than dismissing it as a one-time glitch, the user paused to investigate.

### Step 2 — Differential Diagnosis
A page refresh (F5) was applied. The content appeared complete and correctly formatted. This ruled out a backend generation failure and pointed to a frontend rendering issue.

### Step 3 — Hypothesis Formation
Working hypothesis: *"The bug occurs when a long, structurally complex response is generated via streaming, and the markdown parser fails to process tokens that arrive fragmented."*

### Step 4 — Controlled Reproduction
A deliberate prompt was designed to elicit a long, heavily formatted response (numbered lists, bold text, multiple sections). The prompt was submitted specifically to trigger the bug conditions.

### Step 5 — Documentation
Before sending the prompt, a screenshot of the input was captured ("the catalyst"). After receiving the broken response, a second screenshot was taken ("the evidence"). A screen recording was made to capture the real-time occurrence.

### Step 6 — Reproduction Confirmation
The experiment was repeated multiple times. The bug reproduced consistently, confirming it is **not intermittent** but a **systematic, deterministic fault**.

---

## Evidence

| # | File | Description |
|---|---|---|
| 1 | `Z_IA1.png` | Baseline — normal rendering state |
| 2 | `Z_IA2.png` | Bug active — markdown corruption visible during streaming |
| 3 | `https://youtu.be/3P8fbo6QzS4` | Screen recording — bug occurring in real time |
| 4 | Full chat transcript | Complete interaction log with methodology documented inline |

---

## Technical Diagnosis

The visual pattern of the bug — specifically the presence of raw unprocessed markdown tokens (`**`) mixed with partially rendered text — points to a **race condition** between two frontend processes:

1. **The stream token receiver** — receives text fragments from the server in real time
2. **The markdown renderer** — attempts to parse and display those fragments as formatted HTML

When a markdown token (e.g., `**bold**`) arrives split across multiple stream chunks (e.g., `**bo` | `ld**`), the parser processes the incomplete token prematurely, leaving orphaned syntax characters visible and breaking the layout. On page refresh, the complete HTML string is parsed in a single pass, which is why the content appears correctly.

**Root cause classification:** Streaming markdown render synchronization failure (frontend)  
**Severity:** Medium-High — affects readability and user trust significantly on complex responses  
**Browser/OS specificity:** Confirmed in Chrome / Windows 10

---

## Proposed Solution

The engineering team should investigate the following approaches:

1. **Buffer the stream output** before rendering, until a safe markdown boundary is reached (end of a block element)
2. **Implement incremental markdown parsing** with awareness of incomplete tokens — defer rendering of open tags until their closing counterpart arrives
3. **Add a post-stream render pass** that re-processes the complete response once streaming is confirmed finished

---

## Potential Impact

This class of bug directly affects user experience in high-value use cases:
- Long document generation
- Code generation with syntax highlighting
- Structured reports and step-by-step guides

Users encountering this bug may lose trust in the model's capabilities, attributing the rendering failure to the model's intelligence rather than a UI defect. **This is a perception risk as much as a technical one.**

---

## Key Skills Demonstrated

- Independent bug identification without prior QA training
- Differential diagnosis (backend vs. frontend isolation)
- Experimental design for bug reproduction
- Real-time documentation with structured evidence collection
- Technical root cause analysis using visual pattern recognition
- Lateral thinking applied to software quality problems

---

*This case study is part of the AI Testing Portfolio of Julian Gonzalo Cutini.*  
*All evidence collected on January 20, 2026.*
