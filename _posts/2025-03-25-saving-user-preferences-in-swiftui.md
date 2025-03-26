 <style>
    .share-container {
      display: flex;
      gap: 10px; /* Spacing between buttons */
      margin-bottom: 20px; 
    }
    .share-button {
      background-color: #0077b5; /* Default LinkedIn blue */
      color: white;
      border: none;
      padding: 10px 15px;
      font-size: 14px;
      border-radius: 5px;
      text-decoration: none;
      cursor: pointer;
      text-align: center;
    }
    .twitter { background-color: #1da1f2; }
    .linkedin { background-color: #0077b5; }
    .bluesky { background-color: #353c63; }
    .share-button:hover {
      opacity: 0.8;
    }
  </style>

#  Saving User Preferences in SwiftUI Using @AppStorage, Codable, and RawRepresentable 

<div class="share-container">
    <div>
      <!-- Twitter -->
      <a href="https://twitter.com/intent/tweet?url=https://azamsharp.com/2025/03/25/saving-user-preferences-in-swiftui.html&text=Saving User Preferences in SwiftUI Using @AppStorage, Codable, and RawRepresentable by @azamsharp"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button twitter">
        Share on Twitter
      </a>
    </div>
    <div>
      <!-- LinkedIn -->
      <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://azamsharp.com/2025/03/25/saving-user-preferences-in-swiftui.html"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button linkedin">
        Share on LinkedIn
      </a>
    </div>
    
  </div>

When building apps, one of the most important features you can add is **user personalization**. Whether it’s remembering if the user prefers dark mode, or if they measure distance in miles or kilometers, these little touches make your app feel thoughtful and tailored.

With SwiftUI, storing simple preferences is easy using `@AppStorage`. But what if your preferences aren’t simple? What if you want to store a **custom struct** that wraps multiple preferences?

Let’s walk through how to make that happen — using `@AppStorage` and some Swift protocol magic like `Codable` and `RawRepresentable`.

---

## 🔧 The Problem: `@AppStorage` Only Supports Basic Types

The `@AppStorage` property wrapper in SwiftUI is a powerful tool for binding your views directly to `UserDefaults`. It works great for basic data types like `String`, `Int`, `Double`, `Bool`, and a few others.

```swift
@AppStorage("isDarkMode") var isDarkMode = false
```

This is super useful when storing a single value. But if you want to group related preferences together — say, dark mode **and** measurement unit — you need to get creative.

---

## 🧱 Step 1: Creating a Settings Model

We’ll start by defining a struct to hold the user's settings. This will allow us to encapsulate everything in one place.

```swift
struct UserSettings: Codable {
    var isDarkMode: Bool
    var unit: MeasurementUnit

    init(isDarkMode: Bool = false, unit: MeasurementUnit = .miles) {
        self.isDarkMode = isDarkMode
        self.unit = unit
    }

    enum CodingKeys: CodingKey {
        case isDarkMode
        case measurementUnit
    }
}
```

### ✅ Why `Codable`?

The `Codable` protocol lets us encode and decode the struct to and from JSON. This is the secret to storing complex data types in `UserDefaults` — we serialize them into a format that can be saved as a `String`.

---

## 📏 Step 2: Adding an Enum for Measurement Units

We want to let users choose between miles and kilometers. Here’s a simple enum to represent those options:

```swift
enum MeasurementUnit: String, Codable, CaseIterable, Identifiable {
    case miles
    case km

    var id: Self { self }

    var title: String {
        switch self {
        case .miles: return "Miles"
        case .km: return "Kilometers"
        }
    }
}
```

### 🔍 Let’s Break Down the Conformances

- `String`: The enum's raw value will be a string like `"miles"` or `"km"` — perfect for JSON.
- `Codable`: Allows encoding/decoding via JSON.
- `CaseIterable`: Makes it easy to loop through all possible cases (great for a `Picker`).
- `Identifiable`: Required for SwiftUI’s `ForEach`.

---

## 🔁 Step 3: Making `UserSettings` Work with `@AppStorage`

Here’s the key part: `@AppStorage` requires the property type to be one of the primitive `UserDefaults`-friendly types. Since `UserSettings` is a custom struct, we need to teach Swift how to convert it **to and from a `String`**.

That’s where `RawRepresentable` comes in.

```swift
extension UserSettings: RawRepresentable {
    public init?(rawValue: String) {
        guard let data = rawValue.data(using: .utf8),
              let decoded = try? JSONDecoder().decode(UserSettings.self, from: data)
        else {
            return nil
        }
        self = decoded
    }

    public var rawValue: String {
        guard let data = try? JSONEncoder().encode(self),
              let result = String(data: data, encoding: .utf8)
        else {
            return "{}"
        }
        return result
    }
}
```

### 🔐 What’s Happening Here?

- `rawValue` turns the struct into a JSON string.
- `init?(rawValue:)` decodes the JSON string back into the struct.

This allows SwiftUI’s `@AppStorage("userSettings")` to store our entire settings struct as a single JSON string in `UserDefaults`.

---

## 🎨 Step 4: Building the UI

Let’s build a simple `ContentView` that allows users to:

- Toggle dark mode
- Choose between miles and kilometers
- Reset preferences to default

```swift
struct ContentView: View {
    @AppStorage("userSettings") private var userSettings = UserSettings()

    var body: some View {
        VStack {
            Text("Dark Mode: \(userSettings.isDarkMode ? "Enabled" : "Disabled")")
                .font(.title)

            Toggle("Enable Dark Mode", isOn: $userSettings.isDarkMode)
                .padding()

            Picker("Select measurement", selection: $userSettings.unit) {
                ForEach(MeasurementUnit.allCases) { unit in
                    Text(unit.title)
                }
            }
            .pickerStyle(.segmented)

            Button("Reset Settings") {
                userSettings = UserSettings() // Back to defaults
            }
            .padding()
        }
        .padding()
    }
}
```

### 🧠 What Makes This Cool?

- We’re binding directly to the properties of a **custom struct**, and everything still works!
- As soon as the user toggles something, it’s saved automatically in `UserDefaults`.
- The reset button just replaces the struct with its default initializer.

---

## 🔁 Bonus: Codable Conformance for Custom Keys

Earlier we defined custom coding keys:

```swift
enum CodingKeys: CodingKey {
    case isDarkMode
    case measurementUnit
}
```

This allows us to rename keys in the JSON representation without changing property names in the struct. For example, if we wanted to rename `unit` to `measurementUnit`, this approach gives us full control.

We also manually added the `encode(to:)` and `init(from:)` methods to be explicit:

```swift
extension UserSettings {
    func encode(to encoder: any Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(isDarkMode, forKey: .isDarkMode)
        try container.encode(unit, forKey: .measurementUnit)
    }

    init(from decoder: any Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        isDarkMode = try container.decode(Bool.self, forKey: .isDarkMode)
        unit = try container.decode(MeasurementUnit.self, forKey: .measurementUnit)
    }
}
```

This step is optional — the compiler can synthesize it — but it gives you more control and clarity when debugging.

---

## ✅ Final Thoughts

This pattern gives you a clean and scalable way to store structured user preferences in SwiftUI:

✅ One line to bind your settings using `@AppStorage`  
✅ Codable for safe and consistent serialization  
✅ Easily extendable — just add more fields to `UserSettings`  
✅ Works great with `Toggle`, `Picker`, or any SwiftUI input

You can use this approach for:
- App theme preferences
- Display settings (e.g. font size, language)
- Notification toggles
- Custom onboarding progress
- Any collection of settings you want to group and persist

---

### 🚀 Ready to Go Further?

Try adding more preferences to the `UserSettings` struct, like a username, preferred language, or notification settings. Just remember to make them `Codable`, and your persistence layer will continue working without a hitch.

