# lib-concurrent-js

JavaScript backend of the `concurrent` step-arrow contract — the
isomorphic counterpart of `lib-concurrent-epoll`: the same task model
(spawn/await/channels/locks/timers) runs on browser and server JS hosts.
`src/index.hnl` implements `Run` as a cooperative scheduler in pure
handl over `js.call`/`js.construct` primitives.

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | the scheduler — mirrors `concurrent-epoll` arrow for arrow where expressible without an OS |
| `src/index.test.hnl` | tests |
| `pkg.hnl` | manifest — deps `concurrent`, `js` (`branch:main`) |
| `scripts/hnl-gate.sh` | `.hnl` gate |
| `workflows/gate.yaml` | CI — shared `handl-gates` campaign |

## Contract

- Channels, mutexes, rwlocks, semaphores, atomics and cancel tokens are
  scheduler state with semantics identical to the epoll backend.
- Pure `.hnl` over the `js` ffi; no Rust crate.

## Verification

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
