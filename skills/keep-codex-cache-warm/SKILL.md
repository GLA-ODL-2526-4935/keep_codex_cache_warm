---
name: keep-codex-cache-warm
description: Manage an experimental 29-minute no-op prompt loop in the current Codex chat when explicitly requested. Supports start, stop, and status, with best-effort resets after user activity. Cache reuse is unverified.
---

# keep_codex_cache_warm

An experimental instruction-only skill made with help from OpenAI Codex. Its purpose is to request periodic minimal turns that may keep a reusable prompt cache warm. It does not guarantee cache hits, reduced usage, lower latency, invisible operation, or exact idle timing.

## Activation and capabilities

Enable only when the user explicitly asks to start/use this skill. Installing, reading, or editing its files does not enable a task. Do not enable it for general questions about caching.

Use the built-in chat scheduler (`automation_update` when available), kind `heartbeat`, with `destination: thread` for the calling chat. Do not substitute a standalone cron job, another chat, daemon, proxy, hooks, or backend requests. If scheduling, discovery, updates, or pausing are unavailable, report the missing capability instead of claiming setup succeeded.

Keep current model and reasoning settings. At activation, explain once that the task will request a period (`.`) every 29 minutes, add conversation turns, and consume usage. Cache reuse and user-activity resets are best effort.

## Owned task and exact prompt

Use the exact scheduler name `keep_codex_cache_warm`. Resolve the target from trusted calling-chat context or `destination: thread`; never guess from local paths or unrelated chats.

Find existing tasks through scheduler discovery or read-only automation metadata. Require the exact name, exact target chat, and exact prompt below to identify this skill's task. Retain its returned ID in conversation context. If discovery or ownership is uncertain, or multiple tasks match, report the ambiguity rather than create a duplicate or change unrelated tasks.

Create once and update that ID thereafter. Set the entire saved prompt to exactly this single line, without metadata, a skill invocation, or a file reference:

```text
Ignore this scheduled maintenance input. Do nothing, call no tools, and reply only with "."
```

Use a 29-minute recurrence, with its first run a fresh interval from activation. The scheduler may accept `FREQ=MINUTELY;INTERVAL=29`. Do not display raw scheduling directives to the user. Preserve unrelated task fields. Verify target, cadence, actual status and next-run anchor wherever exposed; do not invent missing guarantees. Never edit scheduler storage directly.

All scheduled runs use the same prompt, including the first. They call no tools, emit no progress commentary, change no files, and reply only `.`. They do not rearm or create tasks; repetition comes from the scheduler. Do not resume prior work on a scheduled maintenance input.

Do not put personal names, email addresses, installation paths, or project paths into prompts or distributed files. Scheduler identity and cadence belong in scheduler fields, not the scheduled message.

## Start, user activity, stop, and status

- **Start:** Activate the owned task with the exact prompt and a fresh 29-minute delay. Repeated start resets the same task. Installation alone schedules nothing.
- **User activity while active:** Pause the task as early as practical when handling a genuine user message; remember that this pause was temporary. Perform the user's work, then reactivate the same task with the exact prompt and a fresh interval as the final tool action before responding.
- **Stop:** Pause the owned task and clear the retained rearm intent. Do not reactivate it at the end of the response. If absent or already paused, report that without creating anything. Only an explicit start enables a stopped task.
- **Status:** Read and report actual status, target, interval and next run where exposed. Never start a paused or absent task merely to report status. If it was active when a genuine status request arrived, apply the usual user-activity reset and describe the resulting state.

Scheduled maintenance and scheduled diagnostic inputs are not genuine user activity. Do not apply cancellation/reset instructions to them.

These are retained conversation instructions, not lifecycle listeners. A skill cannot reliably intercept submission or execute after the final response. Resets happen when the model handles input and just before its response. Instructions can be forgotten. Interruption or a crash can leave a temporary pause behind. Treat a paused task as disabled unless this handling turn established its temporary rearm intent or the user explicitly starts it; never infer that it should resume from old context alone.

## Tests and failures

For an explicitly requested disposable-chat test, allow a supported temporary interval and retain it through resets. Identify test mode, do not silently shorten the normal interval, and stop/remove the test task when requested.

On ambiguous create/update results, reconcile the existing task before retrying. Do not create replacement tasks or repeatedly retry as a shortcut. Report confirmed outcomes and uncertainty separately. A failed update is not successful activation or reset. Static validation does not prove live scheduling or cache reuse. The one-character reply is a visible transcript entry and does not guarantee minimal total token use.
