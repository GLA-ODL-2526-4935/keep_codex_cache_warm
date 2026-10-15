# keep_codex_cache_warm

An experimental Codex skill that requests a minimal no-op response every 29 minutes in the current chat, with the aim of keeping reusable prompt caches warm.

**Made with help from OpenAI Codex. This skill is an experimental subject, not a proven cache optimization.** It is a community experiment and is not an official OpenAI product or endorsement.

## What it does

- Starts only when explicitly requested.
- Maintains one scheduled task per chat, using the same minimal prompt on every run.
- Asks the model to pause the task when handling a real user message and restart the countdown just before responding. This reset is best effort.
- Supports start, stop, and status. It keeps existing model settings.

The entire scheduled prompt, including the first run, is exactly:

```text
Ignore this scheduled maintenance input. Do nothing, call no tools, and reply only with "."
```

The expected response is a single period (`.`). Cadence and task identity live in scheduler fields rather than in the scheduled message.

## Requirements and limitations

Use a Codex environment that discovers personal skills and exposes built-in scheduling for the current chat, including discovery, updates, and pausing. CLI access alone does not establish that chat scheduling is available. The skill reports missing capabilities instead of installing another scheduler. Follow the host's availability requirements; local scheduled work may require the app and machine to remain running.

This is an instruction-only skill: no Python daemon, hooks, proxy, or direct API client. It requests periodic model turns; it does not control server cache lifetime, cache routing, or cache settings.

**Cache reuse has not been verified.** Scheduled turns consume usage and add conversation content. A short visible response does not guarantee minimal reasoning or input-token use, lower latency, or a net usage saving. The 29-minute cadence is an experimental choice; scheduler delays can exceed it. Model/provider cache behavior varies, and API documentation does not establish the behavior of these scheduled Codex runs.

A skill cannot reliably intercept every submitted message or act after a response finishes. Pausing happens when the model handles your input, and rearming happens before the response ends. Forgotten instructions or interrupted turns can leave the task paused or unreset. Ask for status and explicitly start again if needed.

## Install

From the root of your repository checkout, run in PowerShell:

```powershell
$repoPath = (Get-Location).Path
$skillsPath = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $env:USERPROFILE '.codex\skills'
}
$skillName = 'keep-codex-cache-warm'
$destination = Join-Path $skillsPath $skillName
if (Test-Path -LiteralPath $destination) {
    throw "Skill already exists at $destination. Review it before replacing it."
}
New-Item -ItemType Directory -Path $skillsPath -Force | Out-Null
Copy-Item -LiteralPath (Join-Path $repoPath "skills\$skillName") -Destination $destination -Recurse
```

Reopen Codex or start a new chat if the skill is not discovered immediately. The identifier uses hyphens to meet skill naming rules; the display name retains underscores. Installation creates no scheduled task.

## Use

Start in the chat you want to maintain:

```text
Use $keep-codex-cache-warm to start the loop in this chat.
```

Check actual state:

```text
Use $keep-codex-cache-warm to show status in this chat.
```

Stop:

```text
Use $keep-codex-cache-warm to stop the loop in this chat. Do not rearm it.
```

If stopping cannot be confirmed, use the app's scheduled-task controls. Removing the skill folder does not cancel an existing task; stop it first. An explicit start enables a stopped loop. Other chats' tasks are left alone.

## Validation and live smoke test

Run the skill creator's `scripts/quick_validate.py` against `skills/keep-codex-cache-warm` with Python and PyYAML available. This checks skill metadata, not scheduler execution or cache reuse.

For a live test, explicitly request a temporary one-minute interval in a disposable chat. The normal default remains 29 minutes.

1. Start the skill and inspect that exactly one active task targets the test chat and has the exact one-line prompt above.
2. Observe at least two scheduled runs. Both should reply only `.` and call no tools.
3. Send a real user message and check that the task pauses during handling and receives a fresh countdown before the response ends.
4. Start again and verify no duplicate task appears. Ask for status and compare it with the scheduler.
5. Stop and confirm it stays paused beyond an interval. A status request must not restart it.
6. Remove the disposable test task through the app's scheduler controls.

Record unsupported intervals, missing history, interrupted turns, or scheduling failures as limitations. A saved task configuration alone does not prove it ran. Cache reuse requires separate cached-token telemetry; these smoke tests cannot establish it.

## Privacy

Distributed files use generic paths and project contributor attribution. No personal usernames, email addresses, chat IDs, or machine-specific paths are needed. Git author metadata and earlier committed contents are separate from the working files; review repository history before publication if anonymity is required.

## References

- [Scheduled tasks](https://learn.chatgpt.com/docs/automations?surface=app): chat scheduling and minute-based intervals.
- [Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching): prefix reuse, retention, and model-dependent behavior. This does not prove scheduled Codex cache hits.
- [Wirescope](https://github.com/avirtual/wirescope): a related proxy approach to cache observation and warming. No third-party code is incorporated here.

## License

MIT. See [LICENSE](LICENSE). The repository retains its Python `.gitignore`.
