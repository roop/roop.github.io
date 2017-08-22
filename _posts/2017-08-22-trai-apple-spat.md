---
layout: post
title: "TRAI’s spat with Apple over app access to messages and call logs"
description: "India's telecom regulator wants to bring its Do-Not-Disturb Android app to iOS, but a direct port of the app is not possible"
tagline: ""
category: posts
---

[Pankaj Doval], _Times of India_ (via [Applesutra]):

> Telecom regulator Trai has accused iPhone maker Apple of engaging in
> "data colonisation" in India and being "anti-consumer" by not allowing
> customers to pass on details about pesky calls and unwanted messages
> to authorities as well as their mobile operators.
>
> ...
>
> "While Google's Android supports our Do-Not-Disturb (DND) app, Apple
> has just been discussing, discussing, and discussing. They have not
> done anything," Sharma told TOI.

[Pankaj Doval]: http://timesofindia.indiatimes.com/business/india-business/trai-raps-apple-for-colonising-data-in-india-says-its-anti-customer/articleshow/59961674.cms
[Applesutra]: https://applesutra.com/2017/08/11/trai-apple-data-fight/

Sharma is the Chairman of the Telecom Regulatory Authority of India
([TRAI]), a government organization that, among other things, oversees the
functioning of mobile network carriers in India.

TRAI has published an Android app called [Do-Not-Disturb][dnd-goog-play]
to help curb marketing cold calls and SMS messages. The app is tied to
TRAI's [NCPR database][NCPR] of mobile users and their opt-in / opt-out
preferences regarding receiving promotionals calls and SMS messages. A
key feature of the app is helping the user report a promotional call or
SMS they have received that doesn't comply with their registered
preference.  To implement this feature, the app requires access to the
phone's call logs and messages. It displays recent calls and messages
that the user can pick from and report a complaint on.

When the Android app was launched last year, TRAI [did have plans] of
bringing a similar app to iOS as well, but that hasn't happened so far.
A Public Interest Litigation had been filed in the Delhi High Court
asking for such an app to be made available on the App Store, and was
[dismissed][PIL] by the court last month.

[TRAI]: http://www.trai.gov.in/
[NCPR]: http://www.nccptrai.gov.in/
[dnd-goog-play]: https://play.google.com/store/apps/details?id=trai.gov.in.dnd&hl=en
[did have plans]: http://gadgets.ndtv.com/apps/news/trai-launches-dnd-services-app-to-register-pesky-call-complaints-844157
[PIL]: http://www.ptinews.com/news/8932791_HC-declines-to-entertain-plea-against-Apple.html

There's no public API in iOS to read call logs or SMS messages. There
are ways to block certain callers and filter certain SMS messages [^1],
but the provided API is restricted because of privacy considerations. To
block callers, apps can provide a block list but have no way to
intercept calls or access the call log. To filter SMS messages, apps can
classify SMS messages from unknown senders as they come, but are not
allowed to keep track of them and use them later on (like showing a list
of messages to choose from).

[^1]: The message filtering capability is part of iOS 11, to be released later this year

### What TRAI could do

TRAI could, with the current available API in iOS, write an app that
could be useful in reporting spam calls by including a [share extension]
for contacts [^2].

The user, while viewing the call log on an iPhone, can tap the info
button and then the 'Share Contact' button to bring up a share sheet
that could show a TRAI share extension to report complaints.
The share extension would get the phone number from the
shared contact information, but there's no way to share the date the
spam call was made.  However, there are only three possible options:
'Today', 'Yesterday' and 'Two days ago' (because TRAI requires that the
complaint be filed within 3 days), so it should be okay to ask the user
for that information.

So that would give us a pretty good way to report spam calls.

This share extension method would also work for reporting promotional SMS
messages received from actual phone numbers, but would unfortunately not
work for SMS messages with alphanumeric sender IDs. That means in most
SMS reporting scenarios, this wouldn't be useful.

[share extension]: https://www.imore.com/sharing-ios-8-explained
[^2]: With Uniform Type Identifier as "public.vcard"

### What Apple could do

I think Apple is highly unlikely to create public APIs to access call
logs or messages, but that's not really needed for an app like this.

Rather, Apple could provide a way to share an SMS through a share sheet,
so a third-party app can get access to all relevant fields of an SMS
message (i.e. body text, sender id and timestamp) that the user
explicitly chose to share with the app. That way, TRAI can create an app
to report errant SMS messages, while Apple can retain their right to not
give apps blanket access to the phone's SMS data.

So this is how I think TRAI and Apple can settle this without
compromising either party's positions, and simultaneously keep Apple's
privacy-conscious customers happy as well.

---

