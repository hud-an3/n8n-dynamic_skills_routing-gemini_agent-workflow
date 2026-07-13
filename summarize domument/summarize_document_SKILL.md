# Skill: Summarize Document

## Purpose
Read the contents of an uploaded file (from the form/file trigger) or pasted text and produce a concise, accurate summary of its content.

## When to use this skill
Trigger this skill standalone when the user's prompt asks for a summary, overview, TL;DR, or key points of pasted text or an uploaded file, **and gives no instruction to save, upload, generate a file from, or otherwise store the result in Google Drive.** This is also the default action when a file is uploaded with no other instruction attached.

If the user's request also asks for the summary to be turned into a file, saved, or uploaded to Drive (e.g. "summarize this and upload it", "summarize this file and save it to drive"), do **not** handle it here — hand off to the Generate and Save Summary Document skill, which calls this skill's steps internally as part of a larger flow.

## Inputs
- `file_content` (binary/text, required): The extracted content of the uploaded file, or the pasted text from the prompt. If the file is a binary type (PDF, DOCX), it must first pass through a text-extraction node before reaching this skill — this skill does not parse binary itself.
- `file_name` (string, optional): Original filename, used for context and for any downstream folder-routing logic. Not present if the input was pasted text rather than a file.
- `summary_length` (string, optional): "short" (3-5 sentences), "medium" (1 paragraph), or "long" (structured with headers). Default to "medium" if not specified in the prompt.

## Steps
1. Confirm `file_content` is non-empty. If empty or extraction failed, do not attempt to summarize — flag the error instead (see Edge cases).
2. Identify the document type (report, article, contract, transcript, spreadsheet export, etc.) from structure/content — this affects tone of the summary.
3. Extract the main topic, key points, any decisions/action items, and notable figures or dates.
4. Write the summary in the requested length. Do not copy long verbatim passages — paraphrase.
5. If the document is very long (multi-section report), summarize per-section first, then produce one combined summary rather than skimming only the start.

## Output format
```
File: <file_name, or "pasted text" if no file>

Summary:
<summary text at requested length>

Key points:
- <point 1>
- <point 2>
- <point 3>
```

## Edge cases
- If `file_content` is empty or unreadable, output: "Could not read contents of <file_name>. Please check the file format or re-upload." Do not guess at content.
- If the file is not a document type this skill can meaningfully summarize (e.g. an image with no OCR text), say so explicitly rather than inventing a summary.
- If the prompt asks for a summary AND wants it saved/uploaded/turned into a file, only the Generate and Save Summary Document skill should be triggered — it uses this skill's steps 1-5 internally, then hands the result to Create Document in Google Docs and Upload File to Google Drive.
