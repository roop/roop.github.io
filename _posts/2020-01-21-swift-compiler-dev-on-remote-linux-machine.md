---
layout: post
title: "Swift compiler development on a remote Linux machine"
tagline: ""
description: "Notes on setting up a Linode for working on the Swift compiler"
category: posts

---

Building Swift is intensive on CPU, memory, and diskspace. Working on
the compiler in parallel with app development wouldn't have been
possible on my Macbook Air. So, I've been doing my Swift compiler work
on a Linode server instead.

If you're interested in starting on Swift compiler development, renting
out a remote server like this can be a great way to get your feet wet.

So, let's get started on setting up a remote server for Swift compiler
development.

[^1]: Instead of using Vim, it might be possible to run VSCode or Qt
      Creator over VNC, but I haven't experimented with that approach
      yet.

* TOC
{:toc}

### Choose a remote Linux machine

Choosing a machine configuration depends on the system requirements for
building Swift.

For hacking on Swift compiler features, the Swift repository's [README]
recommends building with `--release-debuginfo --debug-swift`, which
includes debug information when compiling llvm, the Swift compiler, and
the Swift standard library. A build like this would require at least 8
GB of RAM.

[README]: https://github.com/apple/swift/blob/master/README.md#building-swift

However, we can reduce the RAM required -- and thereby adopt a less
expensive server configuration -- by building just the compiler part
with debug symbols, with `--release --debug-swift`. Doing that would
enable us build Swift on a 4 GB RAM machine. As long as we're only going
to work on compiler features, this build shall suffice.

In terms of disk space, if we build only Swift with debug symbols, we'll
need about 30 GB per build; if we build everything with debug symbols,
we'll need about 80 GB per build.

Apart from memory and diskspace, it's a good idea to pick a server close
to our location to reduce latency.

Once the Linux server is created and booted, we can start setting it up.

### Set up the remote Linux machine

#### Set up a user

The freshly created server only has a root login now. We can login to
that machine using the root password specified or obtained while
creating the VPS:

    $ ssh root@<ip-address> # Enter password

Once logged in, create a new user. Specify a password when prompted. You will need
this password later when you use `sudo` commands.

    # adduser <user> # Specify and remember password. Other info can be empty.
    # usermod -aG sudo <user> # Give <user> sudo privileges
    # su <user> # Switch to that user

#### Set up SSH keys

Create the `.ssh` directory:

~~~
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
~~~

Assuming you have an SSH keypair handy (they should be in ~/.ssh on
your local machine), add your public key as an authorized key on
the remote server.

~~~
$ cat >> ~/.ssh/authorized_keys # Paste your public key
$ chmod 600 ~/.ssh/authorized_keys
~~~

This should enable you to `ssh` into the remote server without having to
type the server password.

Now, it's a good idea to disable password-based login:

~~~
$ sudo vim /etc/ssh/sshd_config # set 'PasswordAuthentication' to 'no'
$ sudo systemctl reload sshd # Reload the SSH daemon
~~~

#### Set up tmux

We use tmux to keep terminal sessions alive on the server even when our
SSH connection breaks. tmux is a terminal multiplexer, so one tmux
session can encapsulate multiple terminal sessions.

Install tmux on the server, if it's not installed already.

~~~
$ sudo apt-get install tmux
~~~

Create a new tmux session called "swift" in detatched mode.

~~~
$ tmux new-session -s swift -d
~~~

The newly created tmux session will have one tmux window, and therefore
one terminal session. We can list the active tmux sessions by saying:

~~~
$ tmux list-sessions
~~~

We'll see that we have one tmux session with one window. We can create
more tmux windows and thereby, more terminal sessions. There are tmux
commands to do that, but an easier way to work with tmux is through
iTerm2 from your Mac.

### Set up the local macOS machine

#### Set up iTerm2

