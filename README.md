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

## Bench mode

With `&mode=bench` appended to the link, the page is the second peer of the library's bench
(`deploy/test/bench.mjs` in [`meitogi/signal`](https://github.com/meitogi/signal), which prints
that link). It joins two rooms : `<room>`, where a direct path is attempted, then
`<room>-relay` with `connectMs: 1`. The newcomer's `connectMs` is the only one that counts, so
that second room always falls back to the relay. The headline shows both paths. Each
connection's candidates (sent and received) and chosen pair (`getStats()`) are logged when the
bench asks for them, and the bench's summary is displayed when the run ends.

In this mode a text message starting `{"bench"` is a command :

- `{"bench":"report"}` is answered with
  `{"bench":"report", via, lib, ua, local, remote, pair, received, echoed}` ;
- `{"bench":"summary", lines}` is displayed.

Any other message is echoed as in the default mode. `window.state.bench` holds the same facts
per room.

## Library version

The client is imported straight from the package's repository through jsDelivr, **pinned to a
commit** (`LIB` in [index.html](index.html)). jsDelivr caches a branch reference such as `@master`
for up to 12 hours, whereas a commit URL never changes, so the page and a node side running a
local checkout can be checked against each other : the bench compares the `src/` tree of the
pinned commit with its own. After a library change is pushed, move `LIB` to the new commit and
push this repository. Nothing is bundled or copied here ; the page itself is one file,
[index.html](index.html).

The page shows which library it runs, under the headline and in the console. The line gives the
pinned commit and whether it loaded from jsDelivr, then GitHub's compare API says where `master`
stands : `src/ identical to master`, or the number of `src/` files changed since (in red :
re-pin). `window.state.lib` and `window.state.libCheck` hold the same facts. When the library
fails to load, the line says so in red instead of the page waiting forever on « connecting… ».

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
