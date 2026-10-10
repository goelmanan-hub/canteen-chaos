# Debug log

Your notes. One entry per bug you fixed, using the template below.

This file is read as carefully as your code. A correct fix you cannot
explain counts for little; a bug you could not fix but investigated
honestly still counts for something.

Delete the example before you submit.



## CC-01 — "The search suggestions are behind everything"

**Reproduced:** Typed "ch" or "paneer" into the search bar. The suggestions dropdown rendered, but items below the top edge were tucked underneath `.cat-tabs` (the category tab bar). Clicking any suggestion below the top item failed because the click was captured by the category tabs sitting on top of the dropdown.

**Cause:** In `frontend/style.css`, `.search-wrap` had `z-index: 1`, while its sibling `.cat-tabs` had `z-index: 40`. Since `.search-wrap` created its own stacking context with `z-index: 1`, its child `.suggest-box` could not render above `.cat-tabs` despite having `z-index: 100`.

**Fix:** Increased `z-index` of `.search-wrap` from `1` to `50` in `frontend/style.css` so that the search container and its child dropdown sit above `.cat-tabs` (which has `z-index: 40`).

**Checked:** Typed various dish names in the search input; the suggestion box now floats cleanly above the category bar and all suggestions can be clicked directly.

**Time:** about 15 minutes.



## CC-02 — "Can't read anything in dark mode"

**Reproduced:** Toggled theme to dark mode via the theme button (setting `data-theme="dark"` on `<html>`). The dish card background turned dark (`#201913`), but dish names and prices remained dark brown (`#2b2118`), making text practically unreadable against the card background.

**Cause:** In `frontend/style.css`, `.dish-body` had a hardcoded `color: #2b2118;`. In dark mode, `--card` background is `#201913`. Because the color was hardcoded instead of using CSS custom property `--ink` (which is `#f2e8df` in dark mode and `#2b2118` in light mode), the inherited text color did not adapt to dark mode.

**Fix:** Changed `color: #2b2118;` to `color: var(--ink);` on `.dish-body` in `frontend/style.css`.

**Checked:** Switched between light and dark themes; dish names and prices are crisp and high-contrast in both modes.

**Time:** about 10 minutes.



## Could not fix

For anything you investigated but did not solve. Say what you tried and
where you got to. This is worth marks — leaving it blank when you got
stuck is not.

### CC-0X — "<the complaint>"

**What I tried:**

**Where I got to:**

**What I would try next:**



## Extra credit

Anything not on the bug log: a problem you found yourself, a test you
wrote, or a fix you are unsure about. Same format, plus one line on how
you noticed it.
