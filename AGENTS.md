# ls-builder

A Go library for building Language Server Protocol servers: LSP 3.17 types and request
dispatch, and stdio, TCP and WebSocket transports. bluelink's `tools/blueprint-ls` is built on
it, and as a public module on pkg.go.dev it may have users outside NewStack.

## Workspace

This repo is one of NewStack's. NewStack's maintainers develop it alongside the others in a
shared workspace, using the NewStack Claude Code plugin for how work is planned, reviewed,
committed and coordinated across repos. The plugin isn't public, and nothing here depends on it:
this file and the repo's own docs cover what a contribution needs.

## Layout

| Path | Holds |
|---|---|
| `lsp_3_17/` | LSP 3.17 messages and structures; `Handler`, which dispatches client messages to the functions a server sets; `Dispatcher`, which sends server-to-client requests and notifications; `TraceService` |
| `server/` | The server: connections, transports (stdio, TCP, WebSocket), the jsonrpc2 logger adapter |
| `common/` | `LSPContext` and the handler types both use |

Each directory's `README.md` describes it.

## Checks

- **Tools:** Go (`go.mod`; CI uses 1.24) with cgo for `-race`, `staticcheck` on `PATH` (CI
  installs `@latest`), and Node with Yarn 1: run `yarn` in each checkout or worktree before
  committing, since the commit-msg hook runs commitlint through `npm exec`.
- **Lint:** `scripts/lint.sh` (`staticcheck` and `go vet` per package).
- **Tests:** `bash scripts/run-tests.sh` runs every package with `-race` and coverage. Tests
  need no services; the transport tests listen on free local ports. `--update-snapshots` does
  nothing. `lsp_3_17/example_test.go` checks its log output, and the root `README.md` example
  is copied from it by hand.
- **JSON shapes:** `encoding/json` matches field names case-insensitively when decoding, so a
  wrong tag can still decode. Check a new or changed type by marshalling it against the spec.
- **Gates:** with `git config core.hooksPath .githooks`, pre-commit runs `scripts/lint.sh` over
  the working tree on every commit, and commit-msg runs commitlint. CI (`ci.yml`) runs lint,
  tests and SonarCloud on pull requests and pushes to `main`.

## Compatibility

- **Go API:** blueprint-ls and any outside users build against tagged releases. Breaking
  changes have shipped as patch releases so far (v0.2.1, v0.2.2, v0.2.4, per `CHANGELOG.md`);
  a breaking change says in its pull request what blueprint-ls needs.
- **Protocol:** types aim to match the LSP 3.17 specification's field names and JSON shapes,
  which clients such as VS Code send and expect; new or changed types use the spec's names.
  Known deviations: `ServerCapabilities.NotebookDocumentSync` marshals as `notebookSync`,
  `TextDocumentClientCapabilities.Diagnostics` decodes `diagnostics` rather than `diagnostic`,
  and `DocumentRangeFormattingParams.Range` marshals as `Range`. `window/showDocument` isn't
  implemented.
- **Persisted state:** none.

## Conventions

- Commits: conventional commits, checked by commitlint (`commitlint.config.js`). Scopes are
  allowed but unused.
- Releases are manual: a `CHANGELOG.md` entry on `main`, then a `vX.Y.Z` tag and GitHub
  release, as [CONTRIBUTING.md](CONTRIBUTING.md) sets out. There is no release automation.

## Safety

- Don't create or push tags, create releases, or edit `CHANGELOG.md` for a release unless
  asked.

## Known mistakes

<!-- One line per mistake an agent has made here, phrased as the rule that prevents it. -->
