# Investigation: a part exactly on its upper limit

## Reproduce

- Smallest failing call: `classifyDefect(25.05, 'shaft')`
  - Got: `'pass'`
  - Expected: `'rework'` (spec 4.2 and 4.4: a part exactly on a limit is outside)
- Neighbouring call that works: `classifyDefect(25.06, 'shaft')`
  - Got: `'rework'`, which is also what the spec says. The bug starts at the limit itself, one step before this reading.

## Hypothesis

The cause is the comparison on line 20 of `src/inspection/classifyDefect.js`, `reading > lower && reading <= upper`. The `<=` makes the
upper limit part of the passing range, but spec 4.1 says a pass must be strictly below the upper limit. For a shaft, `upper` is
`25 + 0.05`, which is exactly `25.05` in floating point, so the reading equals `upper`, the check is true, and the function returns `'pass'`.

- **If I am right**, a log of `reading`, `upper` and `reading <= upper` on line 20 shows `25.05 25.05 true` for this call. Changing
  `<=` to `<` alone makes this call return `'rework'`.
- **If I am wrong**, the log shows `upper` slightly different from the reading, for example `25.050000000000001`, which would mean
  floating-point error and not the operator. Or, changing `<=` to `<` leaves the answer as `'pass'`.

## Second input, different part

`classifyDefect(40.02, 'bore')` gives `'pass'` and should be `'rework'`. `40 + 0.02` is exactly `40.02`, so it fails for the same reason: the `<=`.

Two other cases look similar but have a different cause, floating point (spec 4.4):
- Spacer lower limit: `6.35 - 0.07` is `6.279999999999999`, not `6.28`, so `6.28 > lower` is true and the part passes instead of being scrapped.
- Sleeve upper limit: `19.05 + 0.03` is `19.080000000000002`, so `19.08` is below `upper` and passes. Changing `<=` to `<` would not fix this.
