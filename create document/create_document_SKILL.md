# Skill: Create Document in Google Docs

## Purpose
Create a new Google Doc containing a given block of text content, placed directly into a named destination folder, with an appropriate generated title. This skill fully owns getting the content into a real, non-empty document in the right place — it does NOT use the Upload File to Google Drive skill. A Google Doc created via this skill is already a file in Drive the moment it's created; there is nothing left to "upload."

## When to use this skill
Trigger this skill only as a **sub-step** of another skill/flow that has already produced text content (e.g. a summary or search-result synthesis) and needs that content saved as a standalone document. Do not trigger this skill directly from a bare user request — it is always called after content has been generated.

This skill is used by:
- Generate and Save Summary Document skill (Case 1 and Case 2 flows)
- Any other flow that explicitly asks to "create a file" or "generate a doc" from existing text

## Inputs
- `content` (string, required): The final text to place inside the document body. This should be the already-finished text (e.g. a summary) — do not truncate or re-summarize it here.
- `title` (string, required): A short, descriptive title for the document, generated from the content (e.g. topic + "Summary", or the search query + "Search Summary"). If the calling flow already produced a title, reuse it — do not invent a second one.
- `folder_name` (string, required): The Drive folder the document should end up in. For Case 1 / Case 2 flows this is always `"summary_folder"`.

## Steps
1. Confirm `content` is non-empty. If empty, do not create a document — flag the error instead (see Edge cases).
2. If no `title` was supplied by the calling flow, generate a concise title (under ~8 words) that reflects the subject of the content.
3. **Resolve the destination folder ID:** use the Search files and folders in Google Drive tool to look up a folder named `folder_name`.
   - If found, use its ID.
   - If not found, create a new folder with that exact name (Create folder in Google Drive tool) and use the newly created folder's ID.
4. **Create the document:** call the "Create a document in Google Docs" tool, passing `title` and the resolved folder ID as `Folder_ID` so the doc is created directly inside the destination folder. Capture the returned document ID.
5. **Insert the content:** call the "Insert text into Google Doc" tool, passing the document ID from step 4 as `Document_ID` and `content` as `Content`. This step is mandatory — a document created in step 4 alone is blank. Never skip it and never call the "Create a document in Google Docs" tool a second time to try to add content; it only supports setting a title on creation.
6. Capture the final document link/ID — the calling flow reports this to the user directly. Do not call the Upload File to Google Drive skill for this document; it was already created inside the correct folder in step 4.

## Output format
```
Document created: <title>
Document ID: <id>
Folder: <folder_name>
Link: <doc link, if available>
```

## Edge cases
- If `content` is empty or missing, output: "No content available to create a document from." Do not create an empty or placeholder document.
- If document creation succeeds (step 4) but the text-insert call (step 5) fails, report this as a partial failure — "Document created but content could not be inserted" — rather than reporting full success. Do not leave the user believing a blank document is the finished summary.
- If the folder lookup in step 3 finds multiple matches, use the first exact-name match and note the ambiguity in the output.
- If folder creation fails (permissions, quota, etc.), report the failure clearly rather than silently placing the doc in the wrong folder.