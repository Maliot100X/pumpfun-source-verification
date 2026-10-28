# Before / after evidence — verify_evidence.py capture-set gate

Revision tested: b855ede02 (lalalune/pumpfun-source main at audit time)

```
BEFORE (gate from HEAD, unmodified):
  evidence intact, 9 captures  ->  9 captures verified            PASS
  four captures deleted (5)    ->  5 captures verified            PASS  <-- defect: evidence loss invisible
  eight deleted (1 left)       ->  AssertionError (assert count>=4)

AFTER (this fix):
  evidence intact, 9 captures  ->  9 captures verified            PASS
  four captures deleted (5)    ->  AssertionError: capture manifest missing from evidence/:
                                   ['evidence/dependencies/associated_token','evidence/dependencies/donation_relay',
                                    'evidence/original/mayhem','evidence/original/pump_amm']
  python3 scripts/test_tooling.py -> Ran 6 tests ... OK
```

Reproduce:
```bash
cp -a evidence /tmp/evidence.keep
python3 scripts/verify_evidence.py            # 9 captures verified
rm -rf evidence/original/pump_amm
python3 scripts/verify_evidence.py            # AssertionError naming the missing capture
mv /tmp/evidence.keep/pump_amm evidence/original/
```
