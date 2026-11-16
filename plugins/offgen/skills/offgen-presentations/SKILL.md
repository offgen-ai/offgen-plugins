---
name: offgen-presentations
description: Generate branded, consultant-quality PowerPoint decks with offgen. Use when the user asks for a presentation, slide deck, pitch, proposal, report or any .pptx built from one of their organization's offgen workflows or from a brief in their company design system, or asks what offgen can generate for them.
license: Proprietary
compatibility: Requires the offgen MCP server (https://api.offgen.ai/mcp) to be connected.
metadata:
  author: offgen
  homepage: https://offgen.ai
---

# offgen presentations

offgen turns a short brief into a finished PowerPoint deck on the user's own company template. Each organization configures **workflows**: reusable, branded deck recipes such as a sales proposal, a monthly report or an information memorandum. You pick a workflow, fill its inputs and offgen builds the `.pptx`.

Use the `offgen` MCP tools. Never try to build the deck yourself from scratch when an offgen workflow fits.

## Flow

1. **List.** Call `list_workflows` with no arguments. Pick the workflow whose description best matches the request. Always address a workflow by its `id`, never by its display name. If several fit, or it is unclear whether one does, show the user the short list and ask. When none fits (or the user explicitly asks for a free-form deck in their design system), build a new deck with `create_deck` ([below](#a-new-deck-from-a-brief-no-workflow-fits)); it is never the first choice when a workflow fits.
2. **Describe.** Call `describe_workflow` with that `workflowId`. Read `inputs` and `inputSchema`: which fields are required, their types and the allowed values of choice fields.
3. **Gather inputs.** Fill every required input from the conversation. Ask the user only for required values you cannot infer. Dates are calendar days in `YYYY-MM-DD` format. Choice fields must use one of the listed values exactly.
4. **Generate.** Call `generate_presentation` with `workflowId` and `inputs`. It returns a `runId` straight away with status `queued`. If it returns an error, fix the inputs it names and try again.
5. **Poll.** Call `get_result` with the `runId`. Each call waits up to 55 seconds. While `pollAgain` is `true`, call `get_result` again with the same `runId`. Do not call other tools or read other skills between polls. `slidesEmitted` shows progress; a deck usually takes one to five minutes.
6. **Share.** When `status` is `completed`, give the user the download link from the result as a clickable Markdown link. The link is signed, works without signing in and expires, so share it right away. Do not try to download, open or summarize the file yourself. When `status` is `failed`, tell the user the `error` message and offer to retry with changed inputs.

## A new deck from a brief (no workflow fits)

When no workflow matches, offgen can build a new deck in the organization's design system:

1. Optional: `list_design_systems` when the user names a brand or design system; otherwise the organization's default is used.
2. `create_deck` with `brief` (topic, audience, storyline, must-have slides), optional `material` (the source text: notes, data, an article), `designSystemId`, `slideCount` and `language`. It returns a `runId` at once.
3. Call `get_result` with the `runId` while `pollAgain` is `true` (each call waits up to 55 seconds; a deck takes about two to five minutes; `progress` shows slides ready). Don't repeat `create_deck` to poll: the same inputs recover the same run, but after a failed one they start a new paid run (at most three).
4. When `completed`, look at the previews against the brief, then give the user the download link. `partial: true` or a slide with `draft: true` means some slides need a look: say so.

## Tips

- Tell the user which workflow you chose and why before generating, in one sentence.
- One run makes one deck. For several decks, start the runs and then poll each `runId`.
- `download_presentation` returns the same link as a completed `get_result`. Use it only to get a fresh link for an older run.
- If a tool says the API key or sign-in is invalid, ask the user to reconnect offgen (API keys live at https://app.offgen.ai/settings/api-keys).
