+++
title = 'Vavoom!'
date = '2025-03-23T10:48:33+11:00'
tags = ['swift']
draft = true
+++

At my 'jobby job' I am fortunate enough to have made not one but two _File -> New Project_ iOS apps targeting modern iOS with Swift 6 within the last 6 months. We test and iterate fairly quickly on these tool-like applications. Risk of deployment of these apps is also low due to them being internal. In this vein I have been searching for a lightweight, fast and robust way to make these applications.

## View as View Model

### Shifting View Paradigm

I have felt moderately strongly since using SwiftUI in anger that what Apple defined as a `View` in SwiftUI does not closely align with what we used to think of as a view. To contrast we can look at the definitions from Apple's own documentation:

> UIKit - UIView
>
> An object that manages the content for a rectangular area on the screen.
> [Source](https://developer.apple.com/documentation/uikit/uiview)

> SwiftUI - View
>
> A type that represents part of your app’s user interface and provides modifiers that you use to configure views.
> [Source](https://developer.apple.com/documentation/swiftui/view)

From this we can see the movement from 'managing the content for a rectangular area' to 'representing a part of your app's user interface' more clearly.

### What's a View Model anyway?

There are countless places to read about the MVVM architecture and the role of a view model. But for the purposes of this article, let's look to Google to see what our Android friends are using a View Model for.

> App Architecture - ViewModel
>
> The `ViewModel` class is a business logic or screen level state holder. It exposes state to the UI and encapsulates related business logic. Its principal advantage is that it caches state and persists it through configuration changes. This means that your UI doesn’t have to fetch data again when navigating between activities, or following configuration changes, such as when rotating the screen.
> [Source](https://developer.android.com/topic/libraries/architecture/viewmodel)

As we can see the Android constraints here are a little different than we deal with over on the iOS side of the fence. Apple has given us the tools we need to make the `View` a 'state holder' (via `@State` and friends). These wrappers also successfully hold state through the equivalent of what Android terms a 'configuration change' e.g. rotation or colour scheme changes. There may or may not be reasons to extract your business logic from a `View` (sharing, consistency, de-duplication, etc). Through this analysis however we can see that there are less imperatives to make that code sharing entity a `ViewModel`.

### The approach

The approach to crafting a View here is a simple one:
1. Encapsulate the source of truth
2. Expose display logic as computed properties
3. Define actions as functions

The desired outcome is having Views that format their data as required, own their view related business logic and are testable.

#### Hurdles / Solutions

Unfortunately a major hurdle to this outcome is Apple's own tooling. `@State` and friends are black boxes and we have next to no control. How this limits testing is probably its own post. However leaning on some lovely work by the folks at [Point Free](https://www.pointfree.co/) we can clear these hurdles. Primarily using the new [Sharing](https://github.com/pointfreeco/swift-sharing) library.

### Implementation

Source code for this implementation can be found [here](https://github.com/adamjcampbell/vavoom/)

So what does one of these Views look like?

```swift
struct QuoteView: View {
    @State.SharedReader(value: Optional<AnimeQuote>.none) var animeQuote

    var quotee: String {
        let quotee = animeQuote?.character ?? "Example Name"
        return "- \(quotee)"
    }
}
```

Above we have a snippet, part of the `QuoteView` that uses a `SharedReader` (wrapped in SwifUI State for nice attachement and detachement behaviour native to SwiftUI). The `SharedReader` `animeQuote` is our goal listed in the approach as `1`. An encapsulation of our state, it manages the ability to store, load and subscribe to updates of state values. Here we just initialise it with a nil quote.

Next we have a computed variable `quotee` that will either contain the character from the anime quote or some dummy text. 

