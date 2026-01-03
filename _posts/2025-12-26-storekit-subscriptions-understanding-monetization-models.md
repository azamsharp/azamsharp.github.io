
# StoreKit Subscriptions: A Practical Guide  
## Part 1: Understanding Different Monetization Models 

I started iOS development back in 2010. My first app was a kids game called ABC Pop, which I built with my wife. She recorded all the audio for it. After that, I went on to release dozens of apps on the App Store, most notably Vegetable Tree Gardening Guide.

Vegetable Tree was featured by Apple multiple times and was also mentioned in several online magazines. Looking back, all of my apps used a one time purchase model. You paid once and got access to the app for life.

In this article, we are going to go over different monetization models available with Apple. 

<!-- AzamSharp Ad: StoreKit 2 Course -->
<div class="az-ad">
  
  <div class="az-ad__content">
    <div class="az-ad__title">StoreKit 2 for In-App Purchases & Subscriptions
    </div>
    <div class="az-ad__text">
      Learn how to ship purchases the right way: products, paywalls, purchase flow, entitlements, and restoring access.
      Practical, real-world, and focused on clean Swift code.
    </div>
<a
    class="az-ad__button"
    href="https://azamsharp.teachable.com/p/storekit-2-for-in-app-purchases-and-subscriptions"
    target="_blank"
    rel="noopener"
  >
    Enroll Now →
  </a>
  </div>
</div>


<style>
  .az-ad {
    --bg: #0b1220;
    --card: rgba(255, 255, 255, 0.06);
    --border: rgba(255, 255, 255, 0.14);
    --text: rgba(255, 255, 255, 0.86);
    --muted: rgba(255, 255, 255, 0.70);
    --accent: #34d399;

    font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji",
      "Segoe UI Emoji";
    background: radial-gradient(1200px 500px at 20% -10%, rgba(52, 211, 153, 0.22), transparent 60%),
      linear-gradient(180deg, rgba(255, 255, 255, 0.06), rgba(255, 255, 255, 0.02));
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 16px;
    color: var(--text);
    max-width: 720px;
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.25);
    position: relative;
    overflow: hidden;
  }

  .az-ad__badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.4px;
    text-transform: uppercase;
    color: rgba(0, 0, 0, 0.78);
    background: linear-gradient(90deg, var(--accent), #60a5fa);
    padding: 6px 10px;
    border-radius: 999px;
    margin-bottom: 10px;
  }

  .az-ad__title {
    font-size: 18px;
    font-weight: 800;
    line-height: 1.2;
    margin: 0 0 8px 0;
  }

  .az-ad__text {
    font-size: 14px;
    line-height: 1.45;
    color: var(--muted);
    margin: 0 0 14px 0;
  }

  .az-ad__cta {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
  }

  .az-ad__button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 10px 14px;
    border-radius: 12px;
    font-weight: 800;
    text-decoration: none;
    color: rgba(0, 0, 0, 0.85);
    background: linear-gradient(90deg, var(--accent), #60a5fa);
    border: 1px solid rgba(255, 255, 255, 0.18);
    transition: transform 160ms ease, filter 160ms ease;
    will-change: transform;
  }

  .az-ad__button:hover {
    transform: translateY(-1px);
    filter: brightness(1.03);
  }

  .az-ad__button:active {
    transform: translateY(0px);
  }

  .az-ad__note {
    font-size: 12px;
    color: rgba(255, 255, 255, 0.62);
  }

  @media (prefers-color-scheme: light) {
    .az-ad {
      --text: rgba(0, 0, 0, 0.86);
      --muted: rgba(0, 0, 0, 0.66);
      background: radial-gradient(900px 400px at 20% -10%, rgba(52, 211, 153, 0.25), transparent 60%),
        linear-gradient(180deg, rgba(0, 0, 0, 0.03), rgba(0, 0, 0, 0.01));
      box-shadow: 0 10px 24px rgba(0, 0, 0, 0.12);
    }
    .az-ad__note {
      color: rgba(0, 0, 0, 0.55);
    }
  }
</style>


### One Time Purchase 

One time purchase are exactly as they sound like. User pay for the app one time and they get lifetime access to the app. This is great deal for the user but in most cases not for the developer. The main reason is that the app needs OS updates, maintenance, features and more and a one time payment cannot justify the cost behind all the work a developer needs to perform. 

That said, there are still apps that make sense as a one time purchase, especially when the scope is limited or the app does not require frequent updates.

### Renewable Subscriptions 

