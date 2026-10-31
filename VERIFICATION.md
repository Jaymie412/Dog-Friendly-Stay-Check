# Dog-Friendly Stay Check verification

Contract file: `contract.py`, version `dog-friendly-stay-0.1`.

## Local checks completed

- Python syntax compilation succeeded.
- `test_contract.py`: **26 tests passed** against a simulated GenLayer runtime.
- The contract uses the same structure and GenLayer SDK calls (`gl.vm.run_nondet_unsafe`, `gl.vm.Return`, `gl.nondet.exec_prompt`, `gl.nondet.web.render`, `gl.vm.UserError`, `gl.message.sender_address`) as my Vegan Recipe Challenge, which ran end to end on GenLayer Studio with the same `py-genlayer` runner.

The official `genlayer-test` package couldn't be installed in the environment used, so the tests use a small simulated runtime. All AI answers in the tests are authored fixtures. They check the contract's deterministic behaviour, not the AI's judgement.

## What the tests cover

**Settling**
1. A listing that welcomes dogs: SETTLED, `dog_friendly`, fee and limit recorded, the claimant wins, and both scores update.
2. A "no pets" listing: `not_dog_friendly`, the challenger wins.
3. A listing that says nothing about dogs: `not_dog_friendly` (`dogs` = NOT_STATED).
4. The listing is read in the same transaction as the acceptance; nothing is read while the claim is open.

**Consent and roles**
5. Only the named challenger can accept, and only with the exact `claim_hash` (case ignored). The claimant and outsiders are refused.
6. The claim hash covers the challenger, the listing name and the link.
7. Claimant and challenger must differ; https links only; names and addresses are checked.
8. Only the claimant can cancel and only the challenger can decline, both only while the claim is open.

**Binding to the listing**
9. A page for another property closes as UNRELATED: no winner, no score change, no retry.
10. If the AI claims another property's name matches, code overrules it; a partial name doesn't count.

**Bounded retries**
11. An unreadable page: UNREACHABLE, one retry by anyone, then CLOSED_UNAVAILABLE with no scores.
12. A retry that can read the page settles normally.
13. A retry is only possible after UNREACHABLE.

**Exactly-once scoring**
14. A settled claim can't be accepted, retried or declined again, and a direct internal replay is refused.
15. Two claims are each scored once.

**Consensus and AI-output checks**
16. A validator reaching a different label blocks the transaction and leaves the claim unchanged.
17. Validators differing only on the fee or limit details but reaching the same label still agree.
18. A failed audit blocks the transaction.
19. Fabricated quotes, missing quotes, missing or extra facts, extra fields and unknown answers are rejected.
20. Quotes on unquoted answers are cleared; the page can't fake the evidence section markers.
21. Fee and limit are ignored when dogs aren't allowed.
22. A malformed AI reply is asked again.

**Demos**
23. The labels follow the code rules; demos create no claims or scores; every demo listing names the property except demo 6.

## Live Studio evidence

See [DEMO_RESULTS.md](DEMO_RESULTS.md).
