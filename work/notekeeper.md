---
layout: page
permalink: /work/notekeeper/index.html
---

<section markdown="1">

  <aside class="roop-intro">
  <p>{% include roop-intro.html %}</p>
  <p>Here are some of the things I've worked on.</p>
  </aside>

## Notekeeper

I made an Evernote client for Nokia's Symbian devices and the Nokia N9.
The app used Qt/QML for the UI, and Qt/C++ for the model, networking and
Evernote API calls.

The UI for Symbian was modelled on the skeuomorphic look of early iOS
versions.

<figure>
<img alt="Notebook-ish look like iOS Notes"
     src="/work/images/notekeeper_symbian_lines.png"
     srcset="/work/images/notekeeper_symbian_lines.png 1x, /work/images/notekeeper_symbian_lines@2x.png 2x"
     style="margin-top: 1.5em" />
<img alt="Text selection"
     src="/work/images/notekeeper_symbian_textselection.png"
     srcset="/work/images/notekeeper_symbian_textselection.png 1x, /work/images/notekeeper_symbian_textselection@2x.png 2x"
     style="margin-top: 1.5em" />
<img alt="Grouped table view"
     src="/work/images/notekeeper_symbian_groupedtable.png"
     srcset="/work/images/notekeeper_symbian_groupedtable.png 1x, /work/images/notekeeper_symbian_groupedtable@2x.png 2x"
     style="margin-top: 1.5em" />
<figcaption>
Notekeeper's iOS-inspired UI: (1) Notebook-ish look like iOS Notes (2) Text selection (3) Grouped table view
</figcaption>
</figure>

I started working on the app in July 2011, made the first release in
March 2012, and continued to make enhancements to it through the year.

I also started working on the Nokia N9 version in parallel, and
developed a way to keep most of the code -- including a lot of the UI
code -- shared between the two versions of the app. The version for N9
was released in Jan 2013.

Both versions of the app were well received. The Symbian version was
[reviewed by All About Symbian](http://www.allaboutsymbian.com/reviews/item/14605_Notekeeper.php).

---

**App website:** <http://www.notekeeperapp.com/> <br/>
**App blog:** <http://blog.notekeeperapp.com/> <br/>
**Source code:** <http://github.com/roop/NotekeeperOpen/>
