# StoreKit 2 Handles Purchases. This Handles Everything Else

Last year I released several apps to the App Store and all of them used In-App Purchases. Some of them were simple one-time purchases while others used recurring subscriptions.

I could have used a third party service to handle subscriptions and receipt validation, but I decided to explore StoreKit 2 instead. To my surprise, StoreKit 2 was actually pleasant to work with. Once I integrated it into one app, repeating the process in my other apps became much easier and faster.

But StoreKit 2 is only one part of the story.

What happens after the purchase? How do you know when a subscription renews, expires, gets refunded, or fails to renew? How do you track purchases once they happen outside the app?

This is where App Store Server Notifications come in.

App Store Server Notifications allow Apple to notify your backend whenever important events happen related to your in-app purchases and subscriptions. Your server can receive updates for new purchases, renewals, cancellations, refunds, billing issues, expired subscriptions, and more.

Once your server receives these notifications, you can do pretty much anything you want with the data. You can save transactions to a database, build dashboards, trigger analytics events, unlock features, monitor refunds, or even send yourself Slack notifications whenever a sale happens.

In this article, we are going to build a lightweight ExpressJS endpoint that receives App Store Server Notifications, decodes the signed payloads sent by Apple, and extracts the actual transaction and renewal information from the notification payloads.

<!-- Book Banner: SwiftUI Architecture Book -->
<div class="azam-book-banner" role="region" aria-label="SwiftUI Architecture Book Banner">
  <div class="azam-book-banner__inner">
    <div class="azam-book-banner__cover">
      <!-- Replace the src with your real book cover image -->
      <img
        src="https://azamsharp.school/images/swiftui-architecture-book-cover.png"
        alt="SwiftUI Architecture book cover"
        loading="lazy"
      />
    </div>
    <div class="azam-book-banner__content">
      <p class="azam-book-banner__eyebrow">SwiftUI Architecture Book</p>
      <h3 class="azam-book-banner__title">Patterns and Practices for Building Scalable Applications</h3>
      <p class="azam-book-banner__subtitle">
        A practical guide to building SwiftUI apps that stay clean as they grow.
      </p>
      <div class="azam-book-banner__actions">
        <a class="azam-book-banner__button" href="https://azamsharp.school/swiftui-architecture-book.html" target="_blank" rel="noopener">
          Get the book
        </a>
      </div>
    </div>
  </div>
</div>

<style>
  .azam-book-banner {
    --bg1: #0b1220;
    --bg2: #111a2d;
    --text: rgba(255, 255, 255, 0.92);
    --muted: rgba(255, 255, 255, 0.74);
    --border: rgba(255, 255, 255, 0.12);
    --shadow: 0 18px 45px rgba(0, 0, 0, 0.28);
    --accent: #6ee7b7; /* tweak to match your brand */
    --accent2: #60a5fa;

    margin: 22px 0;
    color: var(--text);
    border: 1px solid var(--border);
    border-radius: 16px;
    overflow: hidden;
    background: radial-gradient(1200px 600px at 10% 0%, rgba(96, 165, 250, 0.22), transparent 60%),
                radial-gradient(900px 500px at 90% 30%, rgba(110, 231, 183, 0.18), transparent 60%),
                linear-gradient(135deg, var(--bg1), var(--bg2));
    box-shadow: var(--shadow);
  }

  .azam-book-banner__inner {
    display: grid;
    grid-template-columns: 132px 1fr;
    gap: 18px;
    padding: 18px;
    align-items: center;
  }

  .azam-book-banner__cover {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .azam-book-banner__cover img {
    width: 132px;
    height: auto;
    border-radius: 12px;
    border: 1px solid rgba(255,255,255,0.14);
    box-shadow: 0 14px 28px rgba(0,0,0,0.35);
    background: rgba(255,255,255,0.04);
  }

  .azam-book-banner__eyebrow {
    margin: 0 0 6px 0;
    font-size: 12px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--muted);
  }

  .azam-book-banner__title {
    margin: 0 0 8px 0;
    font-size: 18px;
    line-height: 1.25;
  }

  .azam-book-banner__subtitle {
    margin: 0 0 14px 0;
    font-size: 14px;
    line-height: 1.55;
    color: var(--muted);
    max-width: 62ch;
  }

  .azam-book-banner__actions {
    display: flex;
    gap: 12px;
    align-items: center;
    flex-wrap: wrap;
  }

  .azam-book-banner__button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 10px 14px;
    border-radius: 12px;
    font-weight: 700;
    font-size: 14px;
    color: #071018;
    text-decoration: none;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border: 0;
    box-shadow: 0 10px 22px rgba(0,0,0,0.28);
    transition: transform 140ms ease, filter 140ms ease;
  }

  .azam-book-banner__button:hover {
    transform: translateY(-1px);
    filter: brightness(1.02);
  }

  .azam-book-banner__link {
    font-size: 14px;
    color: rgba(255,255,255,0.86);
    text-decoration: none;
    border-bottom: 1px solid rgba(255,255,255,0.22);
    padding-bottom: 2px;
    transition: border-color 140ms ease, color 140ms ease;
  }

  .azam-book-banner__link:hover {
    color: rgba(255,255,255,0.95);
    border-color: rgba(255,255,255,0.45);
  }

  /* Mobile */
  @media (max-width: 520px) {
    .azam-book-banner__inner {
      grid-template-columns: 1fr;
      text-align: left;
    }

    .azam-book-banner__cover {
      justify-content: flex-start;
    }

    .azam-book-banner__cover img {
      width: 120px;
    }
  }
