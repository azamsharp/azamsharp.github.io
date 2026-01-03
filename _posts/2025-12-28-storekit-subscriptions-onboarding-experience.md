
# StoreKit Subscriptions A Practical Guide
## Part 3 Creating a Great Onboarding Experience

### Introduction

When developers think about subscriptions, the first thing that usually comes to mind is pricing, trials, and paywalls. But long before a user ever decides to pay, they make a more important decision first.

Do I trust this app enough to spend my time here

That decision is shaped almost entirely by onboarding.

In this article, I want to share what I learned while evolving the onboarding experience for my app, My Veggie Garden, and how moving from no onboarding at all to a fully interactive onboarding flow completely changed how users engaged with the app and understood its value.

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

---

### The First Version No Onboarding at All

When I first released My Veggie Garden, there was no onboarding experience. The app launched directly into a list of vegetables.

Non subscribers could see the full list but were limited to accessing only five vegetables. My assumption was simple. Users would explore the app, hit the limit, and naturally understand why upgrading made sense.

In reality, this approach left users with very little context. They did not know why the app existed, who built it, or what problem it was trying to solve. It felt more like a utility than a product with a story.

---

### The Second Version Screenshots and an Early Paywall

After doing more research and listening to feedback, I realized the app needed onboarding screens. I added a short sequence of feature screenshots with short, enticing titles explaining what the app could do.

Right after the onboarding screens, I showed a paywall. At the same time, I allowed users to skip it and continue using the app with the five vegetable limit.

This version was better, but it still relied on screenshots and promises. Users were being told what the app could do, not actually experiencing it. There was still an emotional gap.

---

### The Real Shift Interactive Onboarding

The biggest breakthrough came when I started thinking about onboarding as an app within the app.

Instead of showing static screenshots, I let users actively use the core features during onboarding. They could create something meaningful right away. This changes everything.

When users interact with the app during onboarding, they begin to emotionally invest in it. They are no longer imagining the value. They are creating it themselves.

I also read about apps with fifty or even sixty onboarding screens. Personally, that feels excessive to me. But interestingly, many of those apps are generating tens of thousands of dollars per month. The lesson is not that everyone needs dozens of screens, but that thoughtful onboarding works when it is done with intention.

---

### The My Veggie Garden Onboarding Flow

Here is how the current onboarding flow in My Veggie Garden looks today.

---

### Screen 1 Meet the Team

This was a very intentional choice. As indie developers, we do not have a big brand name behind us. Making a personal connection matters.

This screen introduces me as the person behind the app. It tells users there is a real human who built it and cares about it.

![Meet the Team](/images/onboarding-1.png)

---

### Screen 2 Create Your First Garden

This is where interactive onboarding really begins. Users create their first garden by entering a name. Once they sign up, the garden already exists for them.

They are no longer browsing. They are participating.

![Create your first garden](/images/onboarding-2.png)

---

### Screen 3 Choose Your Vegetables

This screen allows users to add vegetables to their garden. They start making real decisions that affect their experience inside the app.

![Add Vegetables to Garden](/images/onboarding-3.png)

---

### Screen 4 Square Foot Gardening

This screen allows users to design their square foot garden using the vegetables they just added. At this point, users are fully engaged and already using one of the core features.

![Square Foot Gardening](/images/onboarding-4.png)

---

### Screen 5 Harvest

This screen shows an animated and celebratory preview of what their harvest can look like. It gives users a positive emotional payoff and reinforces why the app exists.

![Harvesting](/images/onboarding-5.png)

---

### Screen 6 Other Pro Features

This screen highlights additional pro features available in the app. By now, users understand the context and these features feel relevant.

![Other Pro Features](/images/onboarding-6.png)

---

### Screen 7 Customer Reviews

The second to last screen shows testimonials from existing users. This builds trust and reinforces that other people are finding value in the app.

![Customer Reviews](/images/onboarding-7.png)

---

### Screen 8 Paywall

Finally, the paywall appears.

At this point, the user is invested. They have created a garden, added vegetables, and explored key features. The paywall no longer feels like an interruption. It feels like a natural next step.

![Paywall](/images/onboarding-8.png)

---

### Conclusion

A great onboarding experience does more than explain features. It builds trust, creates emotional investment, and helps users understand the value of your app before you ever ask them to pay.

Screenshots and feature lists can work, but letting users experience the product for themselves is far more powerful. When users feel ownership over what they have created, subscribing feels like a continuation rather than a decision.

In the next article in this series, we will move from concepts to code and start implementing an iOS app that uses subscriptions step by step using StoreKit.


