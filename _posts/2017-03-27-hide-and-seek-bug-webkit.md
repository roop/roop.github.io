---
layout: post
title: "The Hide &amp; Seek bug that led me to WebKit"
description: "While trying to debug a problem with Hide &amp; Seek, I ended up
fixing a bug in WebKit."
tagline: ""
category: posts

---

[Hide & Seek](/hideandseek/) is a Safari content blocker I made a couple
of years back.  The goal was to enable someone to use Google search as
an anonymous user, while being able to use other Google web services like
Gmail as a logged in user (and similar stuff for Bing and Microsoft web
services). To do this, the content blocker specifies rules that blocks
cookies only for certain URLs used in web search, preventing Google (or
Bing) from identifying the user performing the search.

While developing the Hide & Seek iOS app, I noticed a problem: When I
tapped on a Google search result, Google knew that I followed that
particular link. My [Google search history](http://history.google.com/)
showed my activity of following that search result, even though it
didn't show the search activity.

### The debug

To debug this, I used [Charles](https://www.charlesproxy.com/) to log
all requests made by iOS Safari when I tap on a link in iOS Simulator. I
could see that Safari was notifying a Google URL whenever I follow a
search result. So I tried to block delivery of cookies to that URL by
adding an appropriate rule to my content blocker. Amazingly, that had no
effect. That is, my content blocker had a rule telling Safari to block
cookies on a particular URL, but Safari was ignoring that rule and
sending the cookies along anyway to that URL. Even if I modify the
content blocker rule to block requests to that URL completely, that rule
too was being ignored.

I noticed that the Google URL, to which the cookie-ful request was being
sent, appeared as the value of a `ping` attribute in the search result
link's anchor (`a`) HTML element. That led me to dig up what `ping` was.

I learnt that the `ping` attribute exists as a faster way to track clicking of
links. Normally, Google tracks clicking of search results with a redirect. In
desktop browsers, a Google search result link looks something like this
in HTML:

~~~html
<a href="https://www.google.com/url?url=http://example.com/search-result/">
Example search result
</a>
~~~

So when the user clicks on the search result link, the user is initially
sent to a Google-hosted URL, with the actual search result link passed
on as a query parameter. Google logs this activity, then redirects to
the actual search result.

For iOS Safari, Google [switched to][ilya] using the `ping` attribute
for performing this tracking, so the HTML code for Google's search
result link for iOS looks something like this now:

[ilya]: https://plus.google.com/+IlyaGrigorik/posts/fPJNzUf76Nx

~~~html
<a href="http://example.com/search-result/"
   ping="https://www.google.com/url?url=http://example.com/search-result/">
Example search result
</a>
~~~

When the user clicks on this link, the browser goes directly to the
search result page, and _in parallel_, sends a POST request to the URL
specified in the `ping` attribute. Google logs the activity as before,
but it doesn't have any redirecting to do now.

This is obviously faster than having to follow a Google-hosted URL
first, and then be redirected to the actual search result. So, it nice
of iOS Safari to support it and of Google to make use of it.

The only problem was that my content blocking rules were being ignored
for these `ping`-induced POST requests. There wasn't anything I could do
to fix this in my content blocker code, so I filed a radar with Apple,
and then shipped my app anyway with this limitation, asking users of my
app to use the _Open in New Tab_ option in iOS Safari rather than
following search results directly.

### The fix

The content blocking capability in Safari comes from WebKit, which is
open source. That meant I could browse the code and see why content
blocking wasn't doing what I thought it should.

I had observed that all the `<a ping>`-related POST requests had their
content type set as `text/ping`, and a search in the WebKit code for
that content type led me to `PingLoader::sendPing` in
[PingLoader.cpp](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/loader/PingLoader.cpp?rev=186663).
The code for the content blocking machinery was at
[WebCore/contentextensions](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/contentextensions?rev=186663),
(the content blocking machinery is always referred to as "content
extensions" in the WebKit codebase). I could see that when loading a page,
the content blocking machinery was consulted, by a call to
`processContentExtensionRulesForLoad`, in
[FrameLoader.cpp](https://trac.webkit.org/browser/webkit/trunk/Source/WebCore/loader/FrameLoader.cpp?rev=186663).
However, `PingLoader::sendPing` had no calls related to content
blocking. It looked like an oversight that content blocking rules were
not being consulted when sending the `<a ping>` pings. I prepared a
testcase as a macOS Safari extension and [filed a
bug](https://bugs.webkit.org/show_bug.cgi?id=149873) on WebKit.

It even seemed possible that I could attempt to fix it, so I was tempted
to try my hand at it. I tried making a call to
`processContentExtensionRulesForLoad` in `PingLoader::sendPing`, and I
could see that in WebKit's single-process test browser (called
MiniBrowser), cookies were correctly blocked for `<a ping>` pings. I
added tests (which took longer than I thought) and submitted that as a
patch to fix the issue.

I tried to see if that would indeed fix the original problem I started
with -- of Google getting to know who followed their search result -- by
trying to use my fix from iOS Simulator, but I couldn't get that
working. I wrote to Benjamin Poulain, the author of the WebKit [blog post
on content blockers][webkitblogpost] on this, who assured me that
testing it on the MiniBrowser would suffice because the content blocking
code was identical on iOS and macOS.

[webkitblogpost]: https://www.webkit.org/blog/3476/content-blockers-first-look/

With feedback from Benjamin Poulain and Alex Christensen, who are behind
a lot of the content blocking code in WebKit, I increased the scope of
my patch to handle the _css-display-none_ content blocking action type
and also work for other types of fire-and-forget requests apart from `<a
ping>` pings.

My patch was [accepted](https://trac.webkit.org/changeset/191167), and
shipped with the WebKit used in iOS 10. I could finally update Hide &
Seek to handle Google search results correctly. It took me a while to
get that done (I'm taking time off to care for my baby daughter), and
that's now
[shipped](https://twitter.com/hidenseekapp/status/843275223243743233).

And that's the story of how a problem in my app led to a fix in WebKit.

