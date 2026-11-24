# GUI WebSocket fix

`cmu-re-gui-ws-fix` · Revised Store package · Mazda CMU150, firmware 74.00.324A

The stock HMI message handler calls _isStack and _delStack as methods, but they are plain functions, so every call throws a TypeError into the log. This makes them methods of Websockets.

## Why

In stock `Websockets.js`, the message handler calls `_isStack` and `_delStack` as methods of the Websockets object, but they are defined as plain functions. Every such call throws a TypeError into the log. This is a correctness fix: on the bench it gave no measurable boot gain.

## What it changes

| File | How | Why |
|---|---|---|
| `/jci/gui/framework/js/Websockets.js` | 3way | _isStack and _delStack become Websockets methods, so the calls stop throwing a TypeError into the log |

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
