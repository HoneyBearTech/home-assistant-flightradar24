# Plan: fr24-fixes-1

## What Sean asked for
"Start the next repo on the shortlist" (AlexandrErohin/home-assistant-flightradar24), respecting the project's contribution rules: fix real bugs and send each fix to the original project as its own pull request.

## Contribution rules
- **Written rules:** there is no CONTRIBUTING.md, pull request template or code of conduct, in this repository or at the owner's account level. The project's checks set the rules: a flake8 style check (lines up to 120 characters), the Python tests on Python 3.14, the dashboard card tests on Node 22, and Home Assistant's own checks (hassfest, HACS).
- **What the owner does in practice:**
  - He merges outside fixes within days, usually squashed into one commit.
  - He writes the changelog and version himself, so contributors don't touch those files.
  - Every recent fix came with tests.
  - Accepted pull requests use a plain sentence title, a `fix/<slug>` branch, and a description covering problem, change, tests and `Fixes #N`.
- **What we'll follow:** all of the above. Nothing touches CHANGELOG, the version, workflows or translations.

## What we found
- **None of the three earlier candidates is a bug on today's code:**
  - #355 is a library upgrade the integration doesn't need.
  - #274 is already fixed.
  - #268 is missing data on Flightradar24's side.
- **Two real bugs were confirmed instead,** described below.

## What we'll change
1. **#151 Tracked flights shown in the wrong place:** when a flight is added to tracking, or found again after landing, its position comes from Flightradar24's search result. That result is rounded to 0.1°, so the plane appears several kilometres away and its drawn track gets a spike. Use the exact position from the flight's own trail instead, and keep the search position only when there is no trail.
   - **File:** `custom_components/flightradar24/api/flight.py`
   - **Check:**
     - a new test with a real rounded search answer, which fails on the current code;
     - the existing path for when details aren't available stays unchanged and tested;
     - at most a couple of live requests to confirm it on a real flight.
2. **Airport input keeps a rejected code (no issue yet):** if you type an airport code that Flightradar24 doesn't know, or type one while scanning is off, the box keeps showing it even though nothing changed. After a restart it's restored and retried every 90 seconds, logging an error each time. Show the airport that is actually tracked instead.
   - **Files:** `custom_components/flightradar24/text.py`, and `coordinator.py` if needed.
   - **Check:** new tests for an unknown code and for scanning off. Existing tests keep passing.
3. **For both:**
   - All the project's checks pass locally: flake8, the Python tests and the card tests.
   - Each fix is one commit that applies on its own to the latest `main`.
4. **Send upstream, only after Sean's final approval on this PR:**
   - Each fix gets its own clean branch from the original project's latest `main`, holding only that fix. The plan file never goes upstream.
   - Sean opens one pull request per fix to AlexandrErohin/home-assistant-flightradar24 from his own account. The full title and description of each are shown here first.
   - Draft titles:
     - "Keep the exact position when a tracked flight is found by search"
     - "Show the tracked airport in the airport input, not a rejected code"

## Who's on it
Researcher (done), Engineer, QA, Security Officer, Auditor.

## Risks
- **Busy file:** `api/flight.py` is the owner's most-changed file, last changed today. The fix stays small, and the branch is rebased onto the latest code right before sending.
- **Live check is partial:** the landing case for #151 wasn't watched live. The "add to track" case is proven with a real search answer.
- **Owner follow-ups:** the owner sometimes adjusts merged pull requests with his own follow-up commits.

## Out of scope
- #355 (library upgrade), #274 and #268 (not bugs on today's code).
- #317, #276, #254, #179 (owner actively working on them, or architectural).
- Login-related issues (need a Flightradar24 account).
- Feature requests.
