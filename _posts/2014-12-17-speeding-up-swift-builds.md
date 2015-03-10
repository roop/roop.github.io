---
layout: post
title: "Speeding up Swift builds"
description: "Building Swift projects incrementally"
category: posts
---

<figure>
    <a href="http://xkcd.com/303/"
    ><img src="http://imgs.xkcd.com/comics/compiling.png" alt="XKCD: Compiling"
    ></a>
<figcaption>The "Compiling" XKCD comic is on my mousepad</figcaption>
</figure>

### Why so slow

Swift has no headers and no concept of including or importing a header.
The external interface of a Swift file is defined by the Swift file
itself. As a consequence, when building a Swift project, even if I'd
changed just one Swift file, it can happen that _that_ change _could_
affect how another Swift file in the project should be compiled.

The Swift build system handles this problem somewhat naively 
(as of Xcode 6.1.1) - by ignoring the concept of incremental building.
When we make any change to any Swift file, the next build would compile
all the Swift files.

For a project with several Swift files, this adds up to a significant
build time. As a result, for any decent-sized Swift project, you just
can't have near-instantaneous build times (unlike in C or Objective-C),
even if you've made only a tiny change to a single file.

As a worst case scenario, in case you have some code that the Swift
compiler finds troublesome (like [this][trouble1] or [this][trouble2]),
that code can cause the compilation of the file containing that piece of
code to take a _loooong_ time. Combined with the non-incremental nature of
the build, this means that every time you change something, be it
anywhere in your project, and hit Cmd+R, you'll have to wait a few
minutes before you can see it running on the Simulator.

That's what happend to me, and that was really annoying. I've still not
figured out how best to make my troublesome piece of code less
troublesome to the Swift compiler (nor have I filed a Radar yet), but
thanks to this [tweet by Andy Matuschak] \(via [This Week In Swift]\), I
have a way of speeding up my builds inspite of that.

[trouble1]: http://blog.impathic.com/post/99647568844/debugging-slow-swift-compile-times
[trouble2]: http://stackoverflow.com/questions/25537614/why-is-swift-compile-time-so-slow

[tweet by Andy Matuschak]: https://twitter.com/andy_matuschak/status/543471763892359168
[This Week In Swift]: https://swiftnews.curated.co/issues/21

### Speeding it up

Taking cue from Andy Matuschak's hack, I observed what the Swift build
system was doing in Xcode, and wrote a quick and dirty Perl script that
can create a Makefile for building the project.

To make use of it:

  1. In Xcode, go to Preferences > Locations > Derived Data > Advanced
     and enable a Shared Folder called 'Build' as the build location
     (This is also the first step in [Andy Matuschak's method][tweet by
     Andy Matuschak])
  2. Save [this gist][makemake_gist] as `makemake.pl` in your project's
     root folder
  3. Edit it to populate the app name and swift source file paths
  4. Run `perl ./makemake.pl`. This creates a Makefile.

That's it. To build after changing your code, just run `make`, then
press Cmd+Ctrl+R in Xcode to run without building (Simulator only).

Things to keep in mind:

 - The script has been tested on a single pure-Swift project only
 - Doing `make` should suffice as long as you don't change the external
   API of any Swift file
 - If you see weird unexplained crashes, build through Xcode :-)

Comments, corrections welcome on [the gist][makemake_gist] or through [Twitter].

[makemake_gist]: https://gist.github.com/roop/ec05db594fae8fd2a8eb
[Twitter]: https://twitter.com/roopeshchander/

**Update 02/Mar/2015:** Swift 1.2 addresses this issue head-on, but I
haven't adopted that yet - I'm not sure Swift 1.2 will be out of beta
before I ship a beta of my app for external testing, so I'm sticking with
Swift 1.1 for now and making incremental builds from the command-line.
