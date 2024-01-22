# Stop Testing Your UI Using View Models 

A few months ago, I attended iOSDevHappyHour and engaged in a discussion with a young man. He was discussing how he tested his SwiftUI interface using view models. I asked him for clarification, and he explained that instead of writing UI tests, he simply wrote tests for the view models. According to him, if the view model tests passed, then the UI was working as expected.

> For me, this was a red flag.

There's nothing inherently wrong with writing unit tests for your view models. View models can contain presentation logic, and if there's complicated logic involved, it's advisable to write tests to ensure the desired level of confidence.

However, merely passing the tests for your view models doesn't guarantee that your UI is functioning as intended. Consider the following view model unit test as an example.

![View Model Test](/images/test-vm.png)

There is nothing wrong with the unit test. It is perfectly acceptable to write unit tests for a view model. However, this test alone does not confirm that the UI is functioning as intended.

Don't take my word for it! Go to your view and delete the Button and Text views. Your test will still pass.

Once again, writing unit tests for view models is not wrong but keep in mind that those tests are not substitute for UI testing. 

How do you test your views? What methods have been effective for you? Do you use ViewInspector, UI Tests in Xcode, or do you avoid writing UI Tests altogether?

> This post is a part of my weekly newsletter where I share concise, thought-provoking articles. If you're someone interested in contemplating app architecture, testing, and design, I invite you to [subscribe to my newsletter](https://azamsharp.teachable.com/p/newsletter).

PS: I am super excited to announce that after days of hard work, I have published a brand new landing page for AzamSharp School. 

Additionally, I have activated “Free 7 Day Membership” option which will give you access to 21 courses, over 120+ hours of content.

Visit https://azamsharp.school/ for more details.