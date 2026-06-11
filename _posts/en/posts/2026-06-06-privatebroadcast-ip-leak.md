---
title: Private Broadcast May Reveal Sender IP Address in Bitcoin Core 31.0
name: privatebroadcast-ip-leak
id: en-privatebroadcast-ip-leak
lang: en
type: posts
layout: post
excerpt: A bug in the -privatebroadcast feature, newly introduced in Bitcoin Core 31.0, may reveal the originator's IP address to the receiving peer under certain network conditions. Workarounds are available, and the issue will be fixed in the upcoming 31.1 release.
---

We have become aware of a privacy bug in the `-privatebroadcast` feature, newly
introduced in Bitcoin Core 31.0, that may cause the originator's IP address to be
revealed to the receiving peer under certain network conditions. A fix is
forthcoming and will be released with 31.1. Users of `-privatebroadcast` are
advised to apply one of the workarounds below until 31.1 is released.

## Affected users

This bug affects users where all the following are true:

- The node is running Bitcoin Core 31.0 and `-privatebroadcast` is set.
- Transactions are broadcast using the `sendrawtransaction` RPC.
  Wallet RPCs (`sendtoaddress`, `sendall`, etc.) do not use private
  broadcast and are not affected.
- Tor is reachable for outbound connections.
- Outbound IPv4 or IPv6 connections can be made directly. No `-onlynet`
  restriction excludes them, and no `-proxy=...` value applies to them.
- BIP324 v2 transport is not disabled with `-v2transport=0`.

## Impact

When private broadcast selects an IPv4 or IPv6 peer that advertises
support for v2 (BIP324) transport, the initial connection is routed
through the Tor proxy as expected. If the v2 handshake fails on that
connection, Bitcoin Core retries it as v1. The v1 retry is not routed
through the Tor proxy and instead connects directly to the peer over
IPv4 or IPv6, exposing the originator's IP address to the recipient.

Initial v1 connections (to peers that do not advertise v2) are
correctly routed through the Tor proxy and are not affected. The
bug is specific to the v1 reconnection that follows a failed v2
handshake. Connections to onion and I2P peers are also unaffected,
because they remain routed through their respective proxies on any v1
retry and therefore never expose a clearnet IP address.

This breaks the privacy guarantee stated in the 31.0 release notes:
"Their IP address (and thus geolocation) is never known to the recipients".

### How this can happen

A v2 handshake is unlikely to fail for a peer that actually supports v2
transport. The bug is most likely to be triggered by a malicious
peer deliberately closing the v2 handshake to force a v1 retry.

## Workarounds

Until they can upgrade to 31.1, users of `-privatebroadcast` should apply one
of the following:

1. **Disable the feature.** Set `-privatebroadcast=0`.

2. **Disable v2 transport.** Set `-v2transport=0`. This causes all of the
   node's connections to use the unencrypted v1 protocol, which has the downside
   that it becomes easier to fingerprint and censor on clearnet.

3. **Route IPv4/IPv6 outbound through Tor.** Set
   `-proxy=127.0.0.1:9050` (replace `9050` with your Tor SOCKS
   port if different). This routes _all_ outbound IPv4/IPv6 P2P
   traffic through Tor exit nodes, which has the downside of making the node
   easier to Sybil attack.

## Credits

Credit to Eugene Siegel for discovering the bug.

{% include references.md %}
