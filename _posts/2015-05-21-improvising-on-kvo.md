---
layout: post
title: "Improvising on KVO"
description: "Key-value Observation code can get messy.
              Here is one way to keep it clean."
category: posts

---

Brent Simmons wrote about [using KVO in a crash-proof way][inessential_kvo]. The solution he describes uses plain-old KVO instead of bindings, and custom setters instead of getters. Brent's solution is well-thought-out, simple, and would no doubt reduce the chance of subsequent programmer errors.

For the most part, my own KVO code looks like Brent's: I don't use bindings [^1] and I use Swift's `didSet` to react to changes in `self`'s properties, which is a lot like Brent's use of a custom setter.

Where my KVO code differs is in the observation part.

If I were to use Cocoa's KVO API, I'm forced to spread out the observation-related code across multiple methods:

  - Starting the observation, typically in `init`
  - Reacting to the property change, in `observeValueForKeyPath`
  - Stopping the observation, typically in `deinit` (or `dealloc` in Objective-C)

I find that having to scatter this code across multiple methods makes code harder to read, and therefore harder to maintain.

I prefer to keep the code related to observation of a particular property within a single method. To achieve this, I use a thin block-based abstraction on top of KVO, which is a little different from the other target-selector-based abstractions I have encountered, namely Daniel Eggert's [KeyValueObserver][objc_io_kvo_code] (with a [nice explanation in objc.io#7][objc_io_kvo]) and Raizlabs' [RZDataBinding][].

As is done in [KeyValueObserver][objc_io_kvo_code], I create a separate helper class to wrap the observation calls.  The helper class' `init` method calls `addObserver` and its `deinit` calls `removeObserver`. The observation is "live" as long as there exists a strong reference to the helper object. The helper class takes in a block to call when the observed property changes. For the observation code to read well, I extend `NSObject` to provide a `onChange` convenience method that creates and returns the helper object.

The end result is that all the observation code ends up
in one place. While this works equally well in both
Objective-C and Swift, I find the Swift code to be
more readable:

    // MyViewController.swift

    // Whenever the model's title changes,
    // update self's title
    _observer = modelObject.onChange("title") {
        [weak self] _ in
        self?.title = modelObject.title
    }

Here, `_observer` is an ivar in `MyViewController`, so that as long as the controller is alive, there's a strong reference to the returned object, and the observation stays alive. With this pattern, the controller doesn't even know about the helper class - the `_observer` can be of type `AnyObject` (or `id` in Objective-C).

This pattern continues to work well when we have to stop
observing one object and start observing another. For
example, my app [Bisect][] includes a tabbed browser. The browser view controller observes the URL property of the current tab, so that when the user taps a link, the displayed URL can be updated:

    // BrowserPaneViewController.swift

    // Whenever the current tab's URL changes,
    // update the displayed URL
    self._observer = currentTab.onChange("URL") {
        [weak self] _ in
        self?._omnibox.URL = currentTab.URL
    }

When the user switches to another tab, we should drop our KVO observations on the previously-in-focus tab and start observing the now-in-focus tab. This is inherently taken care of in the above code: The previous value of `self._observer` gets deallocated, which causes the observation on the previously-in-focus tab to cease.
(This works equally well when using Daniel Eggert's pattern.)

The same API can be adapted for accessing the KVO change dictionary and for listening to multiple properties in one block. Heck, I even use this API to listen to `NSNotification`s, like this:

~~~
let app = UIApplication.sharedApplication()
self._observer = app.onNotification(UIContentSizeCategoryDidChangeNotification) {
    _ in
    textView.font = UIFont.preferredFontForTextStyle(UIFontTextStyleBody)
}
~~~

While this abstraction is nice, it comes with a major caveat that you probably have already noticed: We need to do the weak-self dance whenever we access `self` in the `onChange` block. If we don't, we end up with a retain cycle like this: `self` &rarr; `_observer` &rarr; block &rarr; `self`.

But then, I find it easier to remember to do the
weak-self dance than to maintain observation-related code scattered across different parts of the file.

The source code that enables this abstraction can be found here:  
[ObservationHelper.swift][]

[inessential_kvo]: http://inessential.com/2015/05/14/how_not_to_crash_1_kvo_and_manual_bind

[kvo_feels_so_wrong]: http://www.ianthehenry.com/2014/5/4/kvo-101/

[objc_io_kvo]: http://www.objc.io/issue-7/key-value-coding-and-observing.html#observing_changes
[objc_io_kvo_code]: https://github.com/objcio/issue-7-lab-color-space-explorer/blob/master/Lab%20Color%20Space%20Explorer/KeyValueObserver.m
[objc.io#7]: http://www.objc.io/issue-7/
[RZDataBinding]: https://github.com/Raizlabs/RZDataBinding

[Bisect]: http://bisectapp.com/
[ObservationHelper.swift]: https://gist.github.com/roop/83b13cae7296e5748113

---

[^1]: The real reason why I haven't used bindings is
      because I _can't_ on iOS. I haven't done 
      any Mac stuff, and have no practical knowledge
      on bindings.

[^2]: RZDataBinding's observation API differs
      from Daniel Eggert's API in one significant
      way: instead of returning the helper object, it
      is held onto internally by making it an
      associated object of the observed object.
