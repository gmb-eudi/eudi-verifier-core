# Changelog

Notable changes, newest first. Releases are git tags; this file says what each tag contains.

## v0.1.1

A security release. **The service now builds with Go 1.27.2**, and its libraries move to their Go 1.27.2
releases. The OID4VP surface, the configuration and the database are unchanged; one timing behaviour of the
status-list fetch changes with the framework (see *Changed*).

### Changed

- **Go 1.27.2** (was 1.27.0), both in `go.mod` and as the image's default build toolchain
  (`ARG GO_VERSION` in the `Dockerfile`). Go 1.27.2 fixes standard-library and `golang.org/x/net`
  vulnerabilities that this service's code reaches under 1.27.0: `govulncheck` found nine it calls,
  GO-2026-6603, -6605, -6607, -6608, -6610, -6611, -6612, -6613 and -6617, in `net/http`,
  `crypto/tls`, `net/textproto`, `mime/multipart` and `golang.org/x/net`. Under the new set it finds
  none. **Rebuild the image to pick the fixes up**; an image built before this change still carries
  them.
- **Libraries:** `go-oid4vp` → v0.0.9, `go-dcql` → v0.0.3, `go-eudi-crypto` → v0.0.9,
  `go-eudi-rpcert` → v0.0.7, `go-eudi-trust` → v0.1.3, `go-mdoc` → v0.1.3, `go-sdjwt` → v0.0.9,
  `go-statuslist` → v0.1.6, `go-verifier-helpers` → v0.0.5, `go-platform-kit` → v1.11.4, azugo →
  v0.40.0, OpenTelemetry → v1.47.0, `fasthttp` → v1.75.0, `go-redis` → v9.23.0,
  `VictoriaMetrics/metrics` → v1.44.1, `miniredis` (tests only) → v2.40.0, plus 37 indirect modules.
- **A status-list fetch now stops at the wallet response's deadline.** From azugo v0.40.0 the
  outbound client enforces the deadline of the request it was made for, where before a slow
  status-list host ran to completion and the deadline was only noticed afterwards. A fetch cut off
  this way fails like any unreachable status list. Nothing else in azugo v0.40.0 reaches this
  service: it sets no cookies, issues no redirects and implements no azugo cache.

### Notes

- `LICENSE` drops the boilerplate header that preceded the licence text (merged from the published
  repository). The licence itself is unchanged: EUPL-1.2.
- The image build now caches Go modules and compiled packages between builds. The binary is built by
  the same command with the same flags; only the build's layer order and caches changed.
- CI's linter moved to golangci-lint v2.14.0. The earlier release cannot read Go 1.27.2's compiled
  standard library and stops before linting anything.
- The gate is green on Go 1.27.2, with the `testhelpers` build tag as CI uses it: `go mod verify`,
  `go mod tidy -diff`, build, vet, `gofmt`, golangci-lint v2.14.0, `go test -race` with **0 races**,
  and the fuzz target for 30 seconds with no failing input; `govulncheck` reports **no
  vulnerabilities found**.

## v0.1.0 — initial public release

**The wallet-facing OpenID4VP verifier: it serves the signed request, receives the wallet's response, and runs the ten-step verification pipeline that produces the verdict.**

It speaks HTTPS in-process, because OpenID4VP requires a real `https://` request URI. It verifies
mdoc/mDL (ISO/IEC 18013-5) and SD-JWT VC credentials: structural parse, issuer authenticity against a
trust anchor, digest integrity, revocation, device binding, User binding, query fulfilment. The
verified result is handed on encrypted, and the attribute values are deleted here — this service never
becomes the place where wallet attributes accumulate.

`SPECREFS.md` states which edition of each specification the clause numbers in the source refer to.

### Running it

The image entrypoint is `["/server", "web"]`, with `["/server", "health"]` as the healthcheck
command — a compose or orchestrator file that also passes `command: ["web"]` would append a second
argument. Every setting is an environment variable with a safe default where one is possible; the
README lists them, and any secret can be supplied as `<NAME>_FILE` pointing at a mounted file instead.

### What it needs

Go **1.27.0** to build. Libraries at this release:

`go-oid4vp` v0.0.8 · `go-mdoc` v0.1.2 · `go-sdjwt` v0.0.8 · `go-dcql` v0.0.2 ·
`go-statuslist` v0.1.5 · `go-eudi-crypto` v0.0.8 · `go-eudi-rpcert` v0.0.6 · `go-eudi-trust` v0.1.2 ·
`go-verifier-helpers` v0.0.4 · `go-platform-kit` v1.11.3 · `azugo.io/azugo` + `azugo.io/core` v0.38.1
