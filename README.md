# Audiobookshelf app, with series descriptions

The official [Audiobookshelf Android app](https://github.com/advplyr/audiobookshelf-app) plus one small patch:
the series page shows the series' description, which the server stores but no client displays. Release Radar
writes a one-line status there ("Audio 2 overdue by 10 days", "Audiobook 3 Nov", "Audio 6 est. 13 Feb 2027").

Patch 2 makes streaming over a tunnel bearable: a 60 s back buffer so the auto-rewind after a pause plays from
memory, a 5 s rebuffer threshold, one shared HTTP client that tries IPv4 before IPv6 (a VPN with non-routing
IPv6 used to stall every connection), a VPN counted as a network, and a "Listening sessions" entry in the
player's menu listing the server's sessions for the current book.

Patch 3 adds a "Home network address" field on the account page. The server address stays the server's
identity (the account, tokens, downloads and progress all keep it); the home address is only a route. While
the phone can reach it and it proves to be the same server (same version, our token validates there), every
request including the audio stream and downloads goes to it; when it stops answering, requests go back to the
server address on their own, so the app works unchanged on the home Wi-Fi, on mobile data, and with a VPN.

Patch 4 keeps streamed audio on the phone. Everything the player reads passes through a 768 MB on-disk cache
(the index at the head of the file, the audio around the playhead), and a pre-buffer keeps one minute behind
and ten minutes ahead of where the last book was paused, so the next start plays from the phone while the
server is only asked for a session. The cache is keyed by file, not URL, so it survives new sessions, token
refreshes and the home-network switch. It also writes playback diagnostics to Settings > Logs: audio focus
and suppression, player and device volume, audio outputs coming and going, sink and decoder errors,
underruns, and a heartbeat every five minutes while playing.

It also keeps two positions per book the way Audible does. The phone records where it left each book; when
a session starts at the synced position another device moved to, or when play is pressed after a longer
pause and the server turns out to be ahead, playback continues from the synced position and the player's
menu offers "This phone was at 3:12:08" to jump back (and the synced position to jump forward again).
Both windows are cached, so either jump is instant. The pre-buffer stays off on metered networks.
The patches are `patches/*.patch`.

## How it stays current

A GitHub Actions workflow checks upstream's releases once a day. For a new tag it applies the patch on top of
that tag, builds a release APK, signs it with this repository's key, and publishes `audiobookshelf-series.apk`
as release `v<upstream version>-sd<PATCH_REV>`. If the patch ever stops applying, the run fails and nothing
is published.

Point [Obtainium](https://github.com/ImranR98/Obtainium) at this repository. Releases here are normal
releases, so no prerelease setting is needed.

## Installing over the official app

This build is signed with a different key from the official one, so Android will not update one over the
other. Uninstall the official app once, install this one, and sign in again. Listening progress lives on the
server. Books downloaded to the phone must be downloaded again.

## Secrets the workflow needs

`KEYSTORE_B64` (the signing keystore, base64), `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`.
Keep an offline copy of the keystore: losing it means another uninstall for every install.

## Retiring this

When upstream shows series descriptions itself, or when its client is rewritten, drop the patch and point
Obtainium back at upstream.
