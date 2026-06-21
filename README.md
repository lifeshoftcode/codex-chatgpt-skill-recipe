# Codex ChatGPT Skill Recipe

This repository contains a small `@chatgpt` skill and a recipe for turning it into a local Codex plugin.

It is intentionally simple. The useful idea is the workflow: Codex gathers real context, passes a bounded prompt to ChatGPT when the user explicitly asks, waits for the final answer, and then Codex verifies or implements with tools.

This is not an official OpenAI integration and it does not use the OpenAI API.

## What Is Included

```text
skills/chatgpt/SKILL.md
templates/plugin.json
```

- `skills/chatgpt/SKILL.md` is the actual skill behavior.
- `templates/plugin.json` is suggested plugin metadata for Codex.

## Ask Codex To Create The Plugin

Clone or download this repo, open it in Codex, and paste this prompt:

```text
Create a local Codex plugin named `chatgpt` from this repository.

Use the plugin-creator skill if it is available.

Use:
- `skills/chatgpt/SKILL.md` as the skill source
- `templates/plugin.json` as the plugin manifest template

Create a valid Codex plugin with this shape:

chatgpt/
  .codex-plugin/plugin.json
  skills/chatgpt/SKILL.md

Keep the behavior simple and browser-based:
- open https://chatgpt.com/ in the Codex in-app browser
- only send prompts to ChatGPT when the user explicitly invokes @chatgpt or asks Codex to consult ChatGPT
- gather enough bounded context before consulting ChatGPT
- do not send secrets, credentials, payment data, private family/student data, or broad raw repo dumps
- wait for long-running ChatGPT responses to finish before summarizing

After creating the plugin, validate it if the local Codex plugin validation script is available.
```

## Manual Plugin Shape

If you are building it yourself, the plugin folder should look like this:

```text
chatgpt/
  .codex-plugin/
    plugin.json
  skills/
    chatgpt/
      SKILL.md
```

Copy `templates/plugin.json` to `.codex-plugin/plugin.json`, and copy `skills/chatgpt/SKILL.md` to `skills/chatgpt/SKILL.md`.

## Example Prompts

```text
@chatgpt open ChatGPT
@chatgpt consult ChatGPT and tell me what it recommends
@chatgpt send the full context and wait until it finishes
```

## Why This Exists

The interesting part is not the amount of code. It is the division of labor:

- Codex owns local context, repo inspection, tools, edits, and verification.
- ChatGPT acts as an outside reasoning partner when explicitly invited.
- The skill tries to keep context useful, bounded, and privacy-aware.

## Status

Experimental, local-first, and intentionally small.

Before publishing broadly, choose a license for the repository. MIT is a common option if you want others to reuse or adapt it.
