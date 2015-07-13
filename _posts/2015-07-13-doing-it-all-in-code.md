---
layout: post
title: "Doing it all in code"
description: "I'm one of those developers who like to write down the UI
              in code. I'd like to show you how my setup looks like."
tagline: "Programmatic UI in iOS"
category: posts
plug: all-in-code
---

I'm one of those developers who likes to specify the layout of an app's
UI in source code. I use Interface Builder as well, but to a much lesser
extent.  I think arguments can be made for and against both methods, and
it really comes down to personal preferences, priorities and workflows.
End of the day, doing it all in code is what _I_ am most comfortable with.

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
imperative and more declarative. With Swift's terse syntax for enums and
tuples, we can express the intended layout with very little code.

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
always, we need to ensure that the constraints we specify are not
ambiguous or conflicting.

The source code that enables this abstraction can be found here:
[AutoLayoutHelper.swift](https://gist.github.com/roop/7c91f8ebdfbdcb627c08).
This code is still Swift 1.2, but it can be ported to Swift 2 with
trivial changes.

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
        label.text = "Hello World!"
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
    to make sure we call that on `super` in our `init`. (The Swift
    compiler will ask you override `init(coder:)` as well - just take
    Xcode's Fix-it suggestion to resolve the error.)

 2. Sometimes, the view knows best what its width and/or height should
    be, and it's best to write that down in the view's code in
    `intrinsicContentSize` rather than in it's superview's Auto Layout
    constraints code.

    The Auto Layout system can sometimes come up with a layout where view's
    size exceeds the intrinsic size, where it treats the intrinsic size
    as a minimum (rather than the exact) size. If you encounter this,
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
        self.view = v
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
    `super.init(nibName: nil, bundle: nil)`. (The Swift compiler
    will ask you override `init(coder:)` as well - just take Xcode's
    Fix-it suggestion to resolve the error.)

 2. If we have to access a layout guide when setting up the layout, we
    should populate `self.view` before we can do that. That's why we
    have a `self.view = v` at the start of `init` here.

 2. This is obvious, but sometimes overlooked: In `loadView`, you should
    never read the `view` property before assigning to it because doing
    that will just call `loadView` again. All you get is an infinite
    recursion.

    On the contrary, if you're setting up the view hierarchy in
    `viewDidLoad`, you might want to only read the `view` property and
    not write to it.

### Animations

Animations with Auto Layout need special attention because though we've
constrained the layout of the views, we would generally want those
constraints to be **not** upheld accurately for the duration of the
animation.

 1. **Simple movements**

    For simple horizontal or vertical movements, that constraint can be
    specified separately (i.e. not within an
    `addSubviewsWithConstraints`), so that its constant can be modified
    later on.

    For example, to be able to move the `hostnameLabel` horizontally, I
    can do something like:

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

    and later, modify the constant in an animation block, like:

    ~~~
    self.layoutIfNeeded() // Flush pending layout operations
    UIView.animateWithDuration(0.4,
        animations: {
            self._leftAnchorConstraint?.constant = updatedValue
            self.layoutIfNeeded()
        }
    )
    ~~~

 2. **Animations involving transforms**

    When the animation involves view transforms, I like to switch
    completely to manual layout during the animation, and hook up the
    Auto Layout constraints, if applicable, after the animation is over.

    If the view is created and animated into a view hierarchy, I create
    it as manually placed without any constraints, do the animation, and
    setup Auto Layout constraints in the animation's completion block.

    The abstraction layer provides an `addLayoutConstraintsForSubview`
    method for adding constraints for a previously added subview.

    ~~~
    // Animating the appearance of `newlyCreatedView`
    let subview: UIView = newlyCreatedView
    subview.transform = initialTransform
    subview.center = initialCenter
    UIView.animateWithDuration(0.4, delay: 0, options: .CurveEaseIn,
        animations: {
            subview.transform = finalTransform
            subview.center = finalCenter
        }, completion: {
            subview.translatesAutoresizingMaskIntoConstraints = false
            self.view.addLayoutConstraintsForSubview(
                subview, [ .FillIn(self.view) ]
            )
        }
    )
    ~~~

    If the view is animating out of a view hierarchy and is to be
    removed from the screen, I remove all constraints and switch to
    manual layout before the animation starts.

    The abstraction layer provides a `removeLayoutConstraintsForSubview`
    method to help in switching to manual layout without removing the
    subview from the view hierarchy.

    ~~~
    // Animating the disappearance of `subview`
    self.view.removeLayoutConstraintsForSubview(subview)
    subview.translatesAutoresizingMaskIntoConstraints = true
    self.view.layoutIfNeeded()
    subview.transform = initialTransform
    subview.center = initialCenter
    UIView.animateWithDuration(0.4, delay: 0, options: .CurveEaseIn,
        animations: {
            subview.transform = finalTransform
            subview.center = finalCenter
        }, completion: {
            subview.removeFromSuperview()
        }
    )
    ~~~

    If, instead of disappearing, the view should move to a new view
    hierarchy, we can use either `addSubviewsWithConstraints` or
    `addLayoutConstraintsForSubview` in the completion block to setup
    the view hierarchy after the animation is over.

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

