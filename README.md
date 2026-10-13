# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->
FitFindr helps users search for thrift clothing based on a description, size, and maximum price. It searches available listings and selects a matching item. It then uses the user's wardrobe to suggest outfits that pair with the selected item and creates a short fit-card caption. If no listings match the search, FitFindr stops and suggests ways the user can adjust their search.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** 
Searches the listings data for items matching the user's description and optionally filters the results by size and maximum price.

- **Inputs:** <!-- name and type each: `max_price` (float), not "a price" -->
`description` (str), `size` (str | None), `max_price` (float | None). Size matching is case-insensitive and should match the size component of a listing without confusing unrelated characters in other size formats.

- **Returns:**
 list of matching listing dicts, best match first, up to the configured search result limit. Each dict contains `id`, `title`, `description`, `category`, `style_tags` (list), `size`, `condition`, `price` (float), `colors` (list), `brand` (str or None), and `platform`.

- **When it has nothing:**
Returns an empty list `[]` when no listings match.

### `suggest_outfit`

- **What it does:**
Suggests one or two outfits for the selected thrift listing using items from the user's wardrobe when available.

- **Inputs:**
`new_item` (dict), `wardrobe` (dict containing an `items` list).

- **Returns:**
A non-empty string containing one or two outfit suggestions; when wardrobe items are available, the suggestions name specific pieces the user already owns.

- **When it has nothing:**
If `wardrobe["items"]` is empty, returns a non-empty string containing general styling advice for the new item instead of failing or returning an empty string.

### `create_fit_card`

- **What it does:**
Creates a short, post-style caption describing the selected thrift item and suggested outfit.
- **Inputs:**
 `outfit` (str), `new_item` (dict).
- **Returns:**
A two-to-four sentence caption that mentions the item, its price, and its platform once each and describes the outfit's vibe.
- **When it has nothing:**
If `outfit` is empty or contains only whitespace, returns a descriptive message instead of raising an error.
---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:**
If `search_listings` returns an empty list, put a message in the session and stop. Otherwise, take the first result, store it as the selected item, and continue to `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->

The query is parsed using regular expressions (regex) to extract the size and maximum price. Those parts are removed from the query, and the remaining text is used as the item description.

**What moves through the session:** <!-- which fields, in what order -->
The parsed query is stored in `session["parsed"]`, then the search results are stored in `session["search_results"]`. The first result is stored in `session["selected_item"]`, the outfit recommendation is stored in `session["outfit_suggestion"]`, and the final caption is stored in `session["fit_card"]`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask "looking for a vintage graphic tee under 30"

Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

Outfit:   Here are two outfit ideas combining the Y2K butterfly baby tee with pieces from your wardrobe:

### Outfit 1: Classic Y2K Streetwear
* **Bottoms:** Baggy straight-leg jeans, dark wash (`w_001`)
* **Shoes:** Chunky white sneakers (`w_007`)
* **Accessories:** Black crossbody bag (`w_010`)

**Why it works:** The fitted, cropped silhouette of the baby tee creates a great proportion balance when paired with high-waisted, baggy dark-wash jeans. Finish the look with chunky sneakers and a minimal black bag for an effortless, authentic early-2000s streetwear vibe.

---

### Outfit 2: Edgy Contrast Look
* **Outerwear:** Vintage black denim jacket (`w_006`)
* **Shoes:** Black combat boots (`w_008`)
* **Bottoms:** Wide-leg khaki trousers (`w_002`)

**Why it works:** This outfit plays with contrasting aesthetics. The sweet, pastel butterfly print of the baby tee pops against the tougher elements of the black denim jacket and combat boots, while the khaki trousers ground the look with a relaxed, earthy contrast.

Fit card: Channeling all the early 2000s streetwear vibes in this Y2K Baby Tee — Butterfly Print, styled with baggy denim and chunky sneakers for an effortless throwback look. Snagged this gem for just $18 over on Depop before it found a new home! 🦋✨

2 model calls this session, 1229 prompt + 318 output tokens

```

**The three tools, tested one at a time**

```
```
$ python -c "from tools import search_listings; print([(x['id'], x['title'], x['price']) for x in search_listings('graphic tee', max_price=30)])"
[('lst_002', 'Y2K Baby Tee — Butterfly Print', 18.0), ('lst_006', 'Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('lst_017', 'Mesh Long-Sleeve Top — Black', 15.0), ('lst_033', 'Vintage Band Tee — Faded Grey', 19.0), ('lst_011', 'Low-Rise Cargo Pants — Khaki', 27.0), ('lst_012', 'Oversized Crewneck Sweatshirt — Vintage Navy', 20.0), ('lst_015', 'Vintage Graphic Hoodie — Faded Black', 26.0)]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two outfit ideas combining the vintage Levi's 501s with pieces from your wardrobe:

**Outfit 1: Casual Streetwear (High-Low Contrast)**
* **Top:** White ribbed tank top (`w_003`) tucked in to highlight the waist of the 501s.
* **Outerwear:** Oversized grey crewneck sweatshirt (`w_004`) layered on top for a relaxed, cozy silhouette.
* **Shoes:** Chunky white sneakers (`w_007`).
* **Accessories:** Black crossbody bag (`w_010`).
* why it works:* The medium wash of the thrifted jeans provides a nice contrast to your dark wash jeans (`w_001`), and pairing a fitted tank with an oversized crewneck creates a balanced, effortless streetwear look.

**Outfit 2: Edgy & Cropped**
* **Top:** Black cropped zip hoodie (`w_005`).
* **Outerwear:** Vintage black denim jacket (`w_006`) worn over the hoodie for a layered denim-on-denim texture.
* **Shoes:** Black combat boots (`w_008`).
* **Accessories:** Brown leather belt (`w_009`) to tie in the vintage aesthetic.
* Why it works:* The straight-leg fit of the Levi's 501s balances the cropped layers up top, while the black boots and jacket give the classic medium-wash denim a tougher, vintage-grunge edge.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats the effortless cool of broken-in denim, especially when it's a pair of Vintage Levi's 501 Jeans — Medium Wash. Just style these with your favorite white sneakersfor an easy, laid-back streetwear vibe that goes with literally everything. Grab them on Depop for just $38.00 before someone else snags 'em!
```


---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*

I asked AI for help implementing `search_listings` based on the tool requirements, including filtering by maximum price and size and matching keywords from the user's description.

- *What came back:*

AI suggested loading the listings with the provided data loader, applying the price and size filters, and scoring listings based on keyword overlap.

- *What I changed:*

implemented the suggested logic in `tools.py` and tested it from the terminal. I also tested a search with no matching results to make sure the function returned an empty list `[]` as required.


**Moment 2**

- *What I asked for:*

I asked AI for help connecting the three tools in `run_agent()` while keeping the results in the session state.

- *What came back:*
AI suggested storing the search results in `session["search_results"]`, checking whether that list was empty, selecting the first result, and then passing the stored values through `suggest_outfit` and `create_fit_card`.

- *What I changed:*

I added the branch so an empty search stops the agent before the other tools run. I also changed the error message to tell the user to try increasing the budget, changing the size, or changing the item description instead of only saying there were no results.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

### Actual Test Output — Before


**Criterion 1 — Matching query completes all three tools**

Source: `run_eval.py::run_once`, executing `agent.py::run_agent`

```text
[2] search_listings (via MCP)
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
      →    branch: results found, continuing
[4] suggest_outfit
      out: Here are two outfit suggestions combining the Y2K butterfly baby tee with pieces from your wardrobe:  ### Outf…
[5] model_unavailable
      →    stopping because the model could not be reached
```

**Criterion 2 — Impossible query stops early**

Source: `run_eval.py::run_once`, executing `agent.py::run_agent`

```text
[2] search_listings (via MCP)
      out: [] (empty)
[3] check_search_results
      out: I couldn't find a matching item. Try increasing your budget, changing the size, or using a different style or …
      →    branch: empty, stopping
```

**Criterion 3 — Selected item state consistency**

Source: `run_eval.py::run_once`, executing `agent.py::run_agent`

```text
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
      →    branch: results found, continuing
[4] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two outfit suggestions combining the Y2K butterfly baby tee with pieces from your wardrobe:  ### Outf…
```

This shows the selected item and a successful outfit suggestion, but it does not display the two IDs needed to verify criterion 3.





**Supplementary State Consistency Verification**

I ran five additional tests using the real `run_agent()` and MCP `search_listings` tool. I mocked `suggest_outfit` and `create_fit_card` to avoid Gemini availability issues and capture the actual item ID passed to `suggest_outfit`.

```text
Try 1: PASS - selected=lst_002, passed=lst_002
Try 2: PASS - selected=lst_002, passed=lst_002
Try 3: PASS - selected=lst_002, passed=lst_002
Try 4: PASS - selected=lst_002, passed=lst_002
Try 5: PASS - selected=lst_002, passed=lst_002
```

Result: 5/5 PASS. The selected item was passed consistently to `suggest_outfit` in all five supplementary tests.


**Criterion 4 — Fit card content**

Source: `run_eval.py::run_once`, executing `agent.py::run_agent`

```text
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] model_unavailable
      →    stopping because the model could not be reached
```

**Criterion 5 — Empty wardrobe**

Source: `run_eval.py::run_once`, executing `agent.py::run_agent`

```text
empty wardrobe  (empty wardrobe)
  query: vintage graphic tee under $30
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] model_unavailable
      →    stopping because the model could not be reached
