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

user types a query like `python app.py ask 'vintage graphic tee under $30'` for what they want. the agent searches the listings for a match, picks the best one, and asks the model to suggest an outfit pairing it with the user's wardrobe. then it asks the model to turn that into a short fit card caption. if the search finds nothing, the agent stops and tells the user what to change instead of continuing with nothing to work with.

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

- **What it does:** searches listings data for items that match keyword description. Can be filtered by size and max price
- **Inputs:** <!-- name and type each: `max_price` (float), not "a price" --> description, a string. size, a str or None, max_price, a float or None
- **Returns:** list of dictionaries containing id, title, description, category, style_tags, size, condition, price, colors, band, platform
- **When it has nothing:** [] an empty list

### `suggest_outfit`

- **What it does:** gets one or two outfit combos = paired with clothing the owner already owns
- **Inputs:** new_item, a dictionary, wardrobe, a dictionary with items key (contains a list of wardrobe item dicts)
- **Returns:** str of suggested outfits from the model
- **When it has nothing:** if wardrobe items is empty it give gneral style advice instead of matching with already owned clothing. it always returns something

### `create_fit_card`

- **What it does:** writes a short post about the thrifted piece and the suggested outfit
- **Inputs:** outfit, a str, new_item, a dit
- **Returns:** a string containing the item, price and its platform
- **When it has nothing:** if outfit is empty it. returns a description without calling the model

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

**Branch rule:** if the serach_listings returns an empty list, put a message in session["error"] describing what to change and stop. Else, take the first result as session["selected_item"] and continue to suggest_outfit then create_fit_card 

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->
Regex should pulla $<number> as max_price and a size token. The rest is the description string

**What moves through the session:** query → parsed (description/size/max_price) → search_results → selected_item → outfit_suggestion → fit_card. error gets set instead if the search comes back empty.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'
  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here is an outfit idea that pairs the Y2K butterfly baby tee with pieces from your existing wardrobe:

**Outfit: Y2K Contrast Casual**
*   **Top:** Y2K Baby Tee — Butterfly Print
*   **Bottoms:** Baggy straight-leg jeans, dark wash
*   **Outerwear:** Black cropped zip hoodie (worn open or draped over the shoulders)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** The slim, fitted silhouette of the baby tee balances out the voluminous shape of the baggy dark wash jeans for that quintessential Y2K proportion play. Adding the black cropped zip hoodie and chunky white sneakers ties the look together with an effortless, streetwear-inspired edge.

  Fit card: Found the ultimate Y2K butterfly baby tee on Depop for just $18, and honestly, my inner 2000s pop star is screaming. I'm pairing it with baggy dark-wash jeans and a cropped zip hoodie for that effortless contrast. So obsessed with how this turned out! 🦋✨
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"
[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', ... 'price': 18.0, ...}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', ... 'price': 24.0, ...}, ... 7 results total, all priced $30 or under]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two outfit suggestions featuring your new vintage Levi's 501 jeans and pieces from your existing wardrobe:

**Outfit 1: Casual & Cool**
*   **Top:** White ribbed tank top
*   **Outerwear:** Vintage black denim jacket
*   **Shoes:** Chunky white sneakers
...
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Found my new holy grail pair of Vintage Levi's 501 Jeans for just $38.0 on Depop! Can't wait to live in these with a crisp white tee and fresh sneakers. Nothing beats the hunt.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked Claude to review my acceptance criteria 
- *What came back:* It pointed out that I had used the incorrect field
- *What I changed:* I corrected the field to max_price

**Moment 2**

- *What I asked for:* I asked for assistance implenting the tools.py and agent.py when i hit roadblocks. After it returned code I asked for explanations of certain sections as well as an overview of its general function
- *What came back:* completed functional code
- *What I changed:* I reviewed the code and did not change anything.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

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
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



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

```
$ python app.py ask 'vintage graphic tee under $30' --trace
[1] parse_query
      in:  dict with keys: query
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
[3] suggest_outfit
      in:  dict with keys: item
      out: Here is an outfit idea that pairs the Y2K butterfly baby tee with pieces from your existing wardrobe: ...
[4] create_fit_card
      in:  dict with keys: outfit
      out: Found the ultimate Y2K butterfly baby tee on Depop for just $18, and honestly, my inner 2000s pop star is screaming...
```

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5' --trace
[1] parse_query
      in:  dict with keys: query
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[3] branch
      →    empty search results — stopping before suggest_outfit
```

**On the MCP move:** Moved `search_listings` onto MCP. In `agent.py`, the direct call `search_listings(description, size, max_price)` became `call_tool("search_listings", {...})` from `mcp_client.py`. Nothing behaved differently — same results, same shape — because the server just wraps the existing implementation. The only new thing is the trace step name says "(via MCP)" so the call is visible in the trace.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



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
