# iOS user flow

How a person uses the SMS filter. The containing app is the setup. Messages is where a text is kept or filed under Junk. This follows the [specification](ios-sms-filter-spec.md).

```mermaid
flowchart TD
  Start([Install and open the app]) --> Enable[Enable screen: where to turn the filter on]
  Enable --> OtherScreens[Optional: What is sent, and If a message was filtered]
  Enable --> Settings[Settings, Apps, Messages, Unknown and Spam, SMS Filtering]
  Settings --> Chosen{Select this app?}
  Chosen -->|Not selected| Off[Unknown texts stay in the inbox and are not checked]
  Chosen -->|Selected. Any other SMS filter is replaced| Ready[Phone is used as usual]
  Ready --> Arrives[A message arrives in Messages]
  Arrives --> Channel{SMS or MMS from a sender not in Contacts?}
  Channel -->|iMessage or a Contact| Normal[It appears in the conversation]
  Channel -->|Yes| Decide{Classification}
  Decide -->|Server says junk| JunkFolder[The text is under Junk, not in the conversation list]
  Decide -->|Server says none, or the request fails| Inbox[The text stays in the inbox]
  JunkFolder --> Review[Open Junk and read it there]
  Review --> RealBank{A genuine bank text?}
  RealBank -->|Yes| Restore[Move it back, then open the bank app]
  RealBank -->|No| Leave[Leave it in Junk. Do not open the link, call the number, or read out a code]
  Inbox --> Asks{It asks to pay, log in, or read a code?}
  Asks -->|Yes| BankApp[Open the bank app and ignore the link in the text]
  Asks -->|No| Read[Read it as an ordinary text]
```

## What the person can and cannot see

- The app has no switch. Selection happens in Settings, and the app cannot tell whether it is the active filter.
- A text left in the inbox has not been marked safe. The screens never say credible, verified, or safe.
- Junk is the only signal. There is no banner on the message.
- A text from someone in Contacts, and every iMessage, never enters this flow.
- Offline, or any server failure, takes the inbox branch.