</style>

### Configuring App Store Server Notifications

Before implementing the endpoint, you need to configure App Store Server Notifications in App Store Connect.

You can find this under your app in:

```txt
App Store Connect
    → Apps
    → Your App
    → App Information
    → App Store Server Notifications
```

Apple allows you to configure separate URLs for Sandbox and Production notifications.

Use the Sandbox URL while testing locally and the Production URL for real App Store events.

If you are running your Express server locally, Apple will not be able to access something like:

```txt
http://localhost:8080
```

To solve this, you can use a tool like ngrok to expose your local server through a secure public HTTPS URL.

For example:

```txt
https://abc123.ngrok-free.app
```

This URL forwards requests directly to your local Express server, allowing Apple to send Sandbox notifications to your machine during development and testing. For production you can deploy your app to a hosting provider. I used Heroku for my server. 

You can use the same endpoint for multiple apps if you want. In that case, your backend will need to determine which app sent the notification by checking values like:

```js
bundleId
appAppleId
productId
```

from the notification payload.

### Implementing the Server 

You can implement your App Store Server Notification endpoint using any backend framework or programming language. In this article, I will use ExpressJS with JavaScript because it is lightweight, easy to set up, and works great for handling HTTP endpoints.

The core responsibility of the endpoint is to receive the `signedPayload` sent by Apple, decode it, and then extract the transaction and subscription information contained inside the notification.

One important thing to understand is that Apple does not send a simple JSON payload with all the information directly available. Instead, Apple sends multiple layers of signed JWT payloads. After decoding the outer payload, you will often discover additional encoded payloads nested inside it, such as `signedTransactionInfo` and `signedRenewalInfo`. Those payloads must also be decoded separately to access the actual transaction and renewal details.

In other words, decoding the notification is a multi-step process:

```txt
signedPayload
    ↓
outer notification payload
    ↓
signedTransactionInfo
    ↓
transaction details

signedRenewalInfo
    ↓
renewal details
```

Let's start by looking at the endpoint implementation.

#### 1. Creating the Endpoint

Start with the basic route.

```js
app.post("/asn", async (req, res) => {
  try {
    const { signedPayload } = req.body;

  } catch (err) {
    console.error("ASN error:", err);
    res.sendStatus(500);
  }
});
```

This endpoint is where Apple will send App Store Server Notifications. The route can be named anything, but `/asn` keeps it short and clear. Apple will send a `POST` request to this endpoint whenever something important happens with an in-app purchase or subscription.

The request body contains a `signedPayload` property. This is not plain JSON. It is a signed JWT sent by Apple. That means we first need to decode it before we can read the actual notification data.

#### 2. Reading the `signedPayload`

```js
const { signedPayload } = req.body;

if (!signedPayload) {
  return res.status(400).json({
    error: "Missing signedPayload"
  });
}
```

The `signedPayload` is the main value Apple sends to your server. It contains the App Store Server Notification data.

It is a good idea to check if it exists before trying to decode it. If the request does not contain a `signedPayload`, then the request is invalid and we return a `400 Bad Request`.

#### 3. Decoding the Outer Payload

```js
const { header, payload } = await decodeSignedPayload(signedPayload);

console.log("ASN Header:", header);
console.log("ASN Payload:", payload);
```

The first decode gives us the outer App Store notification.

The `header` tells us how the JWT was signed. For example, Apple uses `ES256`.

The `payload` contains the actual notification information, such as the notification type, subtype, version, signed date, and the `data` object.

This outer payload answers the first important question:

```txt
What happened?
```

For example, the notification may tell us that a subscription expired, renewed, failed to renew, or was refunded.

#### 4. Reading the Main Notification Fields

```js
const {
  notificationType,
  subtype,
  data,
  environment,
  notificationId,
  signedDate,
  version
} = payload;
```

After decoding the outer payload, we extract the fields we care about.

