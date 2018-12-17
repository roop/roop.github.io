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
the Network Extension framework. The iOS version is now [in beta].

[in beta]: https://lists.zx2c4.com/pipermail/wireguard/2018-November/003526.html

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

My pitch was accepted, and I'm currently working on this project. My
work on this project is sponsored by the [NLnet Foundation][nlnet].

[nlnet]: http://nlnet.nl/

The features implemented so far in the iOS version are:

  - Bringing up a WireGuard tunnel
  - Creating, viewing and modifying tunnel configurations
  - Importing from a .conf or .zip file
  - Importing through QR code (adapted from the code before my rewrite)
  - Exporting all tunnels to a .zip file
  - On-Demand VPN
  - State Restoration
  - Support for Dynamic Type

---

**Source code:** [https://git.zx2c4.com/wireguard-ios/](https://git.zx2c4.com/wireguard-ios/)


