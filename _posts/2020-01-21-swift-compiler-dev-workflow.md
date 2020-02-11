---
layout: post
title: "A workflow for Swift compiler development"
tagline: ""
description: "Notes on the workflow I use for working on the Swift compiler"
category: posts

---

If you're new to Swift compiler development, it can be helpful to look
at a workflow of how development can happen.

In this post, I'll be describing the parts of the workflow I use when
working on compiler features. If you work on other parts of Swift (say
the Swift standard library), or on other aspects of the compiler (like
performance), this might not be helpful.

This post assumes that:
  - The Swift source code repositories are checked out under
    `~/swift-source/`
  - The Swift repository is at `~/swift-source/swift`
  - The Swift repository has two remotes, `origin` and `fork`: `origin`
    points to Apple's Swift repository on GitHub, and `fork` points to
    our fork of that repository on GitHib
  - The `SWIFT_BUILD_DIR` environment variable is set to a Swift build
    directory

My [previous post] talks about one way to get a setup like that.

[previous post]: /posts/2020/swift-compiler-dev-on-remote-linux-machine/

My workflow uses the following parts. The order of using these parts
varies depending on what I work on.

* TOC
{:toc}

### Starting off

When starting to work on a fix or feature, we should create a new branch
off master:

    $ git checkout -b <feature-branch> master

### Debugging

To debug a compilation of `swiftc file.swift`, we can't just run `lldb
-- swiftc file.swift` because `swiftc` invokes itself and other
executables as separate processes to do the actual compilation.

To see the underlying commands without running them, we can run:

    $ swiftc file.swift -###

Typically, we want to run the debugger on the first command printed, so
we can say:

    $ lldb -- `swiftc file.swift -### | head -n 1`

The [DebuggingTheCompiler.rst] document in the Swift repository has some
helpful information on debugging. I'm keeping [some notes][roop debug
notes] as well.

[DebuggingTheCompiler.rst]: https://github.com/apple/swift/blob/master/docs/DebuggingTheCompiler.rst
[roop debug notes]: https://gist.github.com/roop/94308a77aed665ed666d0d2472d3eec5

### Making changes

When committing, we should follow the Swift [commit message guidelines].

[commit message guidelines]: https://swift.org/contributing/#commit-messages

To rebuild Swift after making changes, we can run:

    $ cd $SWIFT_BUILD_DIR
    $ ninja swift

If there were no changes to header files, rebuilding Swift would
typically take only a few minutes.

### Syncing up with master

If we've been working on our feature branch for a while (say a
week), and if we have commits as work-in-progress on our feature
branch, it would be a good idea to rebase our feature branch on top
of the latest commits in master.

Assuming we're on the feature branch:

    $ git fetch origin master
    $ git rebase origin/master

If the are conflicts, they would have to be resolved during the
rebase.

After rebasing to master, we'll have to rebuid Swift again.

    $ cd $SWIFT_BUILD_DIR
    $ ninja swift

We're rebuilding only Swift, while the changes we fetched on master
could have included changes to other parts of the project, like llvm
and the Swift standard library.

  - In case the previously built version of Swift standard library
    is out of date, we'll get an error like this when running
    `swift` or `swiftc`:

    ~~~
    error: compiled module was created by an older version of the compiler; rebuild 'Swift' and try again
    ~~~

    Then we should rebuild the standard library as well, like:

        $ cd $SWIFT_BUILD_DIR
        $ ninja swift swift-stdlib

  - In case the previously built version of llvm is out of date, we
    will get errors referring to llvm header files while compiling
    Swift. In that case, we should rerun the build script.

    First, we update all the repositories:

    ~~~
    $ cd ~/swift-source/swift
    $ git checkout master
    $ cd ~/swift-source
    $ ./swift/utils/update-checkout
    ~~~
    
    If there are commits in our feature branch, we can switch to
    that branch now:

    ~~~
    $ cd ~/swift-source/swift
    $ git checkout <feature-branch> # rebase again onto master if reqd.
    ~~~

    We can now run `build-script` with the same options as the [initial
    build]. Incremental runs of `build-script` take lesser time than clean
    runs.

