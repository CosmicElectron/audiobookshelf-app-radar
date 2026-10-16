# Audiobookshelf app, plus

The official [Audiobookshelf Android app](https://github.com/advplyr/audiobookshelf-app), rebuilt from every
upstream release with one set of changes on top. The changes make the app behave well on a self-hosted server
reached through a tunnel, on a home network, in the car, and alongside other devices listening to the same books.

## What is different

**It starts from the phone.** Everything the player reads for a streamed book goes through an on-disk cache
(768 MB), so the last book's index and the audio around your position are already on the phone. The home page
shows the shelves the server sent last time while the connection is still being made, with covers, and a tap
starts the book. Android Auto answers at once with the cached Continue list. Tapping the last book reuses its
session and the cached audio instead of waiting for the server. Every streamed book plays from the library files
with the current token, so a session the server dropped, or a refreshed token, cannot break a stream.

**Positions follow the listener, not the device.** The phone records where it left each book. When a session
starts at the position another device moved to, or play is pressed after a longer pause, the server is asked
where the book is, and asked again until it answers; until then the phone holds its own progress syncs, so an
older position never overwrites one set later elsewhere. The server wins only when it holds a position this
phone never got it to accept, and a book finished elsewhere starts over. Both positions stay one tap away: a
single tappable notice, "This phone was at …" in the player menu, and a button on the car screen.

**Streaming over a tunnel is smooth.** A sixty-second back buffer, so the rewind after a pause plays from memory;
ten minutes of look-ahead on disk; a five-second rebuffer threshold; one shared HTTP client that tries IPv4 before
IPv6 (a VPN with non-routing IPv6 used to stall every connection); a VPN counted as a network.

**A home-network address.** The account page takes a second address for the same server. While the phone can
reach it and it proves to be the same server (same version, your token validates there), every request including
the audio stream and downloads goes to it; when it stops answering, requests go back to the server address on
their own. Wi-Fi, mobile data and VPN switches re-prove the address at once.

**Series descriptions on the series page**, which the server stores but no client shows. Release Radar writes
its one-line status there ("Audio 2 overdue by 10 days", "Audiobook 3 Nov").

**Listening sessions in the player** menu, for the current book only; tapping one continues from where that
session ended, whichever device it was on.

**Diagnostics on the Logs page**: audio focus and suppression, player and device volume, the audio route and
outputs coming and going, sink and decoder errors, underruns, a heartbeat while playing, and a crash's stack
trace for the next launch.

## How it stays current

A GitHub Actions workflow checks upstream's releases once a day. For a new tag it applies the changes
(`patches/*.patch`) on top of that tag, builds a release APK, signs it with this repository's key, and publishes
`audiobookshelf-plus.apk` as release `v<upstream version>-sd<PATCH_REV>`. If the changes ever stop applying, the
run fails and nothing is published.

Point [Obtainium](https://github.com/ImranR98/Obtainium) at this repository. Releases are normal releases; the
pre-releases are test builds and need "Include prereleases" to be seen.

## Installing over the official app

This build is signed with a different key from the official one, so Android will not update one over the other.
Uninstall the official app once, install this one, and sign in again. Listening progress lives on the server.
Books downloaded to the phone must be downloaded again.

Android Auto hides sideloaded media apps. To see this one in the car: Android Auto settings, tap the Version entry
about ten times to enable developer settings, open the menu, Developer settings, turn on "Unknown sources", then
reconnect.

## Settings worth knowing

- Account page, "Home network address": the server's address on your home network, with the same base path as
  the server address. Leave it empty to always use the server address.
- The pre-buffer stays off on mobile data; the player's own look-ahead does not.
- Settings > Logs: the diagnostics; "Mask server address" hides your hosts when sharing them.

## Secrets the workflow needs

`KEYSTORE_B64` (the signing keystore, base64), `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`.
Keep an offline copy of the keystore: losing it means another uninstall for every install.

## Retiring this

When upstream covers these things itself, or when its client is rewritten, drop the patches and point Obtainium
back at upstream.
