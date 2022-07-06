# Displaying App Logo in Navigation Bar for All Views in SwiftUI 

When building iOS applications, sometimes you have a requirement to put application logo in the center of the NavigationBar on all the screens. In this post, you will learn how to acheive that using ZStack. 

## Scenario

You want to create an app, where the navigation bar will always display the app logo. Your first attempt might look like the following:

``` swift 
struct ContentView: View {
    var body: some View {
        NavigationStack {
            VStack {
                Image(systemName: "heart.fill")
                    .foregroundColor(.red)
                    .font(.largeTitle)
                
                Spacer()
                NavigationLink("Go to Detail", value: "DETAIL")
                    .buttonStyle(.borderedProminent)
                
                
                Spacer()
            }.navigationDestination(for: String.self) { stringValue in
                Text("DETAIL")
            }
        }
    }
}
```

The above code will display a view with a "heart" symbol in the view as shown below. 

![Navigation Bar](/images/2022-07-06-1.png)

If you press the **Go to Detail** button, you will be taken to the view which does does not display the navigation bar. This means our heart logo will no longer be visible. Now, let's see how we can fix this issue. 

## Solution: 

We will start by creating a RootView and putting the content inside the ZStack inside the VStack. This is shown below: 

``` swift
struct RootView: View {
    var body: some View {
        ZStack(alignment: .top) {
            Image(systemName: "heart.fill")
                .foregroundColor(.red)
                .font(.largeTitle)
                .zIndex(1)
            
            NavigationStack {
                NavigationLink("Go to Detail", value: "DETAIL")
                    .buttonStyle(.borderedProminent)
                    .navigationDestination(for: String.self) { stringValue in
                        Text("DETAIL")
                    }
            }
        }
    }
}
```

Now, we have placed everything inside the ZStack and use the zIndex modifier to make sure that the image is on the top layer. The NavigationStack wraps the NavigationLink or any other view, which performs the navigation to the destination view. This means the top part remains the same but the bottom part is the one changing. 

![Navigation Bar Displayed](/images/nav-bar-icon.gif)

## Reference: 
https://stackoverflow.com/questions/72850084/how-to-show-app-logo-in-all-navigation-bar-in-swiftui-app

## Conclusion

In this post, you learned how to place an app icon on all the screens of an application. I hope you liked the post. 

<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>

