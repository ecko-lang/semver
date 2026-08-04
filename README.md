# semver - Ecko Std Lib Package

[Semantic Versioning 2.0.0](https://semver.org) in pure Ecko: parse, compare,
sort and match version ranges.

Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/semver
```

```ecko
import semver
```

## Usage

```ecko
semver.valid("1.2.3")                        # true
semver.valid("01.1.1")                       # false - leading zero

semver.compare("1.0.0-alpha", "1.0.0")       # -1, a pre-release comes first
semver.sort(["2.0.0", "1.0.0-rc.1", "1.0.0"])
# ["1.0.0-rc.1", "1.0.0", "2.0.0"]

semver.satisfies("1.9.3", "^1.2.0")          # true
semver.max_satisfying(tags, "^1.2.0")        # the newest match, or null
semver.inc("1.2.3", "minor")                 # "1.3.0"
```

## API

### Parsing

| function | what it does |
|---|---|
| `parse(text)` | `{ major, minor, patch, prerelease, build }`, or raises `{ kind: "semver" }` |
| `valid(text)` | the total form of `parse` |
| `string_of(parsed)` | render a parsed version back to text |

### Precedence

| function | what it does |
|---|---|
| `compare(a, b)` | `-1`, `0` or `1`; accepts text or parsed versions |
| `eq(a, b)`, `lt(a, b)`, `gt(a, b)` | the comparisons by name |
| `sort(versions)` | ascending by precedence, stable |
| `inc(text, part)` | bump `"major"`, `"minor"` or `"patch"` |

### Ranges

| function | what it does |
|---|---|
| `satisfies(version, range)` | does this version match the range |
| `max_satisfying(versions, range)` | the highest match, or `null` |
| `min_satisfying(versions, range)` | the lowest match, or `null` |
| `parse_range(range)` | the expanded comparators, for inspection |

Range syntax follows the npm grammar, because it is what people already have in
their fingers:

| form | means |
|---|---|
| `1.2.3` | exactly that version |
| `^1.2.3` | no change to the leftmost non-zero number: `>=1.2.3 <2.0.0` |
| `^0.2.3` | for `0.x` the minor acts as the major: `>=0.2.3 <0.3.0` |
| `~1.2.3` | patch-level changes only: `>=1.2.3 <1.3.0` |
| `>=1.2.3 <2.0.0` | an AND of comparators |
| `^1.0.0 \|\| ^3.0.0` | an OR of alternatives |
| `*` | any release |

## Notes

**The awkward parts of the spec are implemented, not approximated.** Leading
zeroes are rejected in the numeric fields and in numeric pre-release
identifiers, but allowed in build metadata. Build metadata is excluded from
precedence entirely, so `1.0.0+build.1` and `1.0.0+build.999` compare equal. A
numeric pre-release identifier always ranks below an alphanumeric one, and
`1.0.0-beta.2` precedes `1.0.0-beta.11` because those identifiers compare as
numbers rather than as text.

**A pre-release never satisfies a range that did not ask for one.** `^1.0.0`
will not match `2.0.0-alpha`, and will not match `1.5.0-rc.1` either. A
pre-release only matches when a comparator names a pre-release on the same
`major.minor.patch`. This is deliberate: the alternative is the behaviour every
other package ecosystem ends up apologising for.

**`sort` is `sort_with(versions, compare)`.** It cannot be `sort_by`, which takes
a key function: semver precedence is not expressible as a key, because a
pre-release list mixes numeric and alphanumeric identifiers whose ordering
depends on their types. This package is why `sort_with` exists.

## Testing

```bash
ecko test
```

Offline and deterministic. The precedence cases come from the specification
itself, including the worked ordering example in §11.4 and the valid/invalid
lists the spec publishes.

## License

MIT
