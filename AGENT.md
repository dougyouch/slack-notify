# AGENT.md

Guidance for AI agents working in this repository.

## What this is

`slack-notify` is a small set of standalone Bash scripts for posting Slack messages from the command line, mainly to get notified when long-running jobs (e.g. in `screen`) finish. There is no build system, package manager, or test suite. Each script in `bin/` is meant to be installed on its own by copying it to `/usr/local/bin` (see README).

## Layout

```
bin/
  slack-notify               # post a message via chat.postMessage; prints the message ts
  slack-auth-test            # check SLACK_TOKEN via auth.test
  slack-message-formatter    # rewrite @username -> <@SLACKID> using a mapping file
README.md                    # user-facing docs
LICENSE                      # MIT
```

## Scripts

### `bin/slack-notify`
- Needs `curl` and `SLACK_TOKEN`.
- If the first argument starts with `@`, the rest of it is the channel (`@#general`); otherwise uses `SLACK_CHANNEL`.
- The remaining arguments are joined into the message text. `SLACK_MESSAGE_PREFIX` is prepended with a single space between them.
- Optional env: `SLACK_THREAD_TS` (reply in a thread), `SLACK_USERNAME`, `SLACK_ICON_EMOJI`.
- On success it prints only the message `ts` to stdout, so callers can capture it and reuse it as `SLACK_THREAD_TS` to thread later messages. **Keep stdout clean on success**, since this is the script's output contract.
- On failure it prints `failed to send message` plus the raw API response to stderr and exits 1.
- JSON is parsed with Bash string matching (`*"\"ok\":true"*` and prefix/suffix stripping), not `jq`. This is on purpose so there are no dependencies.

### `bin/slack-auth-test`
- Calls `https://slack.com/api/auth.test` and prints `valid SLACK_TOKEN` or `invalid SLACK_TOKEN` plus the response (exit 1).

### `bin/slack-message-formatter`
- Swaps each `@name` (`[A-Za-z0-9_]+`) for `<@PROFILE_ID>` using lines of the form `name:PROFILE_ID`.
- Mapping file: `SLACK_USER_CONF`, default `/etc/slack-user.conf`. If the file is missing, the message is echoed back unchanged.
- It is standalone. `slack-notify` does **not** call it; users chain it themselves, e.g. `slack-notify "$(slack-message-formatter hi @bob)"`.

## Conventions

- Plain `#!/bin/bash` with no external dependencies other than `curl` (plus `grep`/`cut` in the formatter). Don't add `jq`, Python, etc.
- Configuration comes only from environment variables named `SLACK_*`.
- Check prerequisites first (`which curl`, empty `SLACK_TOKEN`), print a short message to stderr, and `exit 1`.
- 4-space indentation (the formatter mixes in tabs). Use `[ ... ]` for simple tests and `[[ ... ]]` for pattern/regex matching, as the existing code does.
- Each script must keep working when installed alone with `curl -o`, so don't add shared sourced libraries.
- Update `README.md` whenever user-facing behavior or env vars change.
- Commit style is short, lowercase, sometimes conventional-commit (`feat: ...`, `fix typo`).

## Testing

There are no automated tests. To check a change by hand:

```bash
bash -n bin/*                       # syntax check
shellcheck bin/*                    # if available
SLACK_TOKEN=... bin/slack-auth-test
SLACK_TOKEN=... bin/slack-notify @#some-test-channel hello
printf 'bob:U123\n' > /tmp/users.conf
SLACK_USER_CONF=/tmp/users.conf bin/slack-message-formatter hi @bob   # -> hi <@U123>
```

Sending real messages needs a valid token and posts to a real workspace, so ask the user before doing it.

## Gotchas

- Send all `chat.postMessage` fields with `--data-urlencode`. Plain `-d` mangles messages that contain `&`, `+`, `%` or `=`.
- `slack-notify` writes errors to stderr so that `TS=$(slack-notify ...)` only ever captures a timestamp.
- One trailing space is trimmed from `SLACK_MESSAGE_PREFIX` before the separator space is added, so `"prod: "` and `"prod:"` give the same result.
- The formatter walks the message left to right and builds a new string. Don't go back to `${MESSAGE/...}` replacement: it replaces the first textual match, so `@bob` could hit the start of an earlier `@bobby`.
- The formatter's default mapping file is `/etc/slack-user.conf` (singular), and the README matches it.
