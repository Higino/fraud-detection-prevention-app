# Flow 1 — Navigate the repo and create something new

Use this when you are looking for where a fact lives, or before you add a screen, a reason code, a platform, or a server behavior.

The animated map is [animations/navigate.html](animations/navigate.html). It highlights the same nodes this page names.

## What is in the repository

The product is specified before it is implemented. On this branch the tree is documentation.

```text
fraud-detection-prevention-app
├── README.md
└── docs/
    └── training/
        ├── README.md
        ├── 01-navigate-and-create.md      ← this page
        ├── 02-how-the-core-is-wired.md
        └── animations/                    ← the two flows, in the browser
```

The version 1 specification is on draft [PR #1](https://github.com/Higino/fraud-detection-prevention-app/pull/1), branch `cursor/ios-sms-filter-spec-901a`:

```text
docs/
├── ios-sms-filter-spec.md          what the app promises and the server contract
├── ios-mobile-architecture.md      processes, targets, FilterCore
└── ios-user-flow.md                what a person does in Settings and in Messages
```

Read the specification for a rule. Read the architecture for which target owns a type. Read the user flow for what a person can see. Read this training for where a new change is allowed to land.

## What is specified and not checked in

No Xcode project and no server are in the repo yet. When they arrive, they follow the target split already written down. Do not invent a shared library between the containing app and the extension.

```text
SMSFilter.app
├── SMSFilter                         containing app, SwiftUI, iOS 16
│   ├── SMSFilterApp                  NavigationStack, no model object
│   ├── EnableFilterView              Settings path, one-filter limit, Junk is the warning
│   ├── DataSentView                  sender and raw text are transmitted by iOS
│   ├── FilteredMessageView           where Junk is, and to use the bank's own app
│   └── SettingsOpener                opens this app's Settings page
├── SMSFilter.entitlements
│   └── messagefilter:<classification-host>
└── PlugIns/MessageFilter.appex
    ├── MessageFilterExtension        the only type that touches IdentityLookup
    ├── FilterCore                    linked by the extension and its tests, not by the app
    │   ├── QueryGate.canDefer
    │   ├── ServerDecisionDecoder.decode
    │   ├── ActionMapper.map
    │   └── decide(DeferralOutcome)
    └── Info.plist
        ILMessageFilterExtensionNetworkURL = https://<host>/<path>
```

The classification server is a contract, not a directory yet. Its stages are redact, domain rules, Laya, policy. They are drawn in [flow 2](02-how-the-core-is-wired.md).

## Where a new thing goes

Write one sentence before you add files:

> This change lives in [target]. It leaves [forbidden capability] untouched. Acceptance: [what a person or a test observes].

| You are adding | It lives in | It stays out of |
|---|---|---|
| A product rule or a new platform | `docs/`, as a specification | The iOS extension, until the spec names the target |
| A screen or a sentence the person reads | `SMSFilter` containing app | FilterCore, the extension, any URL session |
| A mapping from server JSON to inbox or Junk | `FilterCore` | `URLSession`, `IdentityLookup`, disk |
| The call that reaches the network | `MessageFilterExtension`, one `deferQueryRequestToNetwork` | A second deferral, a socket in the extension |
| A new piece of evidence or a threshold | The server policy, plus a strict `reason` case if the JSON grows | Logs of sender or text, the extension's branch on `reason` |

FilterCore is a static library so the decision can be tested without launching Messages. The containing app does not link it. The extension and the app do not share an app group, files, or memory.

## Worked examples

The animation uses these six. The blocked ones are part of the lesson: knowing where not to put a feature is the navigation skill.

### 1. Record a product decision

Lives in `docs/`. A decision is a markdown spec with the behavior, what is out of scope, and an acceptance check. Application targets do not appear in the same change unless the spec already names them.

Acceptance: a reader can tell what the phone does, what the server does, and which failures leave the message in the inbox.

### 2. Add a screen to the containing app

Lives in `SMSFilter`, on the one `NavigationStack`. Copy is the product. The screen has no account, no message list, and no score.

Leave FilterCore, the extension, and every network client alone. The app cannot learn that a message was junked.

Acceptance: the new screen never describes a message as credible, safe, or verified. It does not deep-link into Messages settings. `SettingsOpener` opens this app's own Settings page only.

### 3. Add a server reason code

Lives in the server policy and in `ServerDecision.Reason` inside FilterCore. Document the row in the specification in the same change.

```swift
struct ServerDecision: Decodable {
    enum Action: String, Decodable { case junk, none }
    enum Reason: String, Decodable {
        case linkDomain = "link_domain"
        case smishingLanguage = "smishing_language"
        case none
    }
    let action: Action
    let reason: Reason
}
```

Decoding is strict. An unknown `action` or `reason` fails the decode. `decide` then returns inbox. `MessageFilterExtension` maps inbox to `.none` and junk to `.junk` with sub-action `.none`. It does not branch on `reason`. `is_spam` does not file Junk.

Acceptance: a body with the new reason and `action: junk` can be junked, and a body with an unrecognized reason stays in the inbox.

### 4. Let the extension open the connection

This does not get a target. The IdentityLookup extension cannot open a connection. Messages is the only process that posts. The extension calls `deferQueryRequestToNetwork` once, after `QueryGate.canDefer` has seen both a sender and a body. A missing field never consumes the single deferral.

`ILMessageFilterExtensionNetworkURL` is fixed in the extension `Info.plist`. Every install posts to that host. The containing app's associated-domains entitlement carries `messagefilter:<host>`. If the host's Apple App Site Association file does not authorize this team and bundle id, deferral fails and FilterCore is never asked to read a body. The message stays in the inbox.

### 5. Add Android

Lives in a new specification under `docs/`. It is a different surface: a default SMS app can redact on the phone and can show a warning in the inbox. The iOS extension cannot.

Leave the iOS spec's rules in place. Junk remains the only iOS warning. One deferral, server-side redaction, and delivery to the inbox when the server cannot decide stay as written. Android does not classify iMessage, and this filter does not classify Contacts on either platform in version 1.

### 6. Confirm the event with the bank

This version does not grow a bank client inside the filter. The classifier answers whether the text looks like a fake bank request. It does not answer whether the bank registered the charge, the block, or the login. A bank API would be a new problem and a new spec, with an account model the extension is not allowed to hold.

The standing instruction already lives on `FilteredMessageView`: a text that asks for a code, a payment, or a login is handled by opening the bank's own app.

## Checklist for a change

1. Name the target from the table above.
2. State the capability you are leaving untouched: socket, app group, on-device model, message cache, "safe" copy, promotional folder.
3. If the server JSON changes, update the spec, the decoder, and the strict-decode failure path together.
4. If a person will read new words, check them against acceptance item 9 of the specification: no screen describes a message as credible, safe, or verified.
5. Point the acceptance line at something observable: Junk, inbox, a decode failure, or a log line that has neither sender nor text.

## Related

Flow 2 is the path a message takes once the targets exist: [02-how-the-core-is-wired.md](02-how-the-core-is-wired.md).
