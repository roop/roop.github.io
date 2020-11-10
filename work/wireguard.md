---
layout: page
permalink: /work/wireguard/index.html
---

<section markdown="1">

  <aside class="roop-intro">
  <p>{% include roop-intro.html %}</p>
  <p>Here are some of the things I've worked on.</p>
  </aside>

## WireGuard

I'm working on bringing the [WireGuard] VPN tunnel to iOS and macOS using
the Network Extension framework.

  - iOS: [App Store][ios_appstore], [Announcement][ios_announcement]
  - macOS: [App Store][macos_appstore], [Announcement][macos_announcement]

[ios_appstore]: https://itunes.apple.com/us/app/wireguard/id1441195209?ls=1&mt=8
[ios_announcement]: https://lists.zx2c4.com/pipermail/wireguard/2018-December/003694.html
[macos_appstore]: https://itunes.apple.com/us/app/wireguard/id1451685025?ls=1&mt=12
[macos_announcement]: https://lists.zx2c4.com/pipermail/wireguard/2019-February/003853.html

I initially [tried] to implement [the WireGuard protocol] in Swift, but
abandoned that after learning that the WireGuard team had plans to make
the Go implementation of WireGuard (used in their Android app) available
for use in iOS.

[WireGuard]: https://www.wireguard.com
[tried]: https://github.com/roop/NEWireGuard
[the WireGuard protocol]: https://www.wireguard.com/protocol/

When I took a look at WireGuard after a few months, a C API based on
WireGuard-Go was available that could be used from iOS, and an effort to
make an iOS app was underway.

I proposed a rewrite of the app to:

  - Use the VPN tunnel for data persistance instead of Core Data
  - Redo the UI to support both the iPhone and iPad
  - Redo the UI in code (no Interface Builder)
  - Minimize dependancies on third-party code
  - Enable a macOS version to be developed with the same code base

My pitch was accepted. My work on this project was sponsored by the
[NLnet Foundation][nlnet].

[nlnet]: http://nlnet.nl/

These features are common to both the iOS and macOS versions:

  - Bringing up / bringing down a WireGuard tunnel
  - Creating, viewing and modifying tunnel configurations
  - Importing from a .conf or .zip file
  - Exporting all tunnels to a .zip file
  - On-Demand VPN (interface-type-based and SSID-based)
  - Viewing and exporting the log
  - Showing bytes transferred for active tunnels

The iOS version includes these additional features:

  - Importing through QR code (adapted from the code before my rewrite)
  - State Restoration
  - Support for Dynamic Type

The macOS version includes these additional features:

  - Bringing up / down a Wireguard tunnel from the status bar
  - The app automatically starts when the user logs in, if it was
    running when the user logged off
  - The app appears in the dock only when a window is shown -- if the
    app is just in the status bar, the app isn't shown in the dock

---

**Source code:** [https://git.zx2c4.com/wireguard-ios/](https://git.zx2c4.com/wireguard-apple/)


