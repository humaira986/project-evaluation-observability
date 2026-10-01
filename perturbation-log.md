# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run it, and record what actually happened. See the starters in the Instructions, or design your own (your own experiment earns more credit).

---

### System 1 — validated, routed pipeline

* **Change I made (file + what I changed):** Used the controlled missing-source edge case where the required `endorsements` field is absent.
* **Command I ran:** `.venv/bin/pytest tests/test_us01_retry.py -k test_ac_01_04_missing_source_halts_immediately -v -s`
* **What I predicted:** A missing required field would trigger `RetryFutileEscalation` and stop immediately without another API call.
* **What actually happened (paste the key output line):** `test_ac_01_04_missing_source_halts_immediately PASSED`; the test asserts `RetryFutileEscalation`, field `endorsements`, detected pattern `endorsements_absent`, and `client.call_count == 1`.
* **How this differs from the unperturbed run:** The missing required field was treated as a futile retry condition and escalated immediately, with exactly one recorded API call, rather than continuing normal extraction.

---

### System 2 — schema-enforced two-pass extraction

* **Change I made (file + what I changed):** Used the `income_sum_mismatch.txt` fixture containing a deliberately inconsistent stated monthly income total.
* **Command I ran:** `.venv/bin/mortgage-extract fixtures/documents/income_sum_mismatch.txt --mode replay`
* **What I predicted:** The mathematical consistency validator would detect that the stated total did not equal the sum of the extracted components.
* **What actually happened (paste the key output line):** `consistent: false` with discrepancy in `total_monthly_income` and delta `-1250`.
* **How this differs from the unperturbed run:** The normal consistent documents were accepted as consistent, while this mismatched input was flagged with a discrepancy instead of being accepted.

---

### System 3 — multi-source synthesis

* **Change I made (file + what I changed):** Simulated a timeout in the logistics source using the coordinator's `--simulate-timeout` option.
* **Command I ran:** `.venv/bin/supply-chain-investigate meridian --offline --simulate-timeout`
* **What I predicted:** The logistics source would become unavailable, and metrics that depended only on that source would be marked incomplete while results from other sources would still be retained.
* **What actually happened (paste the key output line):** `Sources unavailable: logistics unavailable (timeout)` and `late_shipment_count [missing source: timeout reading logistics]`
* **How this differs from the unperturbed run:** In the unperturbed run, logistics data was available and `late_shipment_count` was reported as 11. After the simulated timeout, `late_shipment_count` became incomplete and the surviving sources continued to produce results.