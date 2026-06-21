---
name: chatgpt
description: "Use when the user invokes @chatgpt, asks Codex to open ChatGPT, or asks Codex to consult, talk through, or validate something with ChatGPT based on the current conversation or repo context. This skill opens https://chatgpt.com/ through the Browser integration and supports supervised delegated prompting with rich bounded context, patient waits for long-running ChatGPT Pro responses, and final-answer extraction when the user clearly asks to involve ChatGPT."
---

# ChatGPT Browser Bridge

Use this skill as a bridge from Codex to the ChatGPT website through the Codex in-app Browser.

Two modes are supported:

- Open/view mode: put `https://chatgpt.com/` in front of the user.
- Delegated consultation mode: when the user clearly asks Codex to consult, talk with, validate with, or ask ChatGPT about the current work, Codex may write and submit the prompt to ChatGPT, wait for the full response to finish, then read the answer and bring the useful result back to the user.

## Behavior

1. Use the Browser / in-app browser capability to navigate to `https://chatgpt.com/`.
2. If the current in-app browser tab is already on `https://chatgpt.com/`, do not reload it unless the user explicitly asks.
3. Keep the browser visible when the user asks to open or view ChatGPT.
4. After navigation, verify that the tab is not blank. A useful check is the current URL plus a light page observation such as title, DOM snapshot, or screenshot.
5. If Browser reports `net::ERR_ABORTED`, the tab title remains `about:blank` / `New tab`, or the screenshot stays blank, report that ChatGPT did not load in the in-app Browser and stop. Offer Chrome or the official API as a later fallback only if helpful.
6. Do not use `web.run`, the OS default browser, Chrome, or a raw shell `start` command for this workflow.
7. Do not upload files, paste secrets, credentials, tokens, payment data, private family/student data, or large raw repo dumps into ChatGPT unless the user explicitly authorizes that exact data transfer.

## Delegated Consultation

Treat any of these as clear authorization to send a bounded prompt to ChatGPT for the current topic:

- The user invokes `@chatgpt` and asks a question that depends on ChatGPT's advice.
- The user says phrases like `talk it through with ChatGPT`, `consult ChatGPT`, `ask ChatGPT`, `according to ChatGPT`, `validate it with ChatGPT`, or `use ChatGPT for this`.
- The user asks Codex to have ChatGPT reason from the current conversation context.

### Context Sufficiency Gate

Before sending any delegated consultation to ChatGPT, Codex must decide whether the question depends on repo-specific, app-specific, user-history-specific, runtime-specific, or evidence-sensitive facts, and whether Codex already has enough reliable context from the current conversation or prior inspected evidence.

If Codex already has sufficient context from the conversation, prior tool results, or earlier repo inspection, do not perform unnecessary codebase searches just to satisfy the gate. Instead, think carefully about what ChatGPT needs to know, package the existing context clearly, include the known evidence and decisions, and label any remaining unknowns.

If the task depends on facts Codex does not yet know, Codex must gather enough local context first. This may include reading relevant files, routes, docs, tests, command output, browser observations, error logs, prior conversation decisions, deploy state, or other evidence available through Codex tools.

Do not ask ChatGPT to guess missing repo facts from a vague prompt. The useful value of this integration is that Codex can inspect the real workspace, package the truth clearly, and then ask a stronger external reasoning model for second-order analysis, risks, decision support, or alternatives.

Codex may consult ChatGPT only after one of these is true:

- Codex has inspected enough context to explain the real state of the code, app, bug, decision, or workflow.
- Codex already has enough reliable context from the conversation or prior evidence to explain the situation accurately.
- The user explicitly wants high-level brainstorming where repo or runtime facts are not required.
- The missing context cannot be obtained safely or efficiently, and Codex clearly labels what is unknown.

When context is insufficient, Codex investigates first, then consults ChatGPT with an evidence-backed context packet.

When using delegated consultation:

1. Open or reuse `https://chatgpt.com/`.
2. Build a rich but bounded context prompt for ChatGPT after passing the Context Sufficiency Gate. Do not under-contextualize when the user's request depends on prior discussion, repo state, product decisions, or files Codex has inspected. Include enough context for ChatGPT to be useful:
   - the user's current request,
   - relevant decisions already agreed in the conversation,
   - relevant repo facts, file paths, command results, docs, or small snippets Codex has inspected,
   - constraints, risks, and what Codex has already verified,
   - known unknowns and facts ChatGPT should not assume,
   - the exact question or decision ChatGPT should help with,
   - the requested answer style.
