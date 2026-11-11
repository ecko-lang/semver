# semver

[Semantic Versioning 2.0.0](https://semver.org) in pure Ecko: parse, compare,
sort and match version ranges. Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/semver
```

```ecko
import semver
```

## Usage

```ecko
import semver

print(semver.valid("1.2.3"))  # true
print(semver.valid("01.1.1"))  # false - leading zero

print(semver.compare("1.0.0-alpha", "1.0.0"))  # -1, a pre-release comes first
print(semver.sort(["2.0.0", "1.0.0-rc.1", "1.0.0"]))  # [1.0.0-rc.1, 1.0.0, 2.0.0]

print(semver.satisfies("1.9.3", "^1.2.0"))  # true
tags = ["1.0.0", "1.2.0", "1.9.3", "2.0.0", "latest"]
print(semver.max_satisfying(tags, "^1.2.0"))  # 1.9.3
print(semver.inc("1.2.3", "minor"))  # 1.3.0
```

## API

### Parsing

| function | returns |
|---|---|
| `parse(text)` | `{ major, minor, patch, prerelease, build }` |
| `valid(text)` | `true` when `text` is a valid version, otherwise `false` |
| `string_of(p)` | a parsed version rendered back to text, build metadata included |

### Precedence

| function | returns |
|---|---|
| `compare(a, b)` | `-1`, `0` or `1`; accepts text or parsed versions |
| `eq(a, b)` | whether `a` and `b` have equal precedence |
| `gt(a, b)` | whether `a` is higher than `b` |
| `lt(a, b)` | whether `a` is lower than `b` |
| `sort(versions)` | the versions ascending by precedence, stable |
| `inc(text, part)` | the next version for `"major"`, `"minor"` or `"patch"` |

### Ranges

| function | returns |
|---|---|
| `satisfies(version, range)` | whether `version` falls inside `range` |
| `max_satisfying(versions, range)` | the highest match, or `null`; non-versions are skipped |
| `min_satisfying(versions, range)` | the lowest match, or `null`; non-versions are skipped |
| `parse_range(text)` | the alternatives, each a list of `{ op, version }` comparators |

## Notes

### Errors

| kind | reason | when |
|---|---|---|
| `semver` | `bad_version` | text is not a valid version |
| `semver` | `out_of_range` | a number, or a bump of one, does not fit in a 64-bit integer |
| `semver` | `bad_range` | a range or one of its `\|\|` alternatives is empty, or an operator has no version |
| `semver` | `unsupported_range` | a hyphen range (`1.2.3 - 2.0.0`) or a partial or x-range version (`~1.2`, `^1`, `1.x`) |
| `bug` | `bad_argument` | an argument has the wrong type, or `inc` is given a part other than major, minor or patch |

### Range syntax

The npm grammar, because it is what people already have in their fingers.
Every version in a range is a full `major.minor.patch`; the partial forms and
hyphen ranges are refused with `unsupported_range` rather than read as
something they do not mean.

| form | means |
|---|---|
| `1.2.3` | exactly that version |
| `^1.2.3` | no change to the leftmost non-zero number: `>=1.2.3 <2.0.0` |
| `^0.2.3` | for `0.x` the minor acts as the major: `>=0.2.3 <0.3.0` |
| `~1.2.3` | patch-level changes only: `>=1.2.3 <1.3.0` |
| `>=1.2.3 <2.0.0` | an AND of comparators; `>= 1.2.3` with a space is the same |
| `^1.0.0 \|\| ^3.0.0` | an OR of alternatives |
| `*` | any release |

### What the spec makes awkward

Leading zeroes are rejected in the numeric fields and in numeric pre-release
identifiers, but allowed in build metadata. Build metadata is excluded from
precedence entirely, so `1.0.0+build.1` and `1.0.0+build.999` compare equal. A
numeric pre-release identifier always ranks below an alphanumeric one, and
`1.0.0-beta.2` precedes `1.0.0-beta.11` because those identifiers compare as
numbers rather than as text.

**A pre-release never satisfies a range that did not ask for one.** `^1.0.0`
will not match `2.0.0-alpha`, and will not match `1.5.0-rc.1` either. A
pre-release only matches when a comparator names a pre-release on the same
`major.minor.patch`.

**Bumping a pre-release lands on its release** when the parts below the bumped
one are zero, as npm does: `inc("1.2.0-rc.1", "minor")` is `1.2.0`,
`inc("2.0.0-rc.1", "major")` is `2.0.0`, and `inc("1.2.3-rc.1", "patch")` is
`1.2.3`.

**`max_satisfying` and `min_satisfying` skip entries that are not versions**,
so a tag list holding `latest` still resolves. A leading `v` is not part of a
version, so `v1.2.0` is skipped too; strip it first if your tags carry one.

## Testing

```bash
ecko test
```

Offline and deterministic. The precedence cases come from the specification
itself, including the worked ordering example in §11.4 and the valid/invalid
lists the spec publishes.

## License

[MIT](LICENSE)
