# Changelog — `armature-config`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [0.4.0] - 2026-08-05

### Changed

- **Requires `armature-core` 0.9 (breaking).** The requirement moved `0.8` →
  `0.9`. `armature-core 0.9.0` itself moves `armature-h1` across a breaking
  0.x boundary; because `armature-core` types appear in this crate's own
  public API, the requirement change is breaking here too and the minor moves
  with it. Under Cargo's 0.x caret rules the 0.8 and 0.9 types are distinct
  and do not unify, so a consumer holding an `armature-core 0.8` type cannot
  pass it to this crate. Part of the `armature-core 0.9.0` release train; see
  `armature-core`'s CHANGELOG for the publish order.

## [0.3.1] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.

## [0.3.0] - 2026-08-04

### Fixed

- **Behaviour:** a configuration document whose top level is not an object is
  now rejected instead of silently applying nothing. JSON is the only supported
  format that can parse to something else -- a TOML document is a table and a
  `.env` file is a set of assignments -- so `99`, `[1, 2]` or `"text"` parsed
  happily and carried no keys. `ConfigManager::load_file` then matched only the
  object case and fell through to `Ok(())`, so a file rendered to a bare scalar
  (the usual shape of a templating mistake) started the service on defaults
  with no error to explain it. Both the loader and `load_file` now report it.

- **Breaking:** `__` in an environment variable maps to `.`, so an env var can finally satisfy the dotted lookups the docs demonstrate.
- **Breaking:** `FileFormat::Env` delegates to `dotenvy`. The hand-rolled parser disagreed with the crate's other `.env` path on the same file — no `export`, no escapes, no inline comments — and stripped every quote rather than one matched pair.
- Source precedence is documented and pinned by a test; `in_range`/`one_of` interpolate the bounds they rejected against.

### Fixed

- Environment variables map `__` to the path separator `.` (`APP__DATABASE__URL` → `database.url`). Keys were only lowercased, so no environment variable could satisfy the dotted lookups the docs demonstrate.
- `FileFormat::Env` parses through `dotenvy`, the same parser `load_dotenv` uses. The hand-rolled parser it replaces understood neither `export` prefixes, escapes, inline comments nor multi-line values, and stripped every quote rather than one matched pair — so the two paths disagreed on the same file.
- `ConfigValidator::in_range` and `one_of` interpolate the bounds and the offending value; the messages read literally "must be between min and max". Both gained a `T: Display` bound.

### Changed

- `ConfigManager::has` and dotted-path `get` no longer deep-clone. Traversal was cloning the entire subtree at every level of a path, and `has` cloned a value only to discard it.
- Source precedence (dotenv → env → files, later wins, so a **file overrides an environment variable**) is documented on `ConfigServiceBuilder` and pinned by a test. This is the inverse of 12-factor; it is documented rather than reordered because reversing it would silently change what existing deployments resolve.