```


<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->


| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISSED (0/5) |
| 2. Impossible query stops before second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item ID matches item passed to suggest_outfit | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5, mocked model calls) |
| 4. Fit card has 2–4 sentences and mentions item, price, and platform | 4 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISSED (0/5) |
| 5. Empty wardrobe returns general styling advice | 5 of 5 | FAIL | FAIL | FAIL | FAIL | FAIL | MISSED (0/5) |


During the baseline evaluation, I ran five scenarios five times each with caching disabled using `python run_eval.py --label before`. The impossible-query scenario passed all five attempts because the agent stopped before calling `suggest_outfit` and returned a helpful message.

However, the model-dependent scenarios repeatedly failed because Gemini was unavailable. A separate diagnostic test returned `503 UNAVAILABLE`, indicating that the model was experiencing high demand. Although the agent handled the errors without crashing, it could not consistently generate outfit suggestions or fit cards. This prevented criteria 1, 4, and 5 from meeting their targets.

Criterion 3 requires additional verification to confirm that the selected item's ID matches the ID passed to `suggest_outfit`.

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->


| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Matching query completes all three tools | 4 of 5 | MISSED (0/5) | All five attempts failed to complete the full tool sequence because model availability errors interrupted execution. |
| 2 | Impossible query stops before second tool | 5 of 5 | MET (5/5) | All five attempts returned a helpful message and stopped before calling suggest_outfit. |
| 3 | Selected item ID matches item passed to suggest_outfit | 5 of 5 | MET (5/5 supplementary tests) | Five additional tests with mocked model calls confirmed that the selected item ID matched the ID passed to suggest_outfit. |
| 4 | Fit card has 2–4 sentences and mentions item, price, and platform | 4 of 5 | MISSED (0/5) | None of the five baseline attempts returned a completed fit card because model generation failed. |
| 5 | Empty wardrobe returns general styling advice | 5 of 5 | MISSED (0/5) | All five attempts failed to return styling advice because the Gemini model request was unsuccessful. |


**Diagnoses**

### Criterion 1 — Matching Query Completes All Three Tools

**Target:** 4 of 5
**Result:** 0 of 5
**Verdict: MISSED**

The matching query successfully called `search_listings` through MCP and selected an item. However, the agent did not consistently complete the `suggest_outfit` and `create_fit_card` steps.

**Failure step:** Model-dependent tool execution in `suggest_outfit` or `create_fit_card`.

**Mechanism:** A direct diagnostic request to Gemini returned `503 UNAVAILABLE`, indicating that the model was experiencing high demand. The `generate.py` adapter handles rate-limit retries but does not retry temporary `503` service errors. Consequently, `ModelUnavailable` stopped the agent before it could complete all three tools.

### Criterion 2 — Impossible Query Stops Early

**Target:** 5 of 5  
**Result:** 5 of 5  
**Verdict: MET**

All five attempts returned an empty search result, followed by a helpful message suggesting that the user increase the budget, change the size, or modify the item description. The agent returned before calling `suggest_outfit`.

**Reason:** The empty-results branch in `agent.py::run_agent` worked correctly in all five attempts.

### Criterion 3 — Selected Item State Consistency

**Target:** 5 of 5  
**Result:** 5 of 5 supplementary mocked tests  
**Verdict: MET in supplementary verification**

The original evaluation trace did not expose both item IDs, so it was insufficient to score this criterion directly.

Five supplementary tests used the real `run_agent` and MCP search, with mocked `suggest_outfit` and `create_fit_card` functions. Each test captured the actual item ID passed to `suggest_outfit` and compared it with `session["selected_item"]["id"]`.

All five comparisons passed with the ID `lst_002`.

**Reason:** `agent.py::run_agent` consistently passed the selected item from session state into `suggest_outfit`. This result verifies state consistency independently of Gemini availability.

### Criterion 4 — Fit Card Content

**Target:** 4 of 5  
**Result:** 0 of 5  
**Verdict: MISSED**

None of the five baseline attempts produced a completed fit card that could be evaluated against the requirements of 2–4 sentences mentioning the item, price, and platform.

**Failure step:** Model-dependent outfit generation or fit-card generation.

**Mechanism:** Gemini availability errors interrupted the agent before a completed fit card could be returned. This was an execution failure, not evidence that the fit-card prompt produced incorrectly formatted text.

### Criterion 5 — Empty Wardrobe Styling Advice

**Target:** 5 of 5  
**Result:** 0 of 5  
**Verdict: MISSED**

The empty-wardrobe scenario successfully found and selected an item but did not return general styling advice.

**Failure step:** `suggest_outfit` model generation.

**Mechanism:** The empty-wardrobe branch in `tools.py` constructs a prompt requesting general styling advice, but the Gemini model request failed. Therefore, the tool could not return the required suggestion.

### Pattern Across the Misses

Criteria 1, 4, and 5 share a common failure mechanism: temporary Gemini model unavailability. The search tool and early-stop branch operated successfully, and supplementary testing verified that the selected item was passed correctly through session state.

The recurring problem is in the model execution layer, particularly the lack of retry handling for temporary `503 UNAVAILABLE` responses.

**Potential improvement:** Extend `generate.py` to retry temporary `503` errors with bounded exponential backoff, while preserving the existing friendly error handling if all retries fail. Rerun the same evaluation scenarios afterward to compare results against the baseline.

---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```text
$ python app.py ask 'vintage graphic tee under $30' --trace

