# SingTools build orchestrator

This repository contains only GitHub Actions workflows. It checks out an exact
revision of the private SingTools source repository and delegates all native and
application packaging to that revision's `tool/native_build.dart` and
`setup.dart` entrypoints.

Before packaging, `test.yml` resolves the requested branch, tag, or commit to an
immutable source SHA. All test jobs and subsequent package/native build jobs
check out that SHA, even if the requested branch moves while tests are running.
Both release workflows record this resolved SHA in the release body.

The Go toolchain version is read from the checked-out source's
`libcore/go.mod`; there is no second dependency lock file in this repository.

- `build.yaml` handles `build-trigger` dispatches and replaces the public
  `pre-release` assets.
- `release.yml` handles `release-trigger` dispatches and publishes the matching
  source tag.
- `test.yml` is a reusable test gate and can also be started manually. It runs
  Flutter behavior tests and the CMake manifest contracts on Linux, plus native
  Go tests with race detection on Linux, macOS, and Windows. Coverage reports are
  uploaded separately from package artifacts. Go feature tags come from the
  selected source's `libcore/Makefile`.
- `native.yml` is a manual manifest-only diagnostic build, gated by the same
  test workflow.
- `package.yml` is the shared private-source packaging matrix. It waits for every
  test job to pass before starting the build matrix; a failure prevents packaging
  and the dependent release publication.

The selected source revision must include the test suites, including
`test/native/manifest_contract_test.py`. Older source revisions lacking that
contract are rejected instead of silently skipping the manifest checks. These
jobs do not replace device tests or installed-package/FFI smoke tests. The native
contract suite uses the source's default Go feature tags without building the
external subconverter library.

Required repository secrets are `PRIVATE_REPO` and `ACCESS_TOKEN`. Android
release publication also requires `KEYSTORE`, `KEY_ALIAS`, `STORE_PASSWORD`,
and `KEY_PASSWORD`.
