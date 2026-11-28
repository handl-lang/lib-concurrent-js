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
| `pkg.hnl` | manifest — dep `concurrent` only (`js` is a host/builtin contract, NOT a dep — see Contract) |
| `scripts/hnl-gate.sh` | `.hnl` gate |
| `workflows/gate.yaml` | CI — shared `handl-gates` campaign |

## Contract

- Channels, mutexes, rwlocks, semaphores, atomics and cancel tokens are
  scheduler state with semantics identical to the epoll backend.
- Pure `.hnl` over the `js` ffi; no Rust crate.
- `js` stays out of `pkg.hnl` `deps` and `.handl.toml`
  `[dependencies]`: the `js` contract is a host/builtin surface
  (compile-js binds `from "js"` to `__handl_rt` shims regardless of
  dep tables). Declaring the dep makes `hnl run`/`hnl test` resolve
  `js` to `lib-js`'s signature-only exports — `call` erases to `null`
  and every call dies with "callee is not an arrow-value". Undeclared,
  the import lowers to the ambient `"js.call"` callee and fails
  honestly as `call_host: function 'js.call' not found` on hosts
  without JS natives (the shape `scripts/hnl-gate.sh` tolerates).

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
