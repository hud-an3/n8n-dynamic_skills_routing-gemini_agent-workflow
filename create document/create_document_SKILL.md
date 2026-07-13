# Skill: Create Document in Google Docs

## Purpose
Create a new Google Doc containing a given block of text content, with an appropriate generated title. This skill only creates the document in Google Docs — it does NOT upload/move the file anywhere. Uploading is handled separately by the Upload File to Google Drive skill.

## When to use this skill
Trigger this skill only as a **sub-step** of another skill/flow that has already produced text content (e.g. a summary or search-result synthesis) and needs that content saved as a standalone document. Do not trigger this skill directly from a bare user request — it is always called after content has been generated.

This skill is used by:
- Generate and Save Summary Document skill (Case 1 and Case 2 flows)
- Any other flow that explicitly asks to "create a file" or "generate a doc" from existing text

## Inputs
- `content` (string, required): The final text to place inside the document body. This should be the already-finished text (e.g. a summary) — do not truncate or re-summarize it here.
- `title` (string, required): A short, descriptive title for the document, generated from the content (e.g. topic + "Summary", or the search query + "Search Summary"). If the calling flow already produced a title, reuse it — do not invent a second one.

## Steps
1. Confirm `content` is non-empty. If empty, do not create a document — flag the error instead (see Edge cases).
2. If no `title` was supplied by the calling flow, generate a concise title (under ~8 words) that reflects the subject of the content.
3. Call the "Create a document in Google Docs" tool with `title` and `content`.
4. Capture the returned document ID / link and file name — the calling flow will need these to hand off to the Upload File skill.

## Output format
```
Document created: <title>
Document ID: <id>
Link: <doc link, if available>
```

## Edge cases
- If `content` is empty or missing, output: "No content available to create a document from." Do not create an empty or placeholder document.
- If document creation fails (permissions, API error), report the failure clearly rather than silently retrying or fabricating a success message.
- This skill never decides where the resulting file is stored — that is always the responsibility of the Upload File to Google Drive skill, called next in the flow.