[initial build]: /posts/2020/swift-compiler-dev-on-remote-linux-machine/#build-it

### Testing

#### Run the testsuite

To make sure our changes haven't affected working parts of the compiler,
we should test our version of Swift against the testsuite. The expected
output is embedded inside the test file, and the output is verified
using llvm's [FileCheck] utility.

[FileCheck]: https://www.llvm.org/docs/CommandGuide/FileCheck.html

We use the `lit.py` script for that. The following commands assume that
`lit.py` is available in our `PATH`.

By default, `lit.py` prints one of "PASS:", "FAIL:" or "XFAIL:" (which
means expected failure) for every testcase. Passing `-s` suppresses
that.

If we pass `-v`, `lit.py` shall print the commands it invoked for
failing tests, so we can try to run it ourselves. 

Typically, we know our changes are likely to affect only certain aspects
of the compiler, so we run only certain tests:

  - To run tests under a certain test directory:

    ~~~
    $ lit.py -sv $SWIFT_BUILD_DIR/test-linux-x86_64/<test-directory>
    ~~~

    where `<test-directory>` is a directory path under
    `~/swift-source/swift/test/`.

  - To run tests matching a filename pattern:

    ~~~
    $ lit.py -sv $SWIFT_BUILD_DIR/test-linux-x86_64/<test-directory> --filter=<pattern>
    ~~~

If we're about to open a pull request, we might want to run the full
testsuite:

    $ lit.py -s $SWIFT_BUILD_DIR/test-linux-x86_64/

#### Add to the testsuite

Most changes to the compiler will require that the change be tested. In
that case, the pull request should include an addition to the testsuite
to verify the change.

#### More on the testsuite

  - [Testing.md] in the Swift repository
  - [codafi's post] on the Swift forums

[Testing.md]: https://github.com/apple/swift/blob/master/docs/Testing.md
[codafi's post]: https://forums.swift.org/t/need-a-workflow-advice/12536/14

### Open a pull request

To open a pull request, we should:

 1. Push the feature branch to our fork

        $ git push fork <feature-branch>

 2. Go to your forked repository on GitHub on a web browser,
    where you should see an option to create a pull request

 3. When opening the pull request, we should:

      - include the bug number resolved by the pull request
      - @-mention a potential reviewer, ideally from the Swift team

    If it's not obvious who should review something, we could look at
    [CODE_OWNERS.TXT] in the Swift repository.

[CODE_OWNERS.txt]: https://github.com/apple/swift/blob/master/CODE_OWNERS.TXT

Once we open a pull request, we should wait till a reviewer takes this
up for review. Only those who have commit access can ask
the [@swift-ci build bot] to invoke automated testing on the pull request.

[@swift-ci build bot]: https://github.com/apple/swift/blob/master/docs/ContinuousIntegration.md

### Modify a pull request

We might have to change a pull request for different reasons:

  - to make corrections in the implementation
  - to take a different approach
  - to address conflicts with changes in the master branch

If we add commits to the same branch and push that to the same branch on
our fork, and they will be added to the pull request automatically.

    $ git push fork <feature-branch>

If we end up with a different set of commits altogether (maybe we used
interactive rebase to edit the commit history, or we rebased to a
different base commit), we can force-push to the same branch on our
fork, and the pull request will be updated automatically.

    $ git push -f fork <feature-branch>

Force-pushing can sometimes cause the @swift-ci build bot to report an
error even when there's no error. In case the build bot reports an
error, we should take a look at its logs to check if it's a genuine
error or not.

If you have questions, corrections, or feedback on this workflow, please
[let me know].

[let me know]: /about/#get-in-touch

