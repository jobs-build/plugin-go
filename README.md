# plugin-go

The JOBS **go-build plugin** (`goplugin`) as a standalone, JOBS-buildable repo.

It is a network-free, statically-linked CBOR-stdio subprocess (build.md §6): it reads
`{call:{go_sum:<bytes>}, source}` on stdin, turns each `go.sum` entry into a `gomod`
import spec, and writes the resulting `[{path, version, input}]` array on stdout.

This repo is consumed by the JOBS fetcher manifest (`fetchers.toml`, entry `goplugin`):
JOBS fetches a pinned tarball of this repo and builds it with `BUILD.jobs` — fully
offline, using only the Go toolchain and the seeded shell (deps are vendored, so no
`gomod`/`goplugin` are needed, avoiding the obvious circularity). The build output is
`{ fetch, plugin }`, which is then promoted to `fetcher:goplugin:<platform>`.

`internal/importdef` is a verbatim copy of the JOBS `importdef` package (the import
definition's canonical-CBOR contract); it is vendored here so the plugin builds
standalone.

## Build it

```
jobs run --source .     # or: jobs develop --source .
```
