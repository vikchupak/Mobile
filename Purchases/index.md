# How RevenueCat and Google Play Store(or Apple App Store) are related

## What is Google Play Store’s role?

* Google Play Store is the **official app store and payment processor** on Android devices.
* It manages **subscriptions, purchases, billing, payment methods**, and **user accounts** (Google accounts).
* When a user buys a subscription in your Android app, **Google Play processes the payment and issues a purchase token and receipt**.

## What is RevenueCat’s role?

* RevenueCat is a **subscription management platform** that acts as an intermediary between your app and the app stores (Google Play, Apple App Store).
* It helps you:

  * **Track and validate purchases and subscriptions** (by verifying receipts with Google Play/Apple servers).
  * **Manage entitlements** and offer subscription benefits to users.
  * **Simplify cross-platform subscription logic** (Android + iOS + web).
  * **Provide analytics and reporting** for your revenue.
  * Handle **subscription status updates** (renewals, cancellations, refunds) in real time via server callbacks.

## How they work together

1. **User buys subscription in your app on Android.**
2. Google Play processes the payment and returns a **purchase token** to your app.
3. Your app sends this purchase token to RevenueCat.
4. RevenueCat verifies the purchase token with Google Play’s servers to confirm it’s valid.
5. RevenueCat records the purchase and updates the user’s subscription status in **its backend**.
6. **Your app queries RevenueCat to check if the user has an active subscription and unlocks premium features accordingly.**

---

## Summary

| Role                   | Google Play Store                   | RevenueCat                                 |
| ---------------------- | ----------------------------------- | ------------------------------------------ |
| Payment processing     | Handles actual payments and billing | Does not handle payments                   |
| Purchase validation    | Issues purchase tokens and receipts | Validates tokens with Google Play          |
| Subscription status    | Maintains billing status            | Aggregates status and notifies your app    |
| Entitlement logic      | None (just billing)                 | Decides who gets access based on purchases |
| Cross-platform support | Android only                        | Android, iOS, web, and more                |

# How RevenueCat’s App User ID and Google Play Store account are related

### 1. **RevenueCat App User ID**

* This is **your own app’s unique identifier for a user** — a string or number you assign to identify each user in your system.
* It’s completely **independent** of the user’s Google Play account.
* RevenueCat uses this ID to track purchases, subscriptions, and entitlements **within your app**.
* You send this App User ID to RevenueCat when your app initializes the SDK and when purchases happen.

### 2. **Google Play Store Account**

* This is the **Google account logged into the Google Play Store app on the user’s device**.
* It manages actual payments, subscriptions, billing, and purchase tokens.
* Purchases made via Google Play are linked to the Google Play account **used on the device at purchase time**.
* Google Play knows nothing about your app’s internal App User ID.

### 3. **How they connect during a purchase**

* When a user buys a subscription:

  * The purchase is made through **Google Play Store account** (payment method & billing).
  * Google Play returns a **purchase token** to your app.
  * Your app forwards this purchase token **along with the App User ID** to RevenueCat.
* RevenueCat uses the purchase token to verify the purchase with Google Play.
* RevenueCat links that purchase to the **App User ID** you provided, **not the Google Play account email**.

### 4. **Key consequences**

* If the user logs into your app with a different App User ID (e.g., different app account or email), RevenueCat treats it as a different customer, even if it’s the same Google Play account paying.
* If the user changes their Google Play account on the device, new purchases will be associated with the App User ID used in the app, but billing happens via the new Google Play account.
* **RevenueCat does not know or track the Google Play account email — it only knows the purchase token and the App User ID you give it.**

### Summary

| Aspect                  | RevenueCat App User ID                          | Google Play Store Account              |
| ----------------------- | ----------------------------------------------- | -------------------------------------- |
| What it is              | Your app’s unique user identifier               | Google account logged into Play Store  |
| Purpose                 | Track purchases & entitlements within your app  | Manage payment & billing               |
| Relation                | You assign and control this ID                  | Google controls this account           |
| Linking during purchase | Purchase token + App User ID sent to RevenueCat | Purchase linked to Google Play account |

# How RevenueCat’s App User ID is generated

### How App User ID is typically generated:

1. **Your app or backend assigns it** when a user creates an account or opens the app. Common ways include:

   * **UUID (Universally Unique Identifier):**
     A random unique string generated on device or server, e.g., `a3f47e92-2c41-4d1e-bc9f-f7c823a7a8a3`.

   * **Backend user ID:**
     If your app has user accounts, you can use your internal user database’s unique user ID (numeric or string).

   * **Email or username (less recommended):**
     Some apps use the user’s email or username as the App User ID — but this is risky if emails can change.

2. **It must be unique per user** in your system to properly track purchases and entitlements.

3. **You pass this App User ID to the RevenueCat SDK** when the user first opens the app or logs in.

### Best Practices:

* Use a **stable, immutable identifier** that never changes for the user, e.g., a database user ID or UUID stored at account creation.
* Avoid using emails or other mutable data as the App User ID, because if the user changes their email, it creates a new user in RevenueCat, fragmenting subscription data.
* Generate and assign the ID **on your backend** if possible, to maintain consistency and prevent spoofing.
* In anonymous user flows (no login), generate a **random UUID** on device and persist it locally until the user signs up.

### Example flow:

* User signs up → backend creates user account with `user_id = 123456`.
* Your app initializes RevenueCat SDK with `AppUserID = "123456"`.
* Purchases made by this user are tracked under `123456` in RevenueCat.

# Charging

If on a device there is Google Play Account with email A, and on this device multiple users sign-up in my app with email B and email C. Users with email B and C can both buy subscription, money will still be charged from email A?

Yes, that's correct — and it’s a key behavior to understand when managing subscriptions through **Google Play**:

---

### ✅ **How Google Play handles subscriptions:**

* **Subscriptions are tied to the Google Play account on the device**, not directly to your app's user accounts.
* So even if users **B** and **C** are signed into **your app** with their own email addresses, if they purchase a subscription via Google Play:

  * **The charge goes to the Google Play account (email A)** currently signed into the Play Store on the device.
  * RevenueCat will see the purchase and assign it to whatever **App User ID** is active in your app at the moment of purchase.

---

### 🧠 Implications for your app:

1. **RevenueCat only tracks purchases per App User ID**, not per Play Store account — so:

   * If user **B** is signed in to your app with App User ID = `userB_id`, and makes a purchase,
   * RevenueCat links the purchase to `userB_id`, even though it was charged via Play account `emailA@gmail.com`.

2. **User C** can also purchase a separate subscription, but it would still be **charged to email A’s Play account**, unless:

   * User C switches the **Google Play account** on the device before purchasing.

---

### 🚨 Problem scenario:

* A parent (Google Play Account: `parent@gmail.com`) gives their child access to the device.
* The child logs into your app with `child@gmail.com`, buys a subscription.
* The charge appears on the parent’s Play Store billing — which can cause confusion or refund requests.

---

### 🛠 Recommended actions:

* Let users know which Google Play account will be charged.
* Detect and show the current Play Store account email in-app (if possible).
* In cases where multiple users share a device, inform them that **subscriptions are tied to the Play Store account**, not app login.

---

### 🙋‍♂️ Can users change which Google Play account is used?

* Yes — but they have to **switch the Play account in Google Play manually**, before making a purchase.
* **Your app can’t change which Play Store account is used; only the user can.**