[1] parse_query
      in:  dict with keys: query
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
      →    branch: results found, continuing
[4] suggest_outfit
      in:  dict with keys: new_item, wardrobe
      out: Here are two outfit suggestions combining the **Y2K Baby Tee** with pieces from your wardrobe:  ### Outfit 1: …
[5] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling major early-2000s energy with this super cute Y2K Baby Tee — Butterfly Print! Styled here with bagg
```

**Empty search**

```text
$ python app.py ask 'designer ballgown size XXS under $5' --trace

[1] parse_query
      in:  dict with keys: query
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[3] check_search_results
      out: I couldn't find a matching item. Try increasing your budget, changing the size, or using a different style or …
      →    branch: empty, stopping
```

**On the MCP move:**

I moved `search_listings` from a direct function call in `tools.py` to an MCP tool exposed through `mcp_server.py`. The agent now calls the search tool through `mcp_client.py` using `call_tool()`.

The search behavior remained the same, but the architecture changed. The agent communicates with the search tool through MCP rather than calling the Python function directly.

I added tracing to record the order of tool calls and show the agent's decision-making process. When the search returns no results, the agent stops and provides a helpful message instead of continuing to outfit generation.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

## The Improvement

**What I changed:**

I updated `generate.py` to retry temporary Gemini `503 UNAVAILABLE` errors using the existing retry and exponential-backoff mechanism. Previously, only rate-limit errors were retried, while `503` errors immediately caused `ModelUnavailable`.

**Which failure it was meant to fix:**

The baseline evaluation showed that model-dependent tools frequently failed, preventing the agent from completing `suggest_outfit` and `create_fit_card`. This improvement was intended to make the agent more resilient to temporary model service outages.

### Run Log — After

Command: `python run_eval.py --label after`

The after-evaluation was attempted twice but could not be completed because model requests were taking an excessive amount of time. The evaluation was stopped after troubleshooting, and no complete after-run results file was generated.

Observed partial results:

| Attempt | Observation | Outcome |
|---|---|---|
| First evaluation | The agent completed `suggest_outfit` and `create_fit_card` and generated a 308-character fit card. | One successful workflow |
| Second evaluation, first recorded try | The agent completed `suggest_outfit` but then entered `model_unavailable` before completing `create_fit_card`. | Failed to complete workflow |

**Did it help, and how do I know:**

The improvement showed a promising result because one workflow successfully completed all required tools, compared with zero successful completions in the baseline. However, another attempt still failed because the model became unavailable.

Because the full after-evaluation did not finish, I cannot determine whether the improvement consistently met the five evaluation criteria. The results suggest possible improvement, but they are inconclusive.

The remaining limitation is dependence on Gemini API availability and response time. Retrying temporary errors cannot guarantee that the service will respond successfully.


**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

### What's Still Broken

The improvement in Milestone 5 addressed temporary Gemini `503 UNAVAILABLE` errors by adding retry handling. However, the full after-evaluation could not be completed because model requests were taking too long. The following limitations remain.

**Criterion 1 — Matching query completes all three tools (Missed)**

The baseline result was 0/5. During the after-evaluation, one attempt successfully completed the workflow, but another stopped when the model became unavailable.

To improve this, I would investigate Gemini API availability and request timeouts, then repeat the full evaluation to determine whether the retry mechanism consistently improves completion rates.

**Criterion 2 — Impossible query stops early (Met)**

This criterion passed 5/5 in the baseline. The agent correctly stops when no listings are found. No additional improvement is currently needed.

**Criterion 3 — Selected item consistency (Met in supplementary verification)**

The selected item ID matched the item passed to `suggest_outfit` in all five supplementary tests. These tests used mocked model-dependent calls rather than the original live-model evaluation.

I would improve the evaluation logging to show the selected and passed item IDs directly, allowing this behavior to be verified without separate tests.

**Criterion 4 — Fit card content (Missed)**

The baseline result was 0/5 because the agent could not consistently reach `create_fit_card`.

I would first resolve the model availability problem and complete the evaluation. After that, I would review the generated fit cards and adjust the prompt if necessary to ensure each card contains 2–4 sentences and mentions the item, price, and platform.

**Criterion 5 — Empty wardrobe advice (Missed)**

The baseline result was 0/5 because model availability errors prevented reliable completion.

I would test the empty-wardrobe scenario after the model connection becomes reliable and verify that the agent produces general styling advice without requiring existing wardrobe items.

**Why I stopped here**

I focused on one improvement as required by Milestone 5. I attempted the after-evaluation twice, but repeated model delays prevented a complete run. Rather than introducing additional changes without reliable measurements, I documented the partial results and remaining limitations.

### MCP Migration

I moved the listing search functionality to an MCP server and updated the agent to call `search_listings` through the MCP client instead of calling the search function directly.

The main change was how the agent accessed the listing search tool. The search results and agent decision-making behavior remained consistent: matching queries returned listings, while impossible queries triggered the early-stop branch.

The MCP migration separated the listing search capability from the agent's orchestration logic. It did not resolve the Gemini API availability issues affecting the model-dependent tools.


### How I Used AI

I used AI to help interpret the assignment requirements, understand the evaluation results, and diagnose why certain criteria were failing.

AI helped me identify that `generate.py` retried rate-limit errors but did not retry temporary Gemini `503 UNAVAILABLE` errors. I used this guidance to implement one improvement using the existing retry and exponential-backoff mechanism.

I also used AI to interpret the partial after-evaluation results and organize my README documentation, including the criterion verdicts, failure diagnoses, and remaining limitations.

I reviewed the suggested changes and ran the evaluation commands in my local development environment.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
