
# StoreKit Subscriptions A Practical Guide
## Part 4 Setting Up and Displaying Subscription Products

### Introduction

In previous articles, I covered how different subscription models work, why onboarding plays a critical role in conversion, and how timing a paywall correctly can make a huge difference in whether users subscribe or walk away.

Now it is time to move from concepts to real implementation.

In this article, we will focus on how to properly set up subscription products in App Store Connect and how to display those products inside your iOS app using StoreKit. This is the foundation of any subscription based app. If this part is not done correctly, everything built on top of it becomes harder to reason about and harder to maintain.

---

### Setting Up Subscriptions on App Store Connect

Before writing any StoreKit code, subscriptions must be fully configured in App Store Connect. This step is easy to rush through, but taking the time to set it up correctly will save you from confusion and bugs later. Think of App Store Connect as the source of truth for everything related to pricing, duration, trials, and eligibility.

Start by going to App Store Connect and creating a new app. For this example, I named my app **My Green Place**, a gardening app, but you can name your app anything you want. Once the app is created, the next step is to set up subscriptions.

From the left menu, select **Subscriptions**. The first thing you need to create is a subscription group. I named mine **My Green Place Group**. This group will contain all subscription options that unlock the same level of access in the app.

A subscription group is how Apple organizes related auto renewing subscriptions that unlock the same level of access but differ in duration or price. A user can only be subscribed to one product in a group at a time, but they can freely switch between subscriptions in the same group, such as moving from a monthly plan to a yearly plan, without losing access. Apple treats these changes as upgrades or downgrades and manages billing automatically. Importantly, switching within the same group does not reset the subscription tenure, so after the first year you continue to receive 85 percent of the proceeds, after tax if applicable.

Once the subscription group is created, you can begin adding individual subscriptions to it. Click **Add Subscription** and provide a reference name. This name is internal and only visible to you in App Store Connect. Next, define the product identifier, subscription duration, pricing, and any optional free trial or introductory offer. These values will later be fetched directly by StoreKit and displayed inside your app, so accuracy here is important.

After saving the subscription, repeat this process for each plan you want to offer, such as monthly and yearly. When finished, your subscription products will be ready to be loaded and displayed inside your iOS app using StoreKit.

![Subscription Pricing](/images/subscriptions-pricing.png)

### Displaying Product Names and Prices 

Here is a cleaner, more polished, and more natural version that fits well with the rest of your article and keeps the tone practical and friendly.

---

After setting up your products in App Store Connect, the next step is displaying the product names and prices on your paywall screen. Fortunately, Apple provides a straightforward and well designed StoreKit API that makes this process simple and reliable.

Before writing any code, make sure your app is properly configured. Select your app target in Xcode, open **Signing and Capabilities**, and click **Add Capability**. From the list, choose **In App Purchase**. This adds the required entitlements so your app can interact with StoreKit.

In addition to this, you should add a **StoreKit configuration file** to your project. This file allows you to test subscriptions, prices, renewals, and edge cases directly in Xcode without relying on the real App Store. It is one of the most valuable tools for developing and debugging StoreKit flows and should be part of every subscription based project.

![StoreKit Configuration](/images/storekit-configuration.png)

After setting up StoreKit configuration file also make sure to select it under schemes. 

![StoreKit Configuration Scheme](/images/storekit-configuration-scheme.png)