`notificationType` tells us what happened. Examples include `SUBSCRIPTION_EXPIRED`, `DID_RENEW`, `DID_FAIL_TO_RENEW`, or `REFUND`.

`subtype` gives more context about the notification. For example, a subscription expiration may have a subtype of `VOLUNTARY`, which means the user chose not to continue the subscription.

`data` contains app and transaction related information.

`notificationId` is useful for logging and avoiding duplicate processing.

`signedDate` tells us when Apple signed the notification.

`version` tells us the App Store Server Notification version.

#### 5. Decoding `signedTransactionInfo`

```js
let transaction = null;

if (data?.signedTransactionInfo) {
  const decodedTx = await decodeSignedPayload(data.signedTransactionInfo);
  transaction = decodedTx.payload;

  console.log("Transaction payload:", transaction);
}
```

The outer notification payload does not contain all transaction details directly.

Instead, Apple places another signed JWT inside:

```js
data.signedTransactionInfo
```

This value must also be decoded.

The transaction payload answers the second important question:

```txt
Which transaction did this happen to?
```

After decoding it, you can access details like:

```js
transaction.productId
transaction.transactionId
transaction.originalTransactionId
transaction.purchaseDate
transaction.expiresDate
transaction.environment
transaction.price
transaction.currency
```

This is usually the most important part of the notification because it tells you which product or subscription was affected.

#### 6. Decoding `signedRenewalInfo`

```js
let renewalInfo = null;

if (data?.signedRenewalInfo) {
  const decodedRenewal = await decodeSignedPayload(data.signedRenewalInfo);
  renewalInfo = decodedRenewal.payload;

  console.log("Renewal payload:", renewalInfo);
}
```

Some notifications also contain `signedRenewalInfo`.

This is another signed JWT that contains subscription renewal details.

The renewal payload answers:

```txt
What is the renewal state of this subscription?
```

For example, it may tell you whether auto-renew is enabled or disabled.

Common fields include:

```js
renewalInfo.autoRenewStatus
renewalInfo.autoRenewProductId
renewalInfo.priceIncreaseStatus
```

This is useful when you want to know whether the subscription is expected to continue renewing in the future.

#### 7. Figuring Out the Environment

```js
const rawDataEnvironment = data?.environment ?? null;
const transactionEnvironment = transaction?.environment ?? null;

const effectiveEnvironment =
  transactionEnvironment ??
  rawDataEnvironment ??
  environment ??
  "UNKNOWN";
```

Apple may provide the environment in more than one place.

Sometimes it appears inside `data.environment`. Sometimes it may be available from the decoded transaction. In some cases, you may also check the top-level `environment`.

To avoid guessing, we create one value called `effectiveEnvironment`.

This gives us a single environment value to use for logging and filtering.

#### 8. Creating a Friendly Environment Label

```js
const envLabel =
  effectiveEnvironment === "Sandbox"
    ? "Sandbox 🧪"
    : effectiveEnvironment === "Production"
      ? "Production 💸"
      : effectiveEnvironment;
```

This part is not required, but it makes logs and Slack messages easier to scan.

If the notification came from Sandbox, we display it as:

```txt
Sandbox 🧪
```

If it came from Production, we display it as:

```txt
Production 💸
```

This is especially useful when you are testing locally or receiving both sandbox and production events.

#### 9. Building the Notification Message

```js
const subtypePart = subtype ? ` (${subtype})` : "";

const lines = [
  `🔔 APP NOTIFICATION ${notificationType}${subtypePart}`,
  `• Environment: ${envLabel}`,
  `• Version: ${version}`,
  notificationId ? `• Notification ID: ${notificationId}` : null,
  signedDate ? `• Signed at: ${new Date(signedDate).toISOString()}` : null
];
```

Here we start building a human-readable message.

Instead of sending raw JSON to Slack, we create a clean summary.

The message starts with the notification type and optional subtype. Then we include useful metadata like the environment, version, notification ID, and signed date.

Using an array makes it easy to conditionally add lines and remove empty values later.

#### 10. Adding Transaction Details

```js
if (transaction) {
  const planLabel = getPlanLabel(transaction.productId);

  lines.push(
    "",
    "— Transaction —",
    `• Product: ${planLabel}`,
    `• Product ID: ${transaction.productId}`,
    `• Transaction ID: ${transaction.transactionId}`,
    transaction.originalTransactionId
      ? `• Original Transaction ID: ${transaction.originalTransactionId}`
      : null,
    transaction.type ? `• Type: ${transaction.type}` : null,
    transaction.purchaseDate
      ? `• Purchase Date: ${new Date(transaction.purchaseDate).toISOString()}`
      : null,
    transaction.expiresDate
      ? `• Expires Date: ${new Date(transaction.expiresDate).toISOString()}`
      : null,
    transaction.transactionReason
      ? `• Transaction Reason: ${transaction.transactionReason}`
      : null,
    transaction.price != null && transaction.currency
      ? `• Price: ${transaction.price} ${transaction.currency}`
      : null
  );
} else {
  lines.push("", "— No transaction info in this notification —");
}
```

