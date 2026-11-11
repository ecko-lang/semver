# Changelog

## 0.38.0

### Fixed

- A hyphen range such as `1.2.3 - 2.0.0` used to return `false` silently; it is now refused with `semver`/`unsupported_range`, as are partial and x-range versions (`~1.2`, `^1`, `1.x`), which used to fail with a misleading message.
- An operator separated from its version by a space (`>= 1.2.3`) now works instead of failing with "got ''".
- `inc` on a pre-release now lands on the release it leads to for every part: `inc("1.2.0-rc.1", "minor")` is `1.2.0` and `inc("2.0.0-rc.1", "major")` is `2.0.0` (were `1.3.0` and `3.0.0`).
- A version number too large for a 64-bit integer, in `parse` or as the result of `inc` or a `^`/`~` bound, raises `semver`/`out_of_range` instead of a `parse` or `bug` error.
- `max_satisfying` and `min_satisfying` skip entries that are not versions (a `latest` tag) instead of raising.
- `parse_range` now checks every version in the range when it is parsed, not only when it is matched.
- Every error carries a `reason` (`bad_version`, `out_of_range`, `bad_range`, `unsupported_range`) and a message naming the function and the value.

### Breaking

- `identifiers`, `identifier` and `comparators` are no longer exported; they were internal helpers. Migration: use `parse(text).prerelease` / `.build` and `parse_range(text)` instead.
- An argument of the wrong type (a non-string version, a non-list to `sort`/`max_satisfying`, a non-map to `string_of`) and an unknown `inc` part now raise kind `bug` with reason `bad_argument`; a non-string `parse` argument used to raise kind `semver`. Migration: catch on `semver` only for bad input text, and fix the call for `bug`.
