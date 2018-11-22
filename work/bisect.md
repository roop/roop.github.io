---
layout: page
permalink: /work/bisect/index.html
---

<section markdown="1">

  <aside class="roop-intro">
  <p>{% include roop-intro.html %}</p>
  <p>Here are some of the things I've worked on.</p>
  </aside>

## Bisect

I created a split-screen Markdown editor for the iPad before iOS
introduced Split View.

The app was released in July 2015 (a few weeks after Apple announced
Split View for iPad). It didn't sell well, and was
[discontinued](http://roopc.net/posts/2016/sunsetting-bisect/) in
November 2016.

The app consists of three panes: a Web browser, a Markdown editor and a
Markdown preview pane. We can swipe between the panes and can switch
between split-screen and full-screen modes.

### Browser

The browser had a unified search/address bar, tabs and custom context
menus.

Tabs out of view are offloaded to disk when under memory pressure. To
maintain tab history of offloaded tabs, Bisect maintains a
back-forward navigation list separate from WKWebView's internal list.

### Markdown editor

The editor provides accurate Markdown syntax highlighting by using a
Markdown parser run to generate the highlighting ranges. The same run of
the Markdown parser generates the Markdown preview.

While editing, the keyboard accessory view features context-specific
Markdown actions. Some of the things that can be accomplished using
these context-specific actions are:

 - Open a URL in the browser pane
 - Fill a link's URL from the browser pane
 - Fill in link references using references defined in the document
 - Increase or decrease the heading level

### Markdown preview

To generate a fast preview as the user edits the text, Bisect diffs the
current HTML output with the previous HTML and applies the diff onto the
WKWebView using JavaScript. To make the diffing fast, Bisect creates a
DOM-like tree of the output HTML when parsing the Markdown text.

---

**App website:** <http://bisectapp.com/> <br/>
**App preview video:** <https://vimeo.com/282777436/>
