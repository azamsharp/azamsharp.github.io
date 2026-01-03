
# StoreKit Subscriptions: A Practical Guide
## Part 2: Soft vs Hard Paywalls

Choosing the right paywall strategy is one of the most important decisions you will make when monetizing an iOS app. It influences not just revenue, but also user trust, retention, and the long-term sustainability of your product. A poorly chosen paywall can push users away before they ever see value, while a thoughtful one can feel like a natural part of the experience.

In this article, we will walk through the most common paywall approaches and how they fit together in real apps.

<!-- AzamSharp Course Ad -->
<div class="az-ad">
  <div class="az-ad__body">
    <div class="az-ad__content">
      <div class="az-ad__kicker">AzamSharp Course</div>
      <div class="az-ad__title">StoreKit 2 for In-App Purchases & Subscriptions</div>
      <div class="az-ad__text">
        Learn products, paywalls, purchase flow, entitlements, and restore access. Practical and production-focused.
      </div>
    </div>
    <div class="az-ad__actions">
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
</div>

<style>
  .az-ad{
    font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
    max-width: 820px;
    border-radius: 18px;
    border: 1px solid rgba(0,0,0,.08);
    background:
      radial-gradient(900px 300px at 15% 20%, rgba(52,211,153,.22), transparent 60%),
      radial-gradient(800px 260px at 85% 10%, rgba(96,165,250,.20), transparent 55%),
      #ffffff;
    box-shadow: 0 14px 30px rgba(0,0,0,.10);
    overflow: hidden;
  }

  .az-ad__body{
    display:flex;
    align-items:center;
    justify-content:space-between;
    gap: 18px;
    padding: 18px 18px;
  }

  .az-ad__content{
    min-width: 0;
  }

  .az-ad__kicker{
    font-size: 12px;
    font-weight: 700;
    color: rgba(0,0,0,.55);
    margin-bottom: 6px;
  }

  .az-ad__title{
    font-size: 18px;
    font-weight: 850;
    letter-spacing: -0.2px;
    color: rgba(0,0,0,.88);
    margin-bottom: 6px;
    line-height: 1.2;
  }

  .az-ad__text{
    font-size: 14px;
    color: rgba(0,0,0,.65);
    line-height: 1.45;
    max-width: 56ch;
  }

  .az-ad__actions{
    display:flex;
    flex-direction:column;
    align-items:flex-end;
    gap: 8px;
    flex: 0 0 auto;
  }

  .az-ad__button{
    display:inline-flex;
    align-items:center;
    justify-content:center;
    padding: 11px 16px;
    border-radius: 12px;
    font-weight: 850;
    font-size: 14px;
    text-decoration:none;
    color: rgba(0,0,0,.88);
    background: linear-gradient(90deg, #34d399, #60a5fa);
    border: 1px solid rgba(0,0,0,.08);
    box-shadow: 0 8px 18px rgba(0,0,0,.10);
    transition: transform 160ms ease, filter 160ms ease;
    white-space: nowrap;
  }

  .az-ad__button:hover{ transform: translateY(-1px); filter: brightness(1.03); }
  .az-ad__button:active{ transform: translateY(0); }

  .az-ad__note{
    font-size: 12px;
    color: rgba(0,0,0,.50);
  }

  /* Mobile */
  @media (max-width: 640px){
    .az-ad__body{
      flex-direction: column;
      align-items: flex-start;
    }
    .az-ad__actions{
      align-items:flex-start;
    }
    .az-ad__text{
      max-width: 100%;
    }
  }
</style>

## Soft Paywall

When I launched my iOS app **My Veggie Garden**, I chose a soft paywall. This allowed users to use the entire app without paying upfront. They could create multiple gardens, enable iCloud sync, receive notifications, and use all of the core functionality.

The only limitation was the vegetable catalog. Free users could add up to five vegetables, while Pro users unlocked the complete catalog.

This approach works well because users get real value before being asked to pay. They can see how the app fits into their daily life and decide whether upgrading makes sense based on actual usage rather than marketing promises.

---

## Hard Paywall

A hard paywall takes the opposite approach. Users cannot access any part of the app unless they pay first. There are no free features to explore and no trial period to ease them in.

This model can work well for niche or professional apps where users already know exactly what they need and are actively searching for a solution. However, for most consumer apps, it creates significant friction. Users are being asked to commit before they understand the value, which often leads to early drop off.

---

## Subscription With a Free Trial

Between these two approaches is a third option that often works best. In this model, users are required to start a subscription, but they receive full access to the app for a limited time, such as three or seven days.

After the trial period ends, the subscription automatically renews unless the user cancels.

This approach gives users complete freedom to explore the app without feature limits, while still setting clear expectations from the beginning. There is no confusion about what is free and what is paid. Users understand they are evaluating a paid product during the trial.

After experimenting with different models, I eventually updated **My Veggie Garden** to use a subscription with a free trial. This change simplified the experience and made the value of the app much clearer. Instead of managing feature limits, users could focus entirely on whether the app helped them plan and manage their gardens.

For most apps, this strikes the best balance. Users can decide during the trial whether the app truly adds value to their lives. If it does, continuing feels natural. If it does not, canceling is easy and transparent. Once the trial ends, the paywall becomes permanent, which helps keep the app sustainable in the long term.

---

## Best Practices for Presenting a Paywall

How you present a paywall matters just as much as the model you choose. A good paywall is clear, honest, and respectful of the user. It explains the value of the app in simple language and focuses on benefits rather than features.

Users should immediately understand what they get by subscribing, when they will be charged, and how they can cancel. Avoid surprises. Always show the billing date and clearly state that the subscription can be canceled at any time.

Timing is just as important. A paywall shown too early can feel aggressive, while one shown too late can feel confusing. When done well, the paywall feels less like a barrier and more like a natural step forward.

---

## Showing the Paywall After Onboarding

One of the most effective strategies is showing the paywall after onboarding. Onboarding should introduce the core problem the app solves, set expectations, and help the user achieve a small but meaningful win.

By the time onboarding is complete, the user understands what the app does and why it matters. When the paywall appears at that moment, it feels earned rather than intrusive.

The transition should be smooth and intentional. The paywall should reinforce what the user just learned, remind them of the benefits they are about to unlock, and clearly explain pricing, trials, and cancellation. When users understand the value first, they are far more confident in subscribing and far less likely to abandon the app.

---

## Conclusion

There is no single paywall strategy that works for every app, but the best decisions are always made with the user in mind. Soft paywalls lower the barrier to entry, hard paywalls favor high intent users, and subscriptions with a free trial often strike the best balance between the two.

What matters most is clarity, timing, and trust. Users should understand the value of the app before being asked to pay, know exactly what they are signing up for, and feel confident that they can cancel if the app is not right for them. When a paywall is thoughtfully designed and placed after onboarding, it stops feeling like an obstacle and starts feeling like a natural step forward.

In the next article, we will take a closer look at onboarding and how to design an experience that prepares users for the paywall while building trust and setting clear expectations from the very beginning.