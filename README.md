# Refresher

A customizable, native Swift UI refresh control for iOS 14+

*NOTE: iOS 14 has some jankyness. The `.overlay` or `.system` style works best for iOS 14.* 

## Why?

- the native SwiftUI refresh control only works on iOS 15+
- the native UIKit refresh control works with ugly wrappers, but has buggy behavior with navigation views
- I needed a refresh control that could accomodate an overlay (such as appearing on top of a static image)
- This one is very customizable

## Usage 
First add the package to your project. 

```swift
import Refresher 

struct DetailsView: View {
    @State var refreshed = 0

    var body: some View {
        ScrollView {
            Text("Details!")
            Text("Refreshed: \(refreshed)")
        }
        .refresher { done in // Called when pulled to refresh
            DispatchQueue.main.asyncAfter(deadline: .now() + .seconds(1)) {
                refreshed += 1
                done() // Stops the refresh view (can be called on a background thread)
            }
        }
    }
}

```

See: [Examples](/Examples/) for a full sample project with multiple implementations

### Navigation view

![Navigation](/images/1.gif)

`Refresher` plays nice with both Navigation views and navigation subviews. 

![Subview](/images/3.gif)

### Detail view with overlay

`Refresher` supports an overlay mode to show a refresh indicator over fixed position content

`.refresher(overlay: true)`

![Overlay](/images/2.gif)

### System style
`Refresher`'s default animation is designed to be more flexible that the system animation style. If you want `Refresher` to behave more like they system refresh control, you can change the style:

```swift
.refresher(style: .system) { done in
```

![System](/images/5.gif)

## Customization

Refresher can take a custom spinner view. Your custom view will get a binding instances of the refresher state that contains useful properties for managing animations and translations. Here is a custom spinner that shows an emoji:

```swift
public struct EmojiRefreshView: View {
    @Binding var state: RefresherState
    @State private var angle: Double = 0.0
    @State private var isAnimating = false
    
    var foreverAnimation: Animation {
        Animation.linear(duration: 1.0)
            .repeatForever(autoreverses: false)
    }
    
    public var body: some View {
        VStack {
            switch state.mode {
            case .notRefreshing:
                Text("🤪")
                    .onAppear {
                        isAnimating = false
                    }
            case .pulling:
                Text("😯")
                    .rotationEffect(.degrees(360 * state.dragPosition))
            case .refreshing:
                Text("😂")
                    .rotationEffect(.degrees(self.isAnimating ? 360.0 : 0.0))
                        .onAppear {
                            withAnimation(foreverAnimation) {
                                isAnimating = true
                            }
                    }
            }
        }
        .scaleEffect(2)
    }
}
```

Add the custom refresherView:
```swift
.refresher(refreshView: EmojiRefreshView.init ) { done in
```

![Custom](/images/4.gif)

## Want to help?

### Here are some TODO items

- Support `ListView`
- don't trigger refresh until drag is released (how do you detect `touchDown` in a scrollview?)
- expose the background padding view for customization