---
layout: post
title: "Doing it all in code"
description: "I'm one of those developers who like to write down the UI
              in code. I'd like to show you how my setup looks like."
tagline: "Programmatic UI in iOS"
category: posts
---

Should you specify the UI in Interface Builder, or should you write it
down as source code? Arguments can be made for and against either
method, and you can find great iOS devs in either of the camps. At the
end of the day, it just comes down to personal or team preferences,
priorities and workflows.

Personally, I prefer to keep all my layout in code, and use Interface
Builder to a much lesser extent [^1]. 

The major disadvantage with doing the layout in code is that setting up
the Auto Layout constraints involves writing verbose and "noisy" code,
even when using the [visual format][]. To keep code readable, many
developers create a thin abstraction over Auto Layout as Objective-C
categories or Swift extensions, like [Mike Swanson][] and [Justin
Driscoll][] for example. Some even use heavy abstractions like
[Masonry][] and [PureLayout][] to address this.

I have my own thin abstraction that works well for me, and I'd like to
show you how it works and what my code looks like.

### The Auto Layout abstraction layer

My goal for the abstraction was to make the layout code look less
imperative and more declarative. I took advantage of Swift's terse
syntax for tuples and enums to get there.

For example, let's say we have this design for a browser's omnibox:

<figure>
<a 
    href="{{ site.url }}/images/doing-it-all-in-code/browser_omnibox_design.png"
    ><img 
     src="{{ site.url }}/images/doing-it-all-in-code/browser_omnibox_design.png"
/></a>
</figure>

This design needs three views:

 1. A background button that spans the whole omnibox to make the whole
    thing tappable.
 2. A label at the center to show the hostname
 3. A reload button at the right edge, maybe with some margin, centered
    vertically

The abstraction layer provides a `addSubviewsWithConstraints` method
that lets me specify this layout as:

{% highlight swift %}
// This is inside the `init` of class `Omnibox`,
// which is a subclass of `UIView`.
// So `self` here refers to the omnibox itself.

self.addSubviewsWithConstraints(
    (backgroundButton, [
            .FillIn(self)
        ]),
    (hostnameLabel, [
            .CenterIn(self)
        ]),
    (reloadButton, [
            .CenterVerticallyIn(self),
            .Anchor(.Right, self.right(margin: -10))
        ])
)
{% endhighlight %}

The `addSubviewsWithConstraints` method takes a variable number of
tuples as arguments. Each tuple is of type `(UIView, [LayoutInfo])`,
where `LayoutInfo` is an enum with possible values like `.FillIn`,
`.CenterIn` and `.Anchor`.

