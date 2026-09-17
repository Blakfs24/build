# SingTools build orchestrator

This repository contains only GitHub Actions workflows. It checks out an exact
revision of the private SingTools source repository and delegates all native and
application packaging to that revision's `tool/native_build.dart` and
`setup.dart` entrypoints.

The Go toolchain version is read from the checked-out source's
`libcore/go.mod`; there is no second dependency lock file in this repository.

- `build.yaml` handles `build-trigger` dispatches and replaces the public
  `pre-release` assets.
- `release.yml` handles `release-trigger` dispatches and publishes the matching
  source tag.
- `native.yml` is a manual manifest-only diagnostic build.
- `package.yml` is the shared private-source packaging matrix.

Required repository secrets are `PRIVATE_REPO` and `ACCESS_TOKEN`. Android
release publication also requires `KEYSTORE`, `KEY_ALIAS`, `STORE_PASSWORD`,
and `KEY_PASSWORD`.