If transaction information is available, we add it to the message.

The `productId` is usually something like:

```txt
com.yourapp.subscription.monthly
```

That is useful for code, but not always friendly for humans. So we pass it to a helper function called `getPlanLabel`.

For example:

```js
getPlanLabel("com.yourapp.subscription.monthly")
```

could return:

```txt
Monthly Plan
```

This section also includes transaction ID, original transaction ID, purchase date, expiration date, transaction reason, and price if Apple provides those values.

The `originalTransactionId` is very important for subscriptions because it connects multiple renewal transactions back to the same original subscription.

#### 11. Adding Renewal Details

```js
if (renewalInfo) {
  lines.push(
    "",
    "— Renewal Info —",
    renewalInfo.autoRenewStatus != null
      ? `• Auto Renew: ${renewalInfo.autoRenewStatus === 1 ? "ON" : "OFF"}`
      : null,
    renewalInfo.autoRenewProductId
      ? `• Next Product: ${getPlanLabel(renewalInfo.autoRenewProductId)}`
      : null,
    renewalInfo.priceIncreaseStatus != null
      ? `• Price Increase Status: ${renewalInfo.priceIncreaseStatus}`
      : null
  );
}
```

If renewal information is available, we add it to the message.

`autoRenewStatus` tells us whether the subscription is set to renew again.

A value of `1` means auto-renew is on.

A value of `0` means auto-renew is off.

`autoRenewProductId` tells us which product the subscription will renew into.

`priceIncreaseStatus` can be useful if the subscription is affected by a price increase and Apple is waiting for user consent.

#### 12. Creating the Final Text Message

```js
const text = lines.filter(Boolean).join("\n");
```

Throughout the message-building process, we added some values conditionally. Some of those values may be `null`.

Before sending the message, we remove all empty values using:

```js
filter(Boolean)
```

Then we join all remaining lines with newline characters.

This gives us a clean text message that can be sent to Slack.

#### 13. Ignoring Sandbox Notifications

```js
if (effectiveEnvironment !== "Production") {
  console.log(
    `ASN ignored — NOT production event (${effectiveEnvironment})`
  );

  return res.sendStatus(200);
}
```

In this implementation, we only want to send production notifications to Slack.

Sandbox notifications are useful for testing, but they can create a lot of noise.

The important part is that we still return `200 OK`.

Apple expects your server to acknowledge the notification. Even if you decide not to process the event, you should still return a successful response so Apple knows your server received it.

#### 14. Sending the Message to Slack

```js
await axios.post(SLACK_WEBHOOK_URL, { text });

res.sendStatus(200);
```

If the notification is from production, we send the formatted message to Slack using an incoming webhook.

After Slack receives the message, we return `200 OK` to Apple.

This tells Apple that the notification was successfully handled.

#### 15. Handling Errors

```js
catch (err) {
  console.error("ASN error:", err);
  res.sendStatus(500);
}
```

If something goes wrong while decoding the payload or sending the Slack message, we log the error and return `500`.

During development, this is helpful because it lets you catch invalid payloads, malformed JWT values, missing request body data, or issues with the Slack webhook.

In production, you may also want to store failed notifications in logs or monitoring tools so you can investigate them later.

### Source Code 

You can download the source code from this [Gist](https://gist.github.com/azamsharpschool/812022c3cdb6a0898edd0cebcc34e3fe). The implementation is intentionally lightweight and meant to help you understand the overall flow of App Store Server Notifications. In my real applications, I have added additional functionality such as app-specific handling, price formatting, database persistence, Slack integrations, and more.

### Conclusion 

App Store Server Notifications give you a powerful way to monitor what is happening with your subscriptions and in-app purchases outside the app itself. Instead of depending completely on the device, you can move important subscription logic to your server and react to events in real time.

In this article, we built an Express endpoint that receives notifications from Apple, decodes the outer signedPayload, and then decodes the nested signedTransactionInfo and signedRenewalInfo payloads to access the actual transaction and renewal details.

Once you have access to this information, you can do pretty much anything you want. You can save transactions to a database, build dashboards, monitor refunds, track renewals, send Slack notifications, or automate internal workflows.

The implementation shown in this article is intentionally simple so you can focus on understanding how App Store Server Notifications work without getting distracted by too much infrastructure code. In a real production application, you will most likely expand this further by verifying JWT signatures, storing events in a database, preventing duplicate processing, and building a more complete subscription management system.

Hopefully, this article gives you a good starting point for integrating App Store Server Notifications into your own applications.


