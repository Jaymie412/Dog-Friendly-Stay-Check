# Dog-Friendly Stay Check

A GenLayer Intelligent Contract that settles a friendly argument: **is this holiday listing really dog-friendly?**

One person (the claimant) says a holiday listing is dog-friendly and names a friend to challenge them. The challenger accepts the exact claim using its hash, and in that same transaction GenLayer validators each read the listing and answer three questions: are dogs allowed, is there a pet fee, and is there a limit on dogs? Every answer that matters must quote the listing word for word. **Code** checks the quotes, decides `dog_friendly` or `not_dog_friendly`, names the winner and updates both people's scores once.

Example: "Sorry, no pets allowed" on the Harbour Flat listing is `not_dog_friendly`, so the challenger wins.

## Why I built it

"Can we bring the dog?" comes up every time friends plan a trip. Listings often bury the answer, add a pet fee, or allow only one dog. This contract makes a quick, fair call that anyone can check, because every answer points to the exact words in the listing.

## How it stays fair

| Risk | How the contract handles it |
|---|---|
| Someone being challenged without agreeing | Nothing happens until the named challenger accepts the exact claim with `claim_hash`. The challenger can also decline. |
| The claimant editing the listing after the challenge is accepted | The listing is read **in the same transaction** as the acceptance, so there's no gap to change it. |
| A link to a different property | The AI must quote where the page names the property, and **code** checks that the quote contains the listing name. Otherwise the claim closes as **UNRELATED**: no winner, no score change. |
| A claim left open forever | Before acceptance, the claimant can cancel or the challenger can decline. After acceptance it settles at once. An unreadable page can be retried once by anyone, then closes as CLOSED_UNAVAILABLE with no score change. |
| Counting a result twice | One internal method updates scores. It refuses a claim already counted (a flag on the claim, plus `claim_ids` on both scores). |
| The AI making things up | Every quoted answer is checked word for word by code. The AI never outputs the verdict; code does. |
| Instructions hidden in a listing | The listing is treated as data. Validators audit the leader's answers for signs that hidden instructions were followed. |

## How consensus works

Using `gl.vm.run_nondet_unsafe`, the leader fetches the listing and answers five questions (readable, names the property, dogs, fee, limit). Each validator fetches the listing itself, answers every question independently, checks the leader's answers against the same strict rules, and must reach the **same label** (`dog_friendly`, `not_dog_friendly`, `unrelated` or `unreachable`). It then runs an AI audit of the leader's quotes and answers, returning `{"accept": true/false}`. Nothing is stored until all of this passes. A malformed AI reply is asked again, up to 3 times.

## Rules (code)

- `dog_friendly` only if the listing says dogs are allowed (quoted). The fee and the dog limit are recorded as extra details.
- `not_dog_friendly` if the listing says no pets (quoted), "assistance dogs only", or says nothing about dogs. A claim of "dog-friendly" needs the listing to say so.
- Winner: the claimant if `dog_friendly`, the challenger otherwise. Each settled claim adds one win and one loss.

## Lifecycle

| Step | Who | Method | Status after |
|---|---|---|---|
| Make a claim | Claimant | `create_claim(challenger_address, listing_name, listing_url)` | OPEN (`claim_hash`) |
| Accept, and settle at once | Challenger | `accept_challenge(claim_id, expected_claim_hash)` | SETTLED, UNRELATED or UNREACHABLE |
| Or decline | Challenger | `decline_challenge(claim_id)` | DECLINED |
| Or cancel before acceptance | Claimant | `cancel_claim(claim_id)` | CANCELLED |
| Retry an unreadable page | Anyone | `retry_settle(claim_id)` | SETTLED, UNRELATED or CLOSED_UNAVAILABLE |

Read methods: `get_claim_summary`, `get_claim`, `get_score(address)`, `get_counts`, `get_policy`, `get_demo`, `get_demo_result`, `get_demo_summary`. Demo write: `run_demo(case_id)`.

## Quick start in Studio

1. Import `contract.py` into GenLayer Studio and deploy. There are no parameters.
2. Call `run_demo` with `case_id = 1` and wait for it to finish.
3. Call `get_demo_summary` with `result_id = 0`. Expected: label `dog_friendly`, `matches_expected: true`, `records_created: false`.

See [DEMO_GUIDE.md](DEMO_GUIDE.md) for all six demos and the live walkthrough.

## Demonstrated in Studio

See [DEMO_RESULTS.md](DEMO_RESULTS.md) for the recorded run, deployment address and screenshots.

## Testing

`test_contract.py` contains 26 local tests using a simulated GenLayer runtime and authored AI answers. See [VERIFICATION.md](VERIFICATION.md).

## Boundaries

- This checks what a listing **says** about dogs. It can't confirm the host's actual policy, and rules can change after the check.
- Binding proves the page names the property, not who wrote it.
- The demos use fictional listings. Real claims fetch the live page.
- All data is public on-chain.
- Instructions hidden in a listing are treated as data. This reduces prompt injection but doesn't guarantee against it.
- Different addresses don't prove different people.

## Built with

- [GenLayer](https://docs.genlayer.com) Intelligent Contracts (Python, GenVM v0.2.16 runner)
- Developed with help from Claude