For every view we pass in, `addSubviewsWithConstraints` does the
following:

  1. Adds the passed view as a subview
  2. Sets the passed view's `translatesAutoresizingMaskIntoConstraints` to
     false (which we [need to always do][Adopting Auto Layout] when
     setting a view's Auto Layout constraints in code)
  3. Converts the information in `LayoutInfo` into a bunch of Auto
     Layout constraints and adds the constraints. The views and view
     controllers used in the `LayoutInfo` part should be either the
     parent or a sibling of the passed view.

[Adopting Auto Layout]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/AdoptingAutoLayout/AdoptingAutoLayout.html

The resulting layout code is readable, informative and terse.  As
always, we need to ensure that the constraints we specify are neither
conflicting, nor ambiguous.

The source code that enables this abstraction can be found here:
[AutoLayoutHelper.swift](https://gist.github.com/roop/7c91f8ebdfbdcb627c08)

This abstraction makes heavy use of Swift-only features like tuples and
enum associated values. So, unfortunately, it can't be used for writing
UI layouting code in Objective-C.

### Views

When I create a custom view with encapsulated subviews, I setup the view
hierarchy in the view's `init`.

The code looks something like this:

{% highlight swift %}
class MyView: UIView {
    private let _label: UILabel
    init() {
        // Init instance variables
        let label = UILabel()
        label.text = "Label"
        self._label = label
        // Call designated init on superclass
        super.init(frame: CGRectZero)
        // Set up view hierarchy
        self.addSubviewsWithConstraints(
            (label, [ .CenterIn(self) ])
        )
    }
}
{% endhighlight %}

A few things to keep in mind:

 1. The designated initializer for `UIView` is `init(frame:)`; we need
    to make sure we call that on `super` in our `init`.  (In Swift,
    Xcode will prompt you to add a `init(coder:)` as well, which is
    fine.)

 2. Sometimes, the view knows best what its width and/or height should
    be, and it's best to write that down in the view's code in
    `intrinsicContentSize` rather than in it's superview's Auto Layout
    constraints code.

    By default, the Auto Layout system considers the intrinsic size to
    be a minimum, and can sometimes come up with a layout where view's
    size exceeds the intrinsic size. If you don't want that to happen,
    you will need to set a high content hugging priority with a call to
    `setContentHuggingPriority` in `init`.

 3. For any subviews that need to be laid out manually, we can set their
    frame in `layoutSubviews`. If we are using Auto Layout for some
    other subviews, we need to [make sure we call][oleb1]
    `super.layoutSubviews()`, so that the Auto Layout constraints are
    applied on those subviews.

[oleb1]: http://oleb.net/blog/2014/03/how-i-learned-to-stop-worrying-and-love-auto-layout/

### View Controllers

For view controllers, I setup the view hierarchy in `loadView`.

The code looks something like this:

{% highlight swift %}
class MyViewController: UIViewController {
    override func loadView() {
        let v = UIView()
        let toolbar = UIToolbar()
        let contentView = UIView()
        v.addSubviewsWithConstraints(
            // The toolbar is at the top
            (toolbar, [
                .FillHorizontallyIn(v),
                .Anchor(.Top, self.bottomOfTopLayoutGuide()),
                ]),
            // The content view occupies the rest of the space
            (contentView, [
                .FillHorizontallyIn(v),
                .Anchor(.Top, toolbar.bottom()),
                .Anchor(.Bottom, v.bottom())
                ])
        )
        self.view = v
    }
}
{% endhighlight %}

If there are child view controllers, they should be setup along with the
call to `addSubviewsWithConstraints`, like:

{% highlight swift %}
    override func loadView() {
        let v = UIView()
        ...
        self.addChildViewController(childVC)
        v.addSubviewsWithConstraints(
            (childVC.view, [ .FillIn(v) ])
        )
        vc.didMoveToParentViewController(self)
        ...
    }
{% endhighlight %}

Some developers who setup the view hierarchy in code do so in
`viewDidLoad`. That would work as well. However, `viewDidLoad` is
provided as an override point for performing additional setup after
the Interface-Builder-specified view hierarchy is loaded. `loadView` is
the intended override point for setting up the view hierarchy purely in
code, and I like to stick to the intended purposes for these override
points.

Things to keep in mind here:

 1. In case we write an `init` in our view controller, we need to make
    sure we call the designated initializer of the superclass like this:
    `super.init(nibName: nil, bundle: nil)`. (In Swift, Xcode will
    prompt you to add a `init(coder:)` as well, which is fine.)

 2. I'm quite sure you already know this, but anyway: You should only
    set `self.view` in `loadView`; Never read `self.view`, because doing
    that will invoke `loadView` again, causing an infinite recursion.
    On the contrary, if you're setting up the view hierarchy in
    `viewDidLoad`, you might want to only read the `view` property
    and not write to it.

### Animations

Animating with Auto Layout is not simple, and that's because for the
duration of the animation, we want the constraints to temporarily be
unstatisfied.

In most cases, I can get away with animating the constraint constants.
Whenever view transforms are involved, I don't use Auto Layout for the
duration of the animation.

 1. **Simple movements**

    For simple horizontal or vertical movements, I specify that constraint
    separately, and hold on to that constraint, so that I can modify the
    constant later on.

    For example, to be able to move the `hostnameLabel` horizontally, I
    do something like:

    ~~~
    self.addSubviewsWithConstraints(
        (hostnameLabel, [
                .CenterVerticallyIn(self)
                // Left anchoring done separately
            ])
    )
    var layoutConstraint = NSLayoutConstraint(
        item: hostnameLabel, attribute: .Left, relatedBy: .Equal,
        toItem: self, attribute: .Left, multiplier: 1, constant: 0)
    self.addConstraint(layoutConstraint)
    self._leftAnchorConstraint = layoutConstraint
    ~~~

    For changing the constraint's constant as an animation, I do:

    ~~~
    self.layoutIfNeeded() // Flush pending layout operations
    UIView.animateWithDuration(0.4, delay: 0, options: .CurveEaseInOut,
        animations: {
            self._leftAnchorConstraint?.constant = updatedValue
            self.layoutIfNeeded() // Animate the constraints constant change
        }, completion: { (_: Bool) in
            // Do stuff
        }
    )
    ~~~

 2. **Animations involving transforms**

    When the animation involves view transforms, I like to switch to
    manual layout during the animation, and hook up the constraints
    back again after the animation is over.

    ~~~
    self.view.removeLayoutConstraintsForSubview(subview)
    subview.setTranslatesAutoresizingMaskIntoConstraints(true)
    self.view.layoutIfNeeded()
    subview.transform = CGAffineTransformIdentity
    subview.bounds = initialRect
    subview.center = initialCenter
    UIView.animateWithDuration(0.4, delay: 0, options: (.CurveEaseIn),
        animations: {
            subview.center = initialCenter
            subview.transform = transform
        }, completion: { (_: Bool) in
            subview.removeFromSuperview()
            // Re-add with new constraints if required.
        }
    )
    ~~~

### Other Problems

Interface Builder is also great for specifying static table views, like
for a Settings page, or for context menus. Writing that down in code
involves a lot of boilerplate verbosity.

[Static][] by Sam Soffes and others at Venmo looks great for this use
case, which I intend to try out.

### What's your take?

This is what I use, and it works well for me. What strategies and tricks
do you use when setting up the UI in code?

Comments and feedback welcome on Twitter [@roopeshchander][].

[visual format]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html
[Mike Swanson]: http://blog.mikeswanson.com/post/66133439009/jbnslayoutconstraint-helper-categories
[Justin Driscoll]: http://themainthread.com/blog/2015/02/simple-auto-layout-in-swift.html
[Masonry]: https://github.com/SnapKit/Masonry
[PureLayout]: https://github.com/smileyborg/PureLayout 
[Static]: http://blog.soff.es/static
[@roopeshchander]: http://twitter.com/roopeshchander

---
[^1]: To set the iOS app launch screen, I prefer to [use Interface
      Builder](http://oleb.net/blog/2014/08/replacing-launch-images-with-storyboards/)
      rather than specifying images.

