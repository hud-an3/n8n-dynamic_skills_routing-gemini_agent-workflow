# Skill: Google Search

## Purpose
Search the web for a given topic using the attached SERP API tool and return a clear, structured summary of findings.

## When to use this skill
Trigger this skill standalone when the user's prompt asks the agent to search, look up, research, or find current information about a topic online (e.g. "search for...", "look up...", "what's the latest on...") **and gives no instruction to save the results/summary as a file or upload it to Google Drive.**

If the user's request also asks for the search summary to be turned into a file, saved, or uploaded (e.g. "search X and save the summary to drive", "look this up and generate a doc"), do **not** handle it here — hand off to the Generate and Save Summary Document skill, which calls this skill's steps internally as part of a larger flow.

## Inputs
- `query` (string, required): The topic or question to search for. Extract this directly from the user's prompt — do not rephrase it into something broader or narrower than what was asked.
- `num_results` (integer, optional): Number of results to pull. Default to 5 if not specified.

## Steps
1. Parse the user's prompt and extract the core search query. If the prompt is conversational ("can you check what's new with X"), reduce it to a clean search string ("X news").
2. Call the SERP API tool with the extracted query.
3. From the returned results, extract for each result:
   - Title
   - Source/domain
   - Snippet
   - URL
4. Discard irrelevant results (ads, unrelated domains, duplicate sources).
5. Synthesize the remaining results into a short summary (3–6 sentences) that directly answers the query, followed by a list of the top sources used.

## Output format
```
Summary: <2-4 sentence synthesis of what was found>

Sources:
1. <Title> — <URL>
2. <Title> — <URL>
3. <Title> — <URL>
```

## Edge cases
- If SERP API returns zero results, respond with: "No results found for '<query>'. Try rephrasing the search." Do not fabricate an answer.
- If results are stale or conflicting, note the discrepancy briefly instead of picking one arbitrarily.
- If the query is ambiguous (e.g. a single word with multiple meanings), ask a clarifying question before searching, or search the most common interpretation and state that assumption explicitly.
- If the prompt asks for a search AND wants the summary saved/uploaded/turned into a file, only the Generate and Save Summary Document skill should be triggered — it uses this skill's steps 1-5 internally, then hands the result to Create Document in Google Docs and Upload File to Google Drive.
