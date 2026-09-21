# vira

`vira` is an H# package manager for the same `Bytes.hk` / `bytes.hk`
projects a `bytes` binary already builds — every command, flag, and
manifest key works exactly the same way. Written from scratch in H#,
it is not a fork or a copy of `bytes`' own source.

## The one real difference from `bytes`

`bytes` downloads every dependency into the current project's own
`build/cache/packages/` — a fresh copy per project, deleted the moment
`build/` is cleaned.

`vira` fetches into one shared, global store instead:

```
~/.hsharp/bytes-io/
  packages/<name>-<slot>/   one directory per (name, source) pair,
                            reused by every project that depends on
                            that exact same source
  pyenv/                    one shared Python virtualenv
```

Everything a project *builds* — compiled binaries, the JIT/bytecode
cache, the registry index cache — still lives exactly where `bytes`
puts it, under that project's own `build/` and `build/cache/`. Only
the libraries themselves move out of `workspace/build` and into the
shared store, so ten projects pinning the same dependency fetch it
once, not ten times, and `vira clean --everything` never has to
re-download anything the next build still needs.

## Usage

```
vira new <name> [--lib]
vira build [--release] [-v]
vira run   [--release] [-- args]
vira add   <pkg> [version]
vira install
vira workspace info|build|list
vira cache
```

Run `vira --help` for the full command list — it is the same list a
`bytes --help` prints, command for command.

## Building vira itself

```
h# compile src/main.h# -o build/vira --release
```

or, once you have any working `bytes`/`vira` binary already:

```
bytes build --release   # or: vira build --release
```

## Layout

```
Vira.hk           — vira's own build manifest
src/main.h#        — entry point, argument parsing
src/shell.h#        — every `vira <command>` implementation
src/manifest.h#     — Bytes.hk / bytes.hk parsing
src/fetcher.h#      — dependency resolution (git/version/newest/path/registry)
src/store.h#        — the ~/.hsharp/bytes-io/ global library store
src/registry.h#     — the shared package index
src/lockfile.h#     — vira.lock
src/workspace.h#    — multi-member / multi-language workspace builds
src/fmt.h#          — `.h#` formatter
src/tester.h#       — `#[test]` runner
src/docs.h#         — HTML doc generator
src/prefs.h#        — persisted settings (progress-bar theme, backend)
src/pybridge.h#     — Python package installs
src/ui.h#           — progress bars / spinners / build summaries
```
