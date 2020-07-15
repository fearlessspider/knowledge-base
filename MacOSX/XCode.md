# XCode

Xcode improvements

## Highlight	Warnings in XCode

In Swift Development, we usually use a `TODO` comment as a placeholder for future refactor. However, Xcode cannot remind us for warning or error messages like this scenario. Previously, `#warning` or `#error` comments in Objective-C environment could be highlighted by compiler so that developers will notice. Is there a way for Xcode to achieve similar effects for Swift as those in Objective-C?

The answer is yes. We could add a simple **Run Script** in **Build Phase **like this:

```
TAGS="TODO:|FIXME:|WARNING:"
ERRORTAG="ERROR:"
find "${SRCROOT}" \( -name "*.h" -or -name "*.m" -or -name "*.swift" \) -print0 | xargs -0 egrep --with-filename --line-number --only-matching "($TAGS).*\$|($ERRORTAG).*\$" | perl -p -e "s/($TAGS)/ warning: \$1/"| perl -p -e "s/($ERRORTAG)/ error: \$1/"
```

With this script, after re-building your project, all comments with “TODO:”, “FIXME:”, or “WARNING:” as prefix are treated as warnings of the compiler, and the “ERROR:”’s one is regarded as an error.

You could also custom-define this script to make different kinds of messages show up as compiler warnings or errors. In that case developers won’f forget any edge case and the run script will ensure those messages are going to be fixed in one day.

## Two Useful Environment Vars

The first one is `**DYLD_PRINT_STATISTICS**`. Adding it to the project scheme could enable developers to verify the statics of pre-main time cost, as tons of things happen before system executes your app’s `main()` function and calls app delegate functions like `applicationWillFinishLaunching`.

`Scheme -> Run (Debug) -> Arguments`

The first one is `**DYLD_PRINT_STATISTICS**`. Adding it to the project scheme could enable developers to verify the statics of pre-main time cost, as tons of things happen before system executes your app’s `main()` function and calls app delegate functions like `applicationWillFinishLaunching`

The second environment variable is `**DYLD_PRINT_LIBRARIES**`** **. Adding it to project scheme (similar as `**DYLD_PRINT_STATISTICS**`) allows us to check dynamic loader events. Specifically, it could log events when image are loaded.

For details, Apple’s Document [here](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/LoggingDynamicLoaderEvents.html) tells about these environment variables.

## A Smart Way to Manage Colour Schemes for iOS Application Development

All applications maintains a colour scheme though out every screens. This ensures the consistency over the entire application. Broadly, we can classify them under the following categories.

1. **Theme Colour.** Colours on Navigation Bar, Button Titles, Progress Indicator etc.
2. **Border Colour: **Hair line separators in between views.
3. **Shadow Colour:** Shadow colours for card like design.
4. **Dark Background Colour:** Dark background colour to group UI components with light colour.
5. **Light Background Colour:** Light background colour to group UI components with dark colour.
6. **Intermediate Background Colour: **Used for grouping UI elements with some other colour scheme.
7. **Dark Text Colour:**
8. **Light Text Colour:**
9. **Intermediate Text Colour:**
10. **Affirmation: **Colour to show success, something right for user.
11. **Negation: **Colour to show error, some danger zones for user.

More or less the list remains same for a general purpose application. As we don’t know how things will be (◼️* ***or **◻️), we need to have a dynamic behaviour to adopt to the changes quickly. That’s not an easy task, we need to expense a little bit more time while adding colours to the *UI Components *while developing the application. We need to explicitly create our custom classes for them, and assign the colour values programatically. So, whenever there a change in colour scheme, just putting the new values in *hex-string* will be enough to change the* UI Colour Theme *within a few seconds. To achieve this, we will maintaining all these stuffs in a central place. It’s time to make our hands dirty with cool code 😎.

#### The Scheme:

Now we are ready to create an *enum *to handle each and every colour required in our application.

```swift
enum Color {
    
    case theme
    case border
    case shadow
    
    case darkBackground
    case lightBackground
    case intermidiateBackground
    
    case darkText
    case lightText
    case intermidiateText
    
    case affirmation
    case negation
    // 1
    case custom(hexString: String, alpha: Double)
    // 2
    func withAlpha(_ alpha: Double) -> UIColor {
        return self.value.withAlphaComponent(CGFloat(alpha))
    }
}
```

So I have added two more things.

1. A custom* case custom(hexString: String, alpha: Double) *to get *UIColor *values other than the previous ones. *It makes me safe.*
2. *func withAlpha(_:) *enables me to add transparency to an existing colour case.

#### Lets put the values:

Now it’s time to put the values (hex string or RGB literal) to the following *extension* of *Color enum. *Using* hex string *as colour literals is my personal choice. You can try *init(red:green:blue) *too*. *The *value *is a computed property on *enum Color *and it just expresses the *UIColor *from our colour scheme *cases.*

```swift
extension Color {
    
    var value: UIColor {
        var instanceColor = UIColor.clear
        
        switch self {
        case .border:
            instanceColor = UIColor(hexString: "#333333")
        case .theme:
            instanceColor = UIColor(hexString: "#ffcc00")
        case .shadow:
            instanceColor = UIColor(hexString: "#cccccc")
        case .darkBackground:
            instanceColor = UIColor(hexString: "#999966")
        case .lightBackground:
            instanceColor = UIColor(hexString: "#cccc66")
        case .intermidiateBackground:
            instanceColor = UIColor(hexString: "#cccc99")
        case .darkText:
            instanceColor = UIColor(hexString: "#333333")
        case .intermidiateText:
            instanceColor = UIColor(hexString: "#999999")
        case .lightText:
            instanceColor = UIColor(hexString: "#cccccc")
        case .affirmation:
            instanceColor = UIColor(hexString: "#00ff66")
        case .negation:
            instanceColor = UIColor(hexString: "#ff3300")
        case .custom(let hexValue, let opacity):
            instanceColor = UIColor(hexString: hexValue).withAlphaComponent(CGFloat(opacity))
        }
        return instanceColor
    }
}
```

### How to use?

The usages is pretty simple and far more descriptive. The simplicity will make you lazy to write long colour assigning statements like the followings

```swift
let shadowColor = UIColor(red: 204.0/255.0, green: 204.0/255.0, blue: 204.0/255.0, alpha: 1.0)
let shadowColorWithAlpha = UIColor(red: 204.0/255.0, green: 204.0/255.0, blue: 204.0/255.0, alpha: 0.5)
let customColorWithAlpha = UIColor(red: 18.0/255.0, green: 62.0/255.0, blue: 221.0/255.0, alpha: 0.25)
```

**I will rather prefer to use this one.**

```swift
let shadowColor = Color.shadow.value
let shadowColorWithAlpha = Color.shadow.withAlpha(0.5)
let customColorWithAlpha = Color.custom(hexString: "#123edd", alpha: 0.25).value
```

## Swift Docs

```gem install jazzy```

```documentation:
    @jazzy \
        --min-acl internal \
        --no-hide-documentation-coverage \
        --theme fullwidth \
        --output ./docs \
        --documentation=./*.md
    @rm -rf ./build
```