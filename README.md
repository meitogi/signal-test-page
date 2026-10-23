# signal-test-page

A static second peer for [`@meitogi/signal`](https://github.com/meitogi/signal), served from
GitHub Pages so a phone on its own network can take part in a connectivity test without any
tunnel to the machine running the other side.

**Page** : https://meitogi.github.io/signal-test-page/

Open it with a room link — the output of the library's `formatLink()` :

```
https://meitogi.github.io/signal-test-page/#r=<room>&s=<secret>&relay=wss://<relay host>
```

The page joins that room with the library's default STUN server (`stun:stun.l.google.com:19302`),
shows the path it got in large type (`via direct` or `via relay`), sends the other peer one JSON
message `{ "report": "via", "via": "direct" | "relay", "ua": "<user agent>" }`, then echoes every
message it receives : a string comes back prefixed `echo:`, bytes come back reversed.
`window.state` holds the same facts for a browser driven over CDP.

The client is imported straight from the package's repository through jsDelivr
(`meitogi/signal@master/src/index.js`) ; nothing is bundled or copied here. The page itself is
one file, [index.html](index.html).

## Why

A WebRTC peer that only ever met a browser on the same machine has never proven it can reach the
internet : STUN out, ICE checks out, a remote NAT answering. The simplest second peer for that
proof is a phone on cellular data — and the simplest way to hand that phone a page is to host it
where it already is : on the web. The node side of the test (the room, the assertions) lives with
the project under test ; this page is only the other end.

## Publishing

GitHub → repository Settings → Pages → *Deploy from a branch*, branch `main`, folder `/ (root)`.
A push to `main` redeploys the page within a minute.

## License

MIT — see [LICENSE](LICENSE).
