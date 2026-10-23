---
name: offgen-templates
description: Onboard, find and use offgen Library slide templates. Use when the user wants to turn a slide or a whole .pptx into reusable company templates, find a template ("do we have a timeline slide?"), or build slides or a deck on their own templates and brand.
license: Proprietary
compatibility: Requires the offgen MCP server (https://api.offgen.ai/mcp) to be connected. Works with tool calls only; a shell (curl) helps for large files.
metadata:
  author: offgen
  homepage: https://offgen.ai
---

# offgen templates

An offgen **template** is one slide in the organization's Library, kept on its original slide master. offgen analyzes each template: its name, category, a description, when to use it, and the role of every slot. Use that analysis to choose templates.

**You write all the content and you judge the result.** offgen's template tools never write or shorten text: `compose_deck` puts your text and pictures into the templates exactly as you give them.

## Build a deck

1. **Find templates.** `search_templates` with what each slide should show ("agenda", "consultant profile with photo", "roles list"), any language. Then `get_template` for the candidates (up to 4 at a time get preview images).
2. **Read the slots.** Each slot has a `shapeId`, `kind` (text, image, table), `role`, `guidance` and its text: `currentText`, or `paragraphs` (`n`, `text`, `bullet`) when it has several. `layout.placeholders` lists the placeholders of the template's layout, for slides of your own.
3. **Upload pictures.** `upload_presentation` with `base64` of a PNG or JPEG (under 4 MB), or `kind: "png"`/`"jpeg"` and the curl PUT. Use the returned `uploadId`.
4. **Compose.** `compose_deck` with all slides in order. Each slide is one of:
   - `{ templateId, text, remove, images }`, keyed by the template's slot shapeIds.
     - `text`: a string (one paragraph per line) or paragraphs `{ "text": "…", "like": n }`. `like: n` gives your paragraph the format of template paragraph `n` (heading, bullet, indent, font); without it a paragraph takes the format at its position. Use `like` whenever a slot mixes headings and items and you write a different number of items. `**Lead-in**: rest` writes a bold lead-in.
     - `remove`: shapeIds to delete: rows, cards or bars you don't fill, sample photos and icons, instruction markers such as "Logo einsetzen".
     - `images`: `{ shapeId: uploadId }`; the picture is cropped to the frame.
     - Table cells are keyed `"<tableShapeId>::r<row>c<col>"`.
   - `{ layoutOf: templateId, text }`: a NEW slide on that template's layout, keyed by `layout.placeholders` shapeIds; only the ones you fill appear.
5. **Check every preview** against the request: item counts, nothing left from the template (sample text, names, numbers, photos, icons), the right layout and formatting, and `validator.ok`. Fix by calling `compose_deck` again with the whole deck. Give the user the last download link.

Templates with different slide sizes (16:9 and 4:3) can't share a deck. A `locked` slot was frozen by an admin: use it as it is.

## Onboard a slide or a deck as templates

1. `upload_presentation`: a deck under 4 MB can go in `base64`; otherwise call it without and run the returned `curl` line.
2. `onboard_templates` with the `uploadId`. One template per slide; `slideNumbers` (1-based) keeps only some (at most 30 per call). The organization library is the default; if permissions refuse it, ask the user or use `space: "my"`. A retry with the same upload returns the same templates.
3. Analysis takes 1–3 minutes: `get_template` with `waitForAnalysis: true` until every `analysisStatus` is `READY`.

## Advanced: edit locally (coding agents)

For edits `compose_deck` can't express (moving shapes, new shapes, charts), download the composed deck with the returned `curl` line and edit it with python-pptx: change the text of existing runs so the formatting stays, and don't copy shapes between files. Then run the open-source OOXML validator (`npx @xarsh/ooxml-validator deck.pptx --officeVersion Office2019`) and fix every error it reports: each one makes PowerPoint offer "Repair". Don't onboard an edited deck just to see previews: that adds its slides to the Library as templates.

## Tips

- Links are signed and expire after 24 hours; files offgen makes for you are deleted after about a day.
- `not_found` for a template means it is gone, archived, or not visible to you. Search again.
- If a tool says the API key or sign-in is invalid, ask the user to reconnect offgen.
