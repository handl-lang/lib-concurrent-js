# lib-concurrent-js

JavaScript backend of the `concurrent` step-arrow contract — the
isomorphic counterpart of `lib-concurrent-epoll`: the same task model
(spawn/await/channels/locks/timers) runs on browser and server JS hosts.

`src/index.hnl` implements `Run` as a cooperative scheduler in pure
handl, mirroring `concurrent-epoll` arrow for arrow wherever the op is
expressible without an OS:

- **Channels, mutexes, rwlocks, semaphores, atomics, cancel tokens** —
  scheduler state, identical semantics to epoll. `MessageChannel`/
  `MessagePort` and the JS `Atomics` namespace are deliberately *not*
  used for these: the contract's channel ops are synchronous
  rendezvous/bounded-queue exchanges resolved inside one scheduler step,
  which async message delivery cannot express.
- **`sleep` / `await_timeout`** — parked in the timer list like epoll;
  when the runq drains the scheduler sleeps until the earliest deadline
  via `Atomics.wait` on a `SharedArrayBuffer`+`Int32Array` (real blocking
  sleep on Node, workers, Deno), falling back to a `Date.now` spin where
  `Atomics.wait` is barred (browser main thread) or unavailable. All
  clock/sleep access goes through `js.call`/`js.construct`, so
  timer-free workloads never touch the JS host.
- **fd I/O, curl, TLS, WebSocket ops** — no synchronous JS equivalent;
  they raise `concurrent-js: <op> unsupported` rather than misbehave.

## Layout

```text
src/index.hnl       the .hnl sources (the Run scheduler)
src/index.test.hnl  contract tests mirroring the epoll suite (minus TCP)
pkg.hnl             package manifest — modules, deps, description, license
```

Dependencies: concurrent, js

## Gate

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

The scheduler core is pure handl, so most tests pass on any host.
`test_sleep` (and any timeout-fires `await_timeout` case) crosses
`js.call` and fails as `call_host` on a host without the `js` natives —
the gate tolerates those by construction, same convention as
lib-json-js.

## License

MIT.
