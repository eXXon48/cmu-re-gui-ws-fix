# GUI WebSocket fix

`cmu-re-gui-ws-fix` · Revised Store package · Mazda CMU150, firmware 74.00.324A

The stock HMI message handler holds some screen messages in a reorder queue that never releases them properly: focus changes (for example to CarPlay or Android Auto) and screen transitions get lost, and screen changes can lag. This delivers every message in arrival order instead.

## Why

The native UI controller (MMUI) sends screen changes (`ctxtChg`), app focus changes (`focusStack`) and transitions (`transition`) to the HMI, each with a sequence byte. The stock `onmessage` handler in `Websockets.js` is meant to reorder them, but its queue does not work:

- A queued `focusStack` or `transition` is never matched: `_isStack` compares the whole message text with the literal `"focusStack"` / `"transition"`. (`_isStack` and `_delStack` are also plain functions called as methods, so every call throws a TypeError.)
- Every 50 messages the queue is flushed, but its `[message, seq]` pairs go to `JSON.parse`, fail, and are dropped.
- A focus change without a screen change always lands in the queue, so it is always lost.
- While the queue is not empty, screen changes wait in it until the flush.

Symptoms: the focus moves but the screen does not follow (CarPlay or Android Auto stays behind), stale screens, missed transitions.

This package replaces only the `onmessage` handler: it strips the sequence byte exactly as stock does and delivers each message at once, in arrival order. One WebSocket keeps the sender's order. Everything else in the file stays stock.

Version 0.6 only turned `_isStack` / `_delStack` into methods. That stopped the TypeError but did not fix the lost messages.

## What it changes

| File | How | Why |
|---|---|---|
| `/jci/gui/framework/js/Websockets.js` | 3way | the MMUI `onmessage` handler delivers every message in arrival order; the stock reorder queue lost `focusStack` / `transition` messages |

`xml-node` edits address single nodes by XPath; `3way` edits are small diffs against the stock file. The Store merges them with every other installed package, refuses a real clash, and takes them back out on uninstall. No stock file is replaced as a whole.

## Install and uninstall

Install from the Revised Store app on the unit, then restart. Uninstall the same way: after the restart the unit behaves as stock again.

Safety: every change is recorded in the Store's vanilla index, so the boot-loop tripwire reverts it on its own if the unit fails to boot several times in a row. No package edits the boot script above the tripwire line.

## Requires

- `cmu-re-store (>= 0.13.4)`

## Part of

- `cmu-re-speedup-boot`

## Develop

```sh
sh test/run.sh                                  # manifest and payload checks
sh tools/mkpkg.sh <version> <rvp-pack> out.rvp   # build the .rvp
```

Releases: a `dev/vN` tag on `develop` gives a dev prerelease, a `vN` tag on `main` a release; CI packs the `.rvp` and attaches it to the GitHub release.
