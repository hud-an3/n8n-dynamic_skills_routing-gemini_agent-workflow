# Skill: Generate and Save Summary Document

## Purpose
Handle multi-step requests where the user wants a summary (of entered text, an uploaded file, or a web search) turned into a new Google Doc and saved into the `summary_folder` in Google Drive — all from a single prompt. This skill orchestrates three other skills in sequence: (1) content generation via Document Summarization or Google Search, (2) Create Document in Google Docs, (3) Upload File to Google Drive.

## When to use this skill
Trigger this skill instead of the standalone Document Summarization or Google Search skills whenever the user's prompt combines a summary/search request with any of these signals:
- "...and upload it to drive" / "...and save it to drive"
- "...and create a file" / "...and generate a file/doc"
- "save the summary" / "file this away"

Do **not** trigger this skill if the user only asks for a summary with no mention of saving, generating a file, or uploading — that is a plain summarization request, handled entirely by the Document Summarization skill on its own (see its "When to use" section).

Two entry points feed into this skill:

**Case 1 — Text or uploaded file → summary → saved doc**
Trigger when the user provides pasted text or an uploaded file and asks for a summary to be saved/uploaded/turned into a file.

**Case 2 — Web search → summary → saved doc**
Trigger when the user asks to search a topic online and then wants a summary of the results saved/uploaded/turned into a file.

## Inputs
- `source_type` (string, required): `"text_or_file"` or `"web_search"` — determined from the user's prompt.
- `raw_input` (string/binary, required): The pasted text, uploaded file content, or search query/topic, depending on `source_type`.
- `file_name` (string, optional): Present only when `source_type` is `"text_or_file"` and a file was uploaded.

## Steps
1. **Determine source_type** from the prompt. If a file is attached or text is pasted directly, use `"text_or_file"`. If the user is asking to search the web for a topic, use `"web_search"`.
2. **Generate the summary content:**
   - If `source_type` is `"text_or_file"`: follow the Document Summarization skill's steps exactly (confirm content is non-empty, identify document type, extract key points, write summary — default to "medium" length unless the user specified otherwise) to produce `summary_text`.
   - If `source_type` is `"web_search"`: follow the Google Search skill's steps exactly (call SERP API, extract and filter results, synthesize) to produce `summary_text` as the search skill's "Summary" output (sources list can be appended into the document content as well, so the saved file is self-contained).
3. **Generate a title** for the new document based on the subject matter (e.g. `"<Topic> Summary"` or `"<File name> Summary"`).
4. **Create the document:** call the Create Document in Google Docs skill with `content = summary_text` and the generated `title`. Capture the resulting document ID/file name.
5. **Resolve the destination folder:** using the Search files and folders in Google Drive tool, look up a folder named `summary_folder`.
   - If found, use its ID as the destination.
   - If not found, create a new folder named exactly `summary_folder` and use its ID. (Unlike a user-specified custom folder name, `summary_folder` is a fixed, known destination for all generated summaries — always resolve/create it, never substitute a different folder.)
6. **Upload the document:** call the Upload File to Google Drive skill with the created document as `file` and `folder_name = "summary_folder"` (this reuses that skill's existing "folder name specified" path).
7. **Confirm success** and report back to the user with the summary, the document title, and where it ended up.

## Output format
```
Summary:
<summary text>

Document created: <title>
Uploaded to: summary_folder
Link: <Drive file link, if available>
```

## Edge cases
- If summarization or search produces no usable content, stop before creating a document — report the failure the same way the underlying skill (Document Summarization or Google Search) would, and do not create an empty file.
- If the `summary_folder` lookup finds multiple matches, use the first exact-name match and note the ambiguity in the output, per the Upload File skill's existing rule.
- If document creation succeeds but the upload step fails, report the partial success clearly (document exists but was not moved to `summary_folder`) rather than claiming full success.
- If the user's prompt asks for a search + summary but does NOT mention saving/uploading/generating a file, do not use this skill — hand off to the Google Search skill alone, which already produces a chat-visible summary and sources list.
