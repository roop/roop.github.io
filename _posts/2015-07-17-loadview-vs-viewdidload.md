---
layout: post
title: "<code>loadView</code> vs <code>viewDidLoad</code>"
title_text: "loadView vs viewDidLoad"
description: "What is the best override point to setup your view hierarchy programatically?"
tagline: "For programatic UI setup"
category: posts
readthisnext: /posts/2015/doing-it-all-in-code
plug: all-in-code
---

I was talking to [Justin Driscoll][] on Twitter on setting up the view
hierarchy in code, and whether the best place to do that was `loadView`
or `viewDidLoad`.

[Justin Driscoll]: https://twitter.com/jdriscoll

Developers who choose to setup their view hierarchy in `loadView` do so like this:

{% highlight swift %}
override func loadView() {
    let v = UIView()
    let subview = UIView()
    v.addSubview(subview)
    positionSubviewWithEitherFramesOrConstraints()
    self.view = v
}
{% endhighlight %}

[Here](https://github.com/marcoarment/BugshotKit/blob/e982a2adadf96f371f9a6d72b4fa7efe6a99bdc1/BugshotKit/BSKScreenshotViewController.m#L108)
is one example of this strategy.

Developers who choose to setup their view hierarchy in `viewDidLoad` do so like this:

{% highlight swift %}
override func viewDidLoad() {
    let v = self.view
    let subview = UIView()
    v.addSubview(subview)
    positionSubviewWithEitherFramesOrConstraints()
}
{% endhighlight %}

[Here](https://github.com/marcoarment/BugshotKit/blob/e982a2adadf96f371f9a6d72b4fa7efe6a99bdc1/BugshotKit/BSKMainViewController.m#L51)
is an example of this strategy.

If we setup the view hierarchy in `viewDidLoad` and we want a custom
view as the root view, we should also implement a custom `loadView`:

{% highlight swift %}
override func loadView() {
    self.view = MyCustomView()
}
{% endhighlight %}

Both of these strategies work. So which one is better?

### What does Apple recommend?

Apple's documentation does talk about setting up the view hierarchy in
code without using Interface Builder.

[View Controller Programming Guide][]:

> If you prefer to create views programmatically, instead of using a
> storyboard, you do so by overriding your view controller’s `loadView`
> method. Your implementation of this method should do the following:
>
>  1. Create a root view object. ...
>
>  2. Create additional subviews and add them to the root view.
>
>     For each view, you should:
>
>      1. Create and initialize the view.
>      2. Add the view to a parent view using the `addSubview:` method.
>
>  3. If you are using auto layout, assign sufficient constraints to
>     each of the views you just created to control the position and
>     size of your views. Otherwise, ...
>
>  4. Assign the root view to the `view` property of your view controller.

[UIViewController][]:

> If you cannot define your views in a storyboard or a nib file,
> override the `loadView` method to manually instantiate a view
> hierarchy and assign it to the `view` property.
>
> ...
>
> You can override [the `loadView`] method in order to create your views
> manually.  If you choose to do so, assign the root view of your view
> hierarchy to the view property.
>
> ...
>
> If you want to perform any additional initialization of your views, do
> so in the `viewDidLoad` method.


[UIViewController]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/
[View Controller Programming Guide]: https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/ViewLoadingandUnloading/ViewLoadingandUnloading.html

So, as per Apple's documentation, the recommended place to setup the
view hierarchy is `loadView`.

Apple suggests we do "any additional initialization" of our views in
`viewDidLoad`. I interpret that as: If you'd like to setup additional
views on top of what's in your storyboard or xib, do it in
`viewDidLoad`.

While it's not entirely clear what "additional initialization" is meant
to be done in `viewDidLoad`, it's quite clear that `loadView` is the
recommended override point for setting up the view hierarchy in code.

### Practically speaking

I see a lot of code that sets up the view hierarchy in `viewDidLoad`
without setting up the views in Interface Builder. Though that doesn't
strictly follow what Apple recommends we do, that's perfectly safe,
because of the way in which a view controller loads its view.

When the `view` property of a `UIViewController` is accessed, the
pseudocode for returning a view looks something like this [^1]:

>  1. If there's already a view set, return that view
> 
>  2. Call `loadView`
> 
>     The default implementation of `loadView` does something like this
>     [^2]:
> 
>      - If there's an associated storyboard / xib,
>           - Load the view hierarchy from the storyboard / xib
>           - Assign the root view to the `view` property
>      - If there's no associated storyboard / xib
>           - Create a `UIView` instance with a default `frame` and
>             `autoResizingMask`
>           - Assign that view to the `view` property
> 
>  3. For any layout guides created from a storyboard / xib, setup Auto
>     Layout constraints between the layout guides and the view of the
>     view controller
>  
>  4. Call `viewDidLoad`

So, when you're not using a storyboard or xib, `viewDidLoad` gets called
almost immediately after `loadView`. In practice, it makes no difference
whether you setup the view hierarchy in `loadView` or `viewDidLoad`.

I personally prefer doing it in `loadView`, which is consistent with
Apple's documentation, but I have seen a lot of code that does it in
`viewDidLoad`.

To conclude, we can set up the view hierarchy in either `loadView` or
`viewDidLoad`, as long as we're aware of when which method gets called.
The most important thing to remember is that if you override `loadView`,
you should set the `view` property, while if you override `viewDidLoad`,
you should only read the `view` property, which should have been already
set.

---
[^1]: Thanks to Xcode and Hopper

[^2]: This is why, in your implementation of `loadView`, you shouldn't
      call `super.loadView()` and then also set the `view` property.
      Doing that just throws away the view that the default
      implementation created for you.
