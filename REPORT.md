## Problem

`scripts/verify_evidence.py` — named in the project's `requiredTests` — ended with a **count floor**:

```python
assert count>=4
```

The repository publishes **nine** captures. A floor of four therefore cannot express "the evidence this repository declares is present": up to five captures can disappear while the required gate still reports success.

Reproduced on a sealed copy of `evidence/` at `b855ede02`, using the unmodified gate from `HEAD`:

```
HEAD gate, 9 captures  ->  9 captures verified; no source rebuild claimed.   PASS
HEAD gate, 5 captures  ->  5 captures verified; no source rebuild claimed.   PASS   <- four deleted, gate green
HEAD gate, 1 capture   ->  AssertionError (assert count>=4)                          the only case it catches
```

`evidence/dependencies/associated_token`, `evidence/dependencies/donation_relay`, `evidence/original/mayhem` and `evidence/original/pump_amm` were removed and the required gate still exited 0.

## Requirement served

`approvedScope`: *"Accept bounded improvements to the documented reconstruction, **verification tooling** and security review."* and `requiredTests`: *"Run python3 scripts/verify_evidence.py and python3 scripts/test_tooling.py."*

## Cause

The gate verified the **contents** of each capture it happened to find, but never verified the **set** of captures. A count floor is satisfied by any subset at or above the floor, so deletion of evidence was invisible to the one gate that is supposed to make evidence trustworthy.

## Result

The floor is replaced by set equality against an explicit `EXPECTED_CAPTURES`, with a failure message that names what is missing or unexpected:

```
fixed gate, 5 captures ->
  AssertionError: capture manifest missing from evidence/:
    ['evidence/dependencies/associated_token','evidence/dependencies/donation_relay',
     'evidence/original/mayhem','evidence/original/pump_amm']

fixed gate, 9 captures ->  9 captures verified; no source rebuild claimed.  PASS
python3 scripts/test_tooling.py -> Ran 6 tests ... OK   (file untouched)
```

Both required gates were run on this head. `test_tooling.py` is unchanged and still green.

## Implementation choice, and why not the alternatives

- **Set equality against a literal** (chosen): smallest possible diff to one file, no new dependency, no change to how captures are verified, and the failure names the missing path rather than a number.
- *Derive the expected set by globbing `evidence/`*: rejected — that is the defect restated, since a deleted capture also globs away.
- *Derive it from `repro/status.json` or a manifest*: rejected as the first step — it adds a second source of truth to keep in sync and would change what the gate depends on. Worth considering later if the capture list becomes generated.
- *`assert count==9`*: rejected — it detects the loss but reports only a count, so a maintainer gets `8 != 9` with no indication of which evidence vanished.

## How to test

```bash
cp -a evidence /tmp/evidence.keep          # so the experiment is reversible
python3 scripts/verify_evidence.py         # expect: 9 captures verified
rm -rf evidence/original/pump_amm
python3 scripts/verify_evidence.py         # expect: AssertionError naming evidence/original/pump_amm
mv /tmp/evidence.keep/pump_amm evidence/original/   # restore
python3 scripts/verify_evidence.py         # expect: 9 captures verified
```

## Limits, stated explicitly

- `EXPECTED_CAPTURES` is a literal, so **adding** a capture requires updating it in the same commit; the gate fails loudly (`unexpected capture in evidence/`) rather than passing silently, which is the intended trade.
- The gate still does not verify captures that ship no `capture.json`.
- This is an evidence-integrity defect in verification tooling. It is **not** a vulnerability in any reconstructed program, and it makes no claim about reconstruction completeness or security sign-off.
- Tested at `b855ede02` on a sealed copy of `evidence/`; the experiment never modified this repository's tree.

Made via @slopdotcash
AI provider/model: river / deepseek-ai/DeepSeek-V4.1-Flash
Client / agent tooling: hermes-agent
Contribution skill revision: N/A - ordinary GitHub contribution without optional run evidence
Attribution status: self-reported
— [contributor]
<!-- slop-contribution-attribution:v1 {"provider":"river","model":"deepseek-ai/DeepSeek-V4.1-Flash","client":"hermes-agent","skill_revision":"N/A - ordinary GitHub contribution without optional run evidence"} -->