If you don't have iTerm2 on your Mac,
[download](https://iterm2.com/downloads.html) and install it. The
following instructions assume iTerm2 version 3.3.7.

 1. Open iTerm2

 2. Go to _iTerm2 > Preferences > Profiles_ and add a new profile (say
    "Swift dev") using the "+" at the bottom left.

 3. Select that profile on the left, then select the _General_ tab on
    the right.

    Under the _Command_ section, select _Command_ and enter the
    command as:

    ~~~
    ssh -t <user>@<ip-address> "tmux -CC attach -t swift"
    ~~~

    substituting your remote server username and ip address.

#### Using iTerm2's tmux integration

Once we've created the iTerm2 profile, if we right-click on iTerm2 on
the dock, and click on _New Window … > Swift dev_ (or whatever you named
your profile), iTerm2 will:

  - Open a new window and run the ssh command
  - Once the ssh connection is established, run the tmux attach command
    on the server to attach to the tmux session we'd started
  - Get the state of the tmux windows in the tmux session and show them
    as new tabs or windows in iTerm2

The tmux session we had created on the server has just one window. So if
we right-click on iTerm2 and click on _New Window … > Swift dev_, two
new windows shall open:

 1. First, a window that runs ssh and then the tmux attach command.
    This window shall have no command prompt.
 2. Next, a window that shows the tmux session's tmux window, with a
    command prompt.

With the second window selected, iTerm2's _New Window_ and _New Tab_
commands will show a prompt asking if we want to create a new tmux tab
or window. If we pick that option, iTerm2 will create a new tmux window in
our tmux session and will open that window in a new tab / window.
When closing such a tab / window, iTerm2 will ask us if we want to
kill that window in the tmux session, or do we want to just hide it. I
prefer killing a tmux window when the corresponsing tab is closed.

If you're done working on the server, you can close the first window,
which shall close the ssh connection and will then close all the tmux
windows. This can also happen when the ssh connection is broken due to
inactivity or a broken network connection.

To get back to working on the server, you can right-click on iTerm2 and
click on _New Window … > Swift dev_ again, and all our tmux windows will
be back.

#### Configuring iTerm2's tmux integration

This is how I like my iTerm2 configured for working on Swift:

  - Make each tab correspond to a tmux window without showing the
    prompts mentioned earlier

    Go to _iTerm2 > Preferences > Advanced_, search for "tmux".

      - Under _Tmux Integration_, turn on _Suppress alert asking what
        kind of tab to use in tmux integration_ and _Suppress alert
        asking what kind of window to open in tmux integration_.

      - Under _Warnings_, turn on _Suppress kill/hide dialog when
        closing a tmux tab_ and _Suppress kill/hide dialog when closing
        a tmux window_.

    With that, while working on a tmux window, ⌘-T and ⌘-N open new tmux
    windows, and closing a tab with a tmux window kills the tmux window.

  - Use different-sized windows

    Previously, tmux required that all tmux windows in a session be
    resized to the size of the smallest client viewport. So when we have
    tmux windows from one tmux session spread across multiple iTerm2
    windows, resizing one of those iTerm2 windows will resize the others
    as well.

    If you have tmux version 2.9 or later (to know the tmux version, run
    `tmux -V` on the server), we can fix this:

      - Go to _iTerm2 > Preferences > Advanced_. Under _Experimental
        Features_, turn on _Allow variable window sizes in tmux
        integration_.

      - Go to _iTerm2 > Preferences > General_ and select the _tmux_ tab.
        For _Open tmux windows as:_, select "Native windows".

    I learnt about this option from [here][t1].

    [t1]: https://groups.google.com/forum/#!topic/iterm2-discuss/Etfozb_63Pg

  - Account for many tmux windows in a tmux session

    When connecting to a tmux session, if you have too many tmux windows
    (10 tmux windows by default), iTerm2 will show the "tmux dashboard"
    instead of opening the windows directly. We can then open the
    unopened tmux windows in tabs or windows from the dashboard.

    I find this limit of 10 to be too low, so I set it to 20. This can be
    configured in the tmux dashboard itself, by going to _Shell > tmux >
    Dashboard_ in the iTerm2 menu.

  - Color scheme

    By default, the tmux windows are created with a profile called “tmux”.
    I changed the color scheme for this profile by doing this:

     1. Go to _iTerm2 > Preferences > Profiles_
     2. Select _tmux_ on the left, select the _Colors_ tab on the right
     3. Select a preset in _Color Presets_ (I recommend Solarized Dark)

  - Always show tab bar

    Even if there's just one tab in a window, I like the tab bar to show
    up so that I can easily rearrange tmux windows into iTerm2 windows
    as I want. To do that:

      1. Go to _iTerm2 > Preferences > Appearance_
      2. Go to the _Tabs_ tab
      3. Turn on _Show tab bar even when there is only one tab_

If you want to poke around for more tmux-specific configuration options,
you can try these:

  - See the [iTerm2 wiki page] on configuring iTerm2's tmux integration
  - Go to _iTerm2 > Preferences > General_ and select the _tmux_ tab
  - Go to _iTerm2 > Preferences > Advanced_ and search for "tmux"

[iTerm2 wiki page]: https://gitlab.com/gnachman/iterm2/-/wikis/tmux-Integration-Best-Practices 

### Build Swift

#### Get the dependencies

On the server, install all the required dependancies to build Swift
using the `apt-get` command [from the Swift
repository](https://github.com/apple/swift/#linux).

#### Get the code

Before building Swift, we need to clone Swift repository, and a
bunch of other repositories as well. We create a directory under
which all these repositories will be placed:

    $ mkdir ~/swift-source

And then clone all the required repositories under that directory:

    $ cd ~/swift-source
    $ git clone https://github.com/apple/swift.git
    $ ./swift/utils/update-checkout --clone

#### Build it

The Swift project's build system uses CMake to create Ninja build
files, and Ninja to use those build files to compile and link code.
The Swift repository includes a build script called `build-script`
that builds CMake and Ninja from source, and then uses that CMake
and Ninja to build llvm, Swift and the Swift standard library.

By default, `build-script` will spawn as many parallel jobs as there
are CPUs in the machine, which is good while compiling. However,
linking uses a lot of memory, so we don't want to be doing multiple
link jobs at a time.

<s>There's no way I know of to tell the build
system to not use parallel jobs only while linking, so the easy way
out is to ask the build script to run all jobs sequentially (`-j1`).
This will however increase the initial build time quite a bit.</s>

**Update:** To tell the build system not to use parallel jobs while
linking, we can set the relevant CMake properties using the
`--llvm-cmake-options` and `--swift-cmake-options` arguments.

To be able to work on compiler features, we want to build the compiler
in debug mode. If you work on other parts of Swift (say the Swift
standard library), or on other aspects of the compiler (like
performance), the options would be different.

To build only Swift in debug mode and everything else in release mode,
we say:

    $ cd ~/swift-source
    $ ./swift/utils/build-script --release --debug-swift --llvm-cmake-options==-DLLVM_PARALLEL_LINK_JOBS=1 --swift-cmake-options=-DSWIFT_PARALLEL_LINK_JOBS=1

This build can take about 3 hours, and will write the build results to
`build/Ninja-ReleaseAssert+swift-DebugAssert`.

It helps to set up `SWIFT_BUILD_DIR` and `PATH` environment variables
like this:

~~~
$ cat >> ~/.bash_profile
export SWIFT_BUILD_DIR=~/swift-source/build/Ninja-ReleaseAssert+swift-DebugAssert/swift-linux-x86_64
export PATH=${PATH}:${SWIFT_BUILD_DIR}/bin # For swift, swiftc, etc.
export PATH=${PATH}:~/swift-source/llvm-project/utils/lit # For lit.py
$ source ~/.bash_profile
~~~

Having `swift` and `swiftc` in our `PATH` would help us run the
built Swift from any directory. Having `lit.py` in our `PATH` would
help us run the Swift testsuite from any directory.

In case you want to build everything with debug information (and your
server has 8 GB RAM or more), you can pass `--release-debuginfo` instead
of `--release` to `build-script`. This build will take longer to
complete, and the build results will be placed in a different directory.

### Set up the development environment

#### Git

Setup username and email for creating commit messages.

    git config --global user.name “Roopesh Chander”
    git config --global user.email roop@roopc.net

Setup the editor for editing commit messages and interactive rebase. I
use vim.

    git config --global core.editor vim

#### Vim

The Swift compiler's C++ code uses two spaces for indentation, so we can
add these to our `~/.vimrc`:

    set expandtab
    set tabstop=2
    set shiftwidth=2

For help in editing Swift, SIL and gyb files, we can use the vim support
files from the Swift repository itself by:

  - adding this line to `~/.vimrc`:

        set runtimepath+=~/swift-source/swift/utils/vim

  - making the filetype detection files accessible under `~/.vim/ftdetect`:

        $ mkdir -p ~/.vim/ftdetect
        $ cd ~/.vim/ftdetect
        $ ln -s ~/swift-source/swift/utils/vim/ftdetect/*.vim .

#### lldb

We just need to install lldb:

    $ sudo apt-get install lldb

### Set up our fork of Swift

Contributions to the Swift compiler are done through GitHub pull
requests. To create pull requests, we should have a fork of the Swift
repository under our GitHub account. To fork Swift, go to [Swift's
GitHub page](https://github.com/apple/swift/) on a web browser and click
on the "Fork" button.

We already have a clone of the main Swift repository. We want to be able
to push from there to our fork on GitHub. We don't want to be specifying
our GitHub password every time we push, so we'd like to authenticate to
GitHub automatically using SSH keys. However, if we make our private
keys available on the server, either by copying our private keys or by
using SSH forwarding, it means that in case the server is compromised,
that key can be used to control everything in our GitHub account. So, in
the interest of security, we're going to create a separate keypair that
we shall use to push to just one repository -- our fork of Swift. Since
this key controls just this one repository, we can even afford to make
the key not password-protected.

 1. On the server, create a keypair:

        $ ssh-keygen

    and just press enter for the prompts (no password required). This
    shall create `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`.

 2. Start the SSH agent and add the private key to it:

        $ eval `ssh-agent -s`
        $ ssh-add ~/.ssh/id_rsa

 3. Copy the public key:

        $ cat ~/.ssh/id_rsa.pub # Copy the output

 4. Add this public key as a _Deploy key_ in GitHub:

      - Go to your fork's GitHub page on a web browser (the URL should
        be like `https://github.com/<your-github-username>/swift`) and
        go to _Settings > Deploy keys > Add deploy key_
      - Under _Title_, specify a title so that we can identify the
        deploy key's origin in the future
      - Under _Key_, paste the public key we copied in the previous step
      - Tick the checkbox to _Allow write access_
      - Click on "Add key"

 5. Back on the server, go to our clone of the Swift repository and add
    our fork as another remote called `fork`:

        $ cd ~/swift-source/swift
        $ git remote add fork git@github.com:<your-GitHub-username>/swift.git

At this point, we have a working setup. If you're new to Swift compiler
development, now would be a good time to start looking at [starter
bugs][StarterBug]. In the [next post], we'll see how we can work on the
Swift compiler with a setup like this.

[StarterBug]: https://bugs.swift.org/issues/?jql=project%20%3D%20SR%20AND%20status%20%3D%20Open%20AND%20labels%20%3D%20StarterBug%20AND%20assignee%20in%20(EMPTY)

[next post]: /posts/2020/swift-compiler-dev-workflow/

If the server reboots, we'll lose the tmux sessions. In that case, we
have to start a fresh tmux session. We should also restart the SSH agent
and `ssh-add` the private key for pushing to [our
fork](#set-up-our-fork-of-swift).

Keep in mind that the server is a recurring cost. So if we know we're
going to not work on the compiler for a few weeks, it can be a good idea
to delete the server and spawn a new one when we're ready. Before
deleting, we should take a look at all local branches and the stash,
push anything worth saving to our fork, and file-transfer any uncommitted
unit testcases, so that we can get back to work with minimum disruption.

If you have questions, corrections, or feedback, please [get in touch].

[get in touch]: /about/#get-in-touch
