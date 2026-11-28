# Temano plugin for Claude Code

When Claude Code gets stuck at a passkey, password prompt, CAPTCHA or confirmation on your Mac, it sends you a private link. You open your Mac on your iPhone, finish the step and Claude Code continues.

## Install

In Claude Code:

```
/plugin marketplace add tiipeng/temano-releases
/plugin install temano@temano
```

Or in a terminal:

```sh
claude plugin marketplace add tiipeng/temano-releases
claude plugin install temano@temano
```

## What else you need

1. **Temano for Mac** from [temano.tipengstudio.com](https://temano.tipengstudio.com). Open it and choose **Set up agents…**. This installs the `temano` command the plugin uses.
2. **Temano for iPhone**, paired with your Mac through the QR code in the Mac app.

Everything stays on your own network: at home the iPhone connects directly, away from home through Tailscale. There is no Temano server and no account.

## How a handoff works

1. Claude Code creates a request and posts the link in the chat.
2. You tap the link, see your Mac's screen and finish the step.
3. You reply "done" (or tap "Back to agent" in Temano).
4. Claude Code checks that the step worked and continues.

Claude Code never receives what you type during the handoff.