3. Tell ChatGPT to act as an outside advisor to Codex, not as the final authority. When the user is intentionally using a deeper or extended ChatGPT reasoning model, frame ChatGPT's role as a high-capability second-opinion model that should analyze the evidence, challenge assumptions, surface risks, and suggest practical next checks. Ask it to call out assumptions and to ask Codex to verify repo-specific facts instead of inventing them.
4. Submit the prompt without asking the user again when the authorization above is present.
5. Wait patiently for ChatGPT to finish. ChatGPT Pro answers can take minutes. Do not treat partial text, status text, or "thinking/finalizing" output as the final answer.
6. Read ChatGPT's final answer only after completion signals are clear, then synthesize the useful points for the user and verify any repo-specific or high-impact claim before implementing code.
7. If ChatGPT asks for more context, decide whether Codex can safely provide a useful summary or additional inspected facts. Ask the user before sending sensitive data or broad private content.

## Context Relay

Prefer a "context packet" over a tiny one-line question when the task is non-trivial. A good packet usually has:

- Role: ChatGPT is an external reasoning partner; Codex owns tools, repo inspection, implementation, and final verification.
- User goal: what the user is trying to decide or change.
- Current state: what Codex knows from the conversation, repo, docs, browser, or commands.
- Evidence: concise file paths, key snippets, command outputs, URLs, or observed behavior.
- Constraints: privacy limits, business rules, deadlines, product direction, validation gates.
- Unknowns: what Codex has not verified and what ChatGPT should not assume.
- Ask: the exact advice wanted, with priorities and output format.

Do not send secrets, credentials, tokens, private family/student data, payment data, raw logs containing sensitive values, or broad repo dumps. Summarize instead. If exact code is needed, send only the smallest relevant snippet.

## Completion Discipline

When reading a ChatGPT response:

1. Poll the visible conversation until generation is complete.
2. Treat these as incomplete signals: `Thinking`, `Generating`, `Finalizing response`, `Stop`, localized equivalents of those labels, no visible composer textbox, no available send button, or answer text still changing.
3. Prefer waiting until the composer textbox is visible again and the main answer text has stayed stable across at least two checks separated by a short delay.
4. For Pro/deep-reasoning responses, wait up to 10 minutes by default unless the user asked for a faster attempt or interrupts.
5. If the response is still running after the wait budget, report that it has not finished yet and ask whether to keep waiting or proceed from available evidence. Do not summarize a partial response as final.
6. If ChatGPT produces a brief status update such as "I will compare..." but no actual answer, keep waiting.

While waiting, send short progress updates to the user occasionally. Do not abandon the browser session silently.

Use this default framing unless the user gives a better one:

```text
You are going to talk with Codex about an app or task in progress. Codex will give you enough context from the user, the conversation, the repo, and the tools it has used, without including secrets or unnecessary private data.

Your job is to respond to Codex with deep analysis, questions, risks, and practical recommendations. Take time to reason. Do not assume information Codex has not provided. If context is missing, ask for only the minimum needed.

Do not ask for credentials, private data, API keys, payment information, family/student data, or secrets. If something requires verification in the repo, app, internet, or command output, ask Codex to check it before concluding.

Respond clearly, completely, and actionably so Codex can bring the answer back to the user, verify it with tools, or implement changes.
```

## Useful Phrases

- `@chatgpt open ChatGPT`
- `@chatgpt open chatgpt.com`
- `@chatgpt I want to talk with ChatGPT`
- `@chatgpt talk it through with ChatGPT using this context`
- `@chatgpt consult ChatGPT and tell me what it recommends`
- `@chatgpt ask ChatGPT what risks it sees`
- `@chatgpt send the full context and wait until it finishes`
- `@chatgpt consult it in Pro mode and do not stop until the full answer is ready`

## Follow-Up Scope

This plugin is intentionally browser-based. It is not an API bridge and does not bypass login. It may send prompts and read ChatGPT responses only when the user has clearly asked to involve ChatGPT for the current topic and the page state allows browser interaction. It should preserve the division of labor: ChatGPT can do slow external reasoning, while Codex uses tools, checks real sources, edits files, and verifies. Some ChatGPT browser sessions may remain blank or get stuck in the in-app Browser; report that plainly when it happens.
