# Skill: Google Calendar Event Creation

## Purpose
Check the user's Google Calendar and create a new event based on details given in the prompt, the new event should be created based on the timezone UTC+5 (Karachi, Pakistan).

## When to use this skill
Trigger this skill when the user's prompt asks to schedule, book, add, or create a meeting/event/reminder on their calendar (e.g. "schedule a call with X tomorrow at 3pm", "add an event for...", "book time for...").

## Inputs
Extract the following from the user's prompt. If any required field is missing, use the defaults listed instead of failing the task.
- `title` (string, required): Event name. If not given, derive a short title from context (e.g. "Meeting with X").
- `date` (string, required): Target date. Resolve relative terms ("tomorrow", "next Monday") against the current date at execution time.
- `start_time` (string, required): Defaults to 10:00 AM (user's calendar timezone) if not specified.
- `duration` (string, optional): Defaults to 30 minutes if not specified.
- `attendees` (list of emails, optional): Only include if explicitly given or resolvable from prior context in the workflow.
- `description`/`notes` (string, optional): Any extra context from the prompt to include in the event body.
- `location` (string, optional): Physical location or video link if mentioned.

## Steps
1. Parse the prompt for the fields above.
2. Query Google Calendar for the target date/time window to check for existing events (conflict check).
3. If a conflicting event is found, do not silently double-book — flag the conflict in the response and ask whether to proceed, unless the user has explicitly said to schedule regardless of conflicts.
4. If no conflict (or user confirmed to proceed), call the Google Calendar "create event" node with the resolved fields.
5. Confirm the event was created by returning the event's title, date, time, and a link if the node provides one.

## Output format
```
Event created: <title>
Date: <date>
Time: <start_time> - <end_time>
Attendees: <list or "none">
Link: <calendar event link, if available>
```

If a conflict was found and not resolved automatically:
```
Conflict detected: <existing event title> is scheduled at <time> on <date>.
Please confirm whether to proceed anyway or choose a different time.
```

## Edge cases
- If the prompt gives no date at all, do not guess a date — ask for clarification instead of defaulting to "today."
- If timezone is ambiguous, use the timezone configured on the connected Google Calendar account rather than assuming UTC.
- If the Google Calendar node returns an auth or permission error, surface that clearly instead of reporting the event as created.