Apple introduced auto renewable subscriptions in 2011 and later announced StoreKit 2 with async APIs and modern transaction handling. Today, most of my apps are subscription based. Subscriptions provide recurring revenue, which is far more sustainable for developers compared to a one time purchase.

It is also important to remember that apps require constant updates, feature enhancements, and bug fixes. Successful apps are like toddlers. You have to tend to them constantly.

### Consumables 

Consumables are purchases that a user can use at one time or at a later interval. I have seen consumables primarily used in games, where you can buy hints, level ups etc to make it easy for you to clear the stage. One common example of consumables is that you allow users to buy currency in your game. User can then use that currency to purchase items like rocket ship, faster car, different skins etc. 

Although consumables are primarily used in games, you can still utilize them in your non-game applications. In my apps [Instant Contracts](https://apps.apple.com/us/app/instant-contracts/id6756391187) and [Bullet Invoices](https://apps.apple.com/us/app/bullet-invoices/id6756518951), along with providing monthly and yearly subscription I also allow the user to create a single one-time invoice or contract. This option is perfect for people who are not creating invoices or contracts on regular basis but only want it   

You already have the **big three** covered. What you are missing are a few important Apple specific and real world variants that developers often overlook or misunderstand. I will list them in a way that fits naturally into your article and teaching style.

Below are the **additional monetization models you may want to include**.

---

### Non Renewing Subscriptions

Non renewing subscriptions are time based purchases that expire after a fixed period but do not renew automatically. For example, a user might purchase access for three months or one year, and once that period ends, access simply stops.

Apple does not manage renewals for this type of subscription. You are responsible for tracking expiration dates and handling access logic yourself.

These are useful when you want subscription like behavior without automatic billing. Common examples include exam prep apps, short term courses, or limited access programs.

They are less popular today because auto renewable subscriptions provide a much better user experience and require far less manual work.

---

### Non Consumable In App Purchases

Non consumables are one time purchases that unlock permanent functionality inside an app rather than unlocking the entire app itself.

Examples include:

* Removing ads
* Unlocking a pro mode
* Enabling advanced features
* Unlocking additional content packs

This model works well when you want to keep the app free to download but offer a permanent upgrade. Many developers also combine this with subscriptions, offering a lifetime unlock alongside monthly or yearly plans.

---

### Freemium Model

Freemium is not a StoreKit product type, but it is a very common monetization strategy that uses StoreKit behind the scenes.

In this model, the app is free to download and use with limitations. Payments are used to unlock more features, remove restrictions, or enhance the experience.

Freemium works especially well when users can experience real value before paying. Subscriptions, non consumables, and even consumables can all be used as part of a freemium strategy.

---

### Paywalls and Feature Gating

This is more of a design approach than a StoreKit product, but it is critical to mention.

Paywalls control when and how users are asked to pay. Feature gating limits what free users can access. These decisions often matter more than the actual product type.

Two apps can sell the same subscription but have completely different conversion rates based on how the paywall is designed.

This topic fits perfectly as a bridge into future parts of your series.

---

### Hybrid Models

Many successful apps do not rely on just one model.

Common combinations include:

* Subscription plus consumables
* Subscription plus lifetime unlock
* Freemium with optional non consumables
* Subscription with one time usage based purchases like you described for Instant Contracts and Bullet Invoices

Hybrid models allow users to choose what works best for them while maximizing revenue and flexibility for developers.

---

### Resources 

1. [Part 1 - Understanding Different Monetization Models](https://azamsharp.com/2025/12/26/storekit-subscriptions-understanding-monetization-models.html)
2. [Part 2 - Understanding Soft and Hard Paywalls](https://azamsharp.com/2025/12/27/storekit-subscriptions-soft-vs-hard-paywalls.html)
3. [Part 3 - Creating a Great Onboarding Experience](https://azamsharp.com/2025/12/28/storekit-subscriptions-onboarding-experience.html)
4. [Setting Up and Displaying Products](https://azamsharp.com/2025/12/30/storekit-subscriptions-setting-up-and-displaying-products.html)

### Conclusion 

Choosing the right monetization model is not about copying what other apps are doing. It is about understanding your users, the value your app provides, and the long term effort required to maintain and improve it. What works for a small utility app may not work for a content heavy or frequently updated product.

StoreKit gives developers a wide range of options, from one time purchases to subscriptions and hybrid approaches. Each model comes with tradeoffs, and there is no single best choice. The goal is to find a balance that feels fair to users while making it possible to sustain development over time.

In the next part of this series, we will look at soft and hard paywalls and how they influence user experience, conversion rates, and long term retention.
