Background info on how subscriptions work.

### The main point is to distinguish:
- **Store Account** → The account you use to sign in to the App Store (Apple) or Google Play. This account is what actually gets charged for the subscription.
- **In-App Account** → The account you use to sign in to the app itself.
When a user subscribes, the payment always comes from the Store Account — this is true for all apps.

**Example**: If a device is signed into Google Play with email A (Store Account), and inside the app two users sign up (email B and email C, In-App Accounts), and both purchase subscriptions, email A will be billed twice — once for each subscription.

### Where Subscription Information Is Stored:
- Google Play Console / Apple App Store Connect → Store Account subscription & payment info (Apple/Google don’t show the account email to developers for privacy reasons).
- RevenueCat → In-App Account subscription & payment info.

### Changing Which Store Account Is Charged:
If a user wants to switch the Store Account used for payments, he must:
  1. Sign into the old Store Account and cancel the current subscription.
  2. Sign into the new Store Account and purchase a subscription while logged into the desired In-App Account.
- Note 1: This does not transfer app data between In-App Accounts — they remain separate.
- Note 2: Developers cannot directly change, transfer, or upgrade/downgrade store subscriptions for a user.
