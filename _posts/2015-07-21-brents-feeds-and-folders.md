---
layout: post
title: "Brent's feeds and folders problem"
description: "Here's one way of solving a problem Brent Simmons had when
              adopting protocol-oriented programming with Swift 2.0"
category: posts
---

[Brent Simmons described][] a problem he faced when adopting
protocol-oriented programming with Swift 2.0. (To follow along, please
read his post in full before continuing with this post.)

[Brent Simmons described]: http://inessential.com/2015/07/19/secret_projects_diary_2_swift_2_0_prot

The crux of the problem is that equality between two values of a custom
type is not implicit in Swift. So, with a custom type `Feed`, it's not
possible out of the box to check if an array of feeds contains a
particular feed - we first need to tell Swift what it means to say two
feeds are the same.

The quick Objective-C version that Brent wrote does have a similar
problem as well, though it's less obvious.

### The Objective-C version

In the Objective-C version of the project, I'm assuming both `Feed` and
`Folder` would be defined as Objective-C protocols, and we'd have
`LocalFeed` and `LocalFolder` as `NSObject`-based Objective-C classes
conforming to those protocols respectively.

Brent's Objective-C version of `addFeeds` (in say `LocalFolder`)
looks like this:

{% highlight objective-c %}
- (void)addFeeds:(NSArray *)feedsToAdd {
  NSMutableArray *feedsArray = [self.feeds mutableCopy];
  for (id<Feed> oneFeed in feedsToAdd) {
    if (![feedsArray containsObject:oneFeed]) {
      [feedsArray addObject:oneFeed];
    }
  }
  self.feeds = [feedsArray copy];
}
{% endhighlight %}

`self.feeds` contains the feeds that are already in the local folder.
`feedsToAdd` contains the feeds that we'd like to add to the folder.
It's likely that the objects in `feedsToAdd` were created at a different
point in time than the objects already in `self.feeds`.  The default
equality checking in `NSObject` is to check for the equality of
pointers, so `addFeeds` wouldn't work correctly if the feed urls are the
same, but the object addresses are different.

One way to fix this problem is to override `isEqual:` for `LocalFeed`
to check the equality of the feed urls rather than memory addresses.
This would override the implicit equality checking with the equality
checking that we really want.

(**Update 22/Jul/2015:** [Brent clarified][brent's twitter reply] that pointer
equality was really what he wanted.)

Also, I think the concept of a feed is more like a struct than a class.
But there's no way for a value type to conform to a protocol in
Objective-C, so we are forced to make `LocalFeed` a class here.

### Let's Swift

This is how I would model this scenario in Swift.

A feed is anything that has `url` as a gettable property.

{% highlight swift %}
protocol Feed {
    var url: String { get }
}
{% endhighlight %}

`LocalFeed` is a value type conforming to the `Feed` protocol.

{% highlight swift %}
struct LocalFeed: Feed {
    var url: String
    init(url: String) {
        self.url = url
    }
}
{% endhighlight %}

We need to be able to check if a feed exists in an array, so we should
define what equality for feeds means. We do that by making the `Feed`
protocol inherit from the `Equatable` protocol.

The `Equatable` protocol requires that we implement a function with
signature `func ==(lhs: Self, rhs: Self) -> Bool`. The `Self` here
refers to the actual type conforming to the protocol (like `LocalFeed`)
that we're checking for equality. So, a function signature of `func
==(lhs: Feed, rhs: Feed) -> Bool` will not help us conform to
`Equatable`, because `Feed` is a protocol, not a type.

We should write a function that can take two values of a particular
type, where that type conforms to the `Feed` protocol. We can do that
using generics.

{% highlight swift %}
protocol Feed: Equatable {
    var url: String { get }
}

func ==<T: Feed>(lhs: T, rhs: T) -> Bool {
    return (lhs.url == rhs.url)
}
{% endhighlight %}

A folder contains a bunch of feeds.  We can have different types of
folders, and they would contain the corresponding types of feeds. For
example, a LocalFolder would contain a bunch of LocalFeeds, and a
FeedlyFolder would contain a bunch of FeedlyFeeds. It's not possible for
a FeedlyFolder to contain a FeedBinFeed.

To represent a bunch of feeds in a folder, let's use an array.
To represent an array of feeds, we might write `[Feed]`. But if we
really think about it, that's not correct because `Feed` is not a type -
it's a protocol. What we want is an array of values of a certain _type_,
where that type is a type that conforms to the `Feed` protocol.

We need a `typealias` to represent this. When we say `typealias
FeedType: Feed`, we create a placeholder type called `FeedType` that
conforms to the `Feed` protocol. We can then proceed to use `FeedType`
in further declarations in our definition of a folder.

{% highlight swift %}
protocol Folder {
    typealias FeedType: Feed
    var feeds: [FeedType] { get }
    func addFeeds(feedsToAdd: [FeedType])
}
{% endhighlight %}

Note that the type of the `feeds` property and the type of `feedsToAdd`
in the signature of `addFeeds` are now tied to each other. If you create
a type that conforms to the `Folder` protocol, these types must be the
_same types_.  They cannot be two different types, even if both of them
conform to the `Feed` protocol.

So, when we create a `LocalFolder` we need to use a specific type
conforming to the `Feed` protocol, rather than just saying `Feed`.

{% highlight swift %}
class LocalFolder: Folder {
    var feeds: [LocalFeed] = []
    func addFeeds(feedsToAdd: [LocalFeed]) {
        for oneFeed in feedsToAdd {
            if !feeds.contains(oneFeed) {
                feeds += [oneFeed]
            }
        }
    }
}
{% endhighlight %}

We can use it like this:

{% highlight swift %}
let folder = LocalFolder()
folder.addFeeds([ LocalFeed(url: "1"), LocalFeed(url: "2") ])
folder.addFeeds([ LocalFeed(url: "2"), LocalFeed(url: "3") ])
print(folder.feeds.count) // 3
{% endhighlight %}

You can see the complete code
[here](https://gist.github.com/roop/5bb4713110093e96adb1). 

Compared to Brent's original version, the only significant additional
code is the `==` function. But I would argue that likewise, we needed an
`isEqual:` in Objective-C to make it work correctly.

I'd love to hear your feedback on this, especially Brent's. You can find
me on Twitter [@roopeshchander](http://twitter.com/roopeshchander).

(**Update 22/Jul/2015:** Brent's [feedback tweet][brent's twitter reply].)

[brent's twitter reply]: https://twitter.com/brentsimmons/status/623510343344455680
