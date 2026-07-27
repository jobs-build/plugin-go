# plugin-go

<p align="center">
  <img src="docs/assets/jobs-logo.jpg" alt="JOBS — Jonas' Own Build System" width="520">
</p>

The JOBS **go-build plugin** (`goplugin`) as a standalone, JOBS-buildable repo.

It is a network-free, statically-linked CBOR-stdio subprocess (build.md §6): it reads
`{call:{go_sum:<bytes>}, source, dir}` on stdin, turns each `go.sum` entry into a `gomod`
import spec, and writes the resulting `[{path, version, input}]` array on stdout.

**Monorepo mode** (sibling-sources design §12.1): when the recipe also passes a
`go_mod` (and/or `go_work`) kwarg, the plugin parses the manifest's `replace`/`use`
directives for RELATIVE targets (`./x`, `../x` — Go honors replaces only in the main
module, so the consumer's manifest enumerates the whole local sibling closure),
resolves them against `dir` (the consumer package's dir within the mounted source),
and responds with `{modules: <the array above>, sources: ["//root/relative", ...]}`
instead. Without the kwarg the bare-array response is byte-identical to before.

**Closure mode** (source-closure design §8): a `go_closure = ["<entry-dir>", ...]`
kwarg (dir-relative entry package dirs, e.g. `["."]`; requires `go_mod`) makes the
plugin walk the transitive local import graph — pure Go, no toolchain: every `.go`
file's imports count, including build-tag-ignored and `_test.go` files — and add a
`closure` key to the monorepo response: the `//`-rooted COMPLETE cover (reached
package dirs as recursive covers, so `//go:embed`/cgo/asm/`testdata` ride along;
module-root packages enumerate files + embed globs + `testdata/`; each involved
module's `go.mod`/`go.sum`; `go.work(.sum)` at the consumer dir). The recipe
forwards it into `build() closure=` for precise KP cutoff — the build dir is not
auto-seeded.

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
