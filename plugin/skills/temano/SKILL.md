---
name: temano
description: Get a human to finish a step on this Mac when you are blocked by a passkey, password prompt, CAPTCHA, "verify you are human" page, two-factor code, macOS permission dialog or any confirmation only the user may give. Creates a private link that opens the user's Mac on their iPhone, waits in the background, then resumes once they are done.
---

# Temano: hand a step to the human

Temano runs locally on this Mac. Its command-line tool is `~/.local/bin/temano`. The user opens the link in the Temano iPhone app and sees and controls this Mac's real screen. A headless browser is not visible there: put the step in a visible window first, without closing other sessions.

## Before the first request

Run `test -x ~/.local/bin/temano && echo ok`. If it is missing, Temano is not set up on this Mac. Tell the user, in their language, and stop:

> Temano isn't set up on this Mac yet. Install Temano for Mac from https://temano.tipengstudio.com, open it and choose "Set up agents…". Then install Temano on your iPhone and pair it with the QR code the Mac app shows.

Do not install anything yourself.

## Request and link

1. Pause the browser or Mac automation that is stuck. Pick a short reason without passwords, codes, tokens or private content. Write it in the user's language.
2. Create **one** request and keep its `id` and `url` for this task:

```sh
~/.local/bin/temano create --agent 'Claude Code' --reason 'Please confirm the passkey sign-in' --target mac --timeout 15m
```

At most `--timeout 15m`. The JSON answer comes back at once. `url` is the full private link including its `#` fragment: never shorten it or add parameters.

3. **The link has to reach the user.** They are often only on their phone and see your replies, not command output. A link that only appears in tool output has not been delivered.

   Right after creating the request, start exactly one background command (Bash with `run_in_background: true`):

   ```sh
   ~/.local/bin/temano wait REQUEST_ID
   ```

   It ends by itself once the request is `done`, `declined`, `failed` or `expired`. Then **end your turn immediately** with a short reply that contains the link, for example: "Please confirm the sign-in: [Open Mac](URL). Reply 'done' when you're finished." No loops with `--max-wait`, no further tool calls in this turn. You are resumed automatically when the background command ends.

4. Read the result:

- `done`: the user chose "Back to agent" in Temano. Check yourself whether the original sign-in or step actually worked before you continue. Once you have really resumed, run `~/.local/bin/temano confirm REQUEST_ID`. Never confirm before that.
- `declined`: the user declined. Do not continue on your own.
- `failed`: cancelled or connection lost. Do not continue on your own.
- `expired`: time ran out. Say so instead of creating new links automatically.
- Not found or service unreachable: not a success; name the cause. Requests do not survive a restart of the Temano service.

**The user replies in chat** (for example "done"): this is the normal path. Check the result yourself. If it worked, close the request with `~/.local/bin/temano complete REQUEST_ID` and then `~/.local/bin/temano confirm REQUEST_ID`; a background `wait` still running then ends with `done`. If it did not work, say so and leave the request open. Never keep waiting after the user has answered in chat.

If the task was called off: `~/.local/bin/temano cancel REQUEST_ID`. That only cancels this request; it does not mark anything as done.

## Limits

While the user has control, take no screenshots, input or browser actions on the affected target. Temano does not lock other agents out technically; they have to pause. The user types secrets themselves: never ask for passwords in chat, and never solve CAPTCHAs or passkeys automatically. A Temano link does not replace separate consent for purchases, publishing or other external actions. Do not change Tailscale, firewall or screen sharing settings for a handoff.
