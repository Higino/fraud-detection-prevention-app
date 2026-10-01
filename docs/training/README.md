# Developer training

Two flows. Read them in order the first time. After that, use flow 1 when you are about to add something, and flow 2 when you need to see where a message goes.

| Flow | Written guide | Animation |
|---|---|---|
| 1. Navigate the repo and create something new | [01-navigate-and-create.md](01-navigate-and-create.md) | [animations/navigate.html](animations/navigate.html) |
| 2. How the classification core is wired to the phone | [02-how-the-core-is-wired.md](02-how-the-core-is-wired.md) | [animations/wired.html](animations/wired.html) |

Open [animations/index.html](animations/index.html) in a browser for the animated versions. GitHub will not run those pages. The markdown stands on its own.

## What "the core" means here

The classification core is two pieces that share one contract:

- **FilterCore**, the on-device decision function inside the Message Filter extension. It never sees IdentityLookup types and never opens a socket.
- The **server pipeline**: redact secrets, apply domain rules, ask Laya, apply the junk-or-none policy.

The phone is wired to that core through Apple's message-filter deferral. Messages, not the extension, posts the sender and the raw text. This repository has no `core-v2` package and no IoT bus. Flow 2 is the wiring that does exist.

## Source of the rules

Training restates the version 1 specification so a developer can work from this branch. The normative write-up, the process diagram, and the user flow live on draft [PR #1](https://github.com/Higino/fraud-detection-prevention-app/pull/1):

- [iOS SMS filter specification](https://github.com/Higino/fraud-detection-prevention-app/blob/cursor/ios-sms-filter-spec-901a/docs/ios-sms-filter-spec.md)
- [On-device architecture](https://github.com/Higino/fraud-detection-prevention-app/blob/cursor/ios-sms-filter-spec-901a/docs/ios-mobile-architecture.md)
- [User flow](https://github.com/Higino/fraud-detection-prevention-app/blob/cursor/ios-sms-filter-spec-901a/docs/ios-user-flow.md)

If a training page and that specification disagree, the specification wins. Update the training in the same change.
