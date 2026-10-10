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



## CC-03 — "The menu is wider than my phone"

**Reproduced:** Resized browser viewport to mobile width (~390px - 600px). The dishes in the grid caused horizontal overflow; scrolling sideways was necessary and the right-hand dishes had their Add to Cart buttons clipped past the right viewport boundary.

**Cause:** In `frontend/style.css`, under `@media (max-width: 760px)`:
1. `.name-btn` was set to `width: max-content;`, preventing long dish names from wrapping and forcing the button and card to expand horizontally.
2. `.dish-card img` had `width: auto; min-width: 100%; height: 170px; aspect-ratio: 3 / 2;`, forcing an intrinsic width of at least 255px per column.
3. The grid used `repeat(2, 1fr)` where `1fr` tracks default to `min-width: auto`, preventing cards from shrinking to fit smaller screens.

**Fix:** Changed `.name-btn` to `width: 100%` so dish titles wrap neatly; updated `.dish-card img` to `width: 100%; height: 170px; object-fit: cover;` without forced aspect ratio/min-width blowout; and used `grid-template-columns: repeat(2, minmax(0, 1fr));` so grid cells shrink within the viewport.

**Checked:** Tested at various mobile viewport widths (375px, 414px, 540px, 720px). Dishes and buttons fit completely within the screen with no horizontal scrolling or clipped buttons.

**Time:** about 25 minutes.



## CC-05 — "The category bar scrolls away on my phone"

**Reproduced:** On mobile viewports (<= 480px), scrolling down the dishes list caused the sticky category/filter bar to scroll away out of sight instead of remaining pinned at the top.

**Cause:** In `frontend/style.css`, `@media (max-width: 480px)` contained `.view { overflow-x: hidden; }`. In CSS specification, applying `overflow: hidden` or `overflow-x: hidden` to any parent ancestor breaks `position: sticky` on all its descendant elements (in this case `.filters`, which has `position: sticky; top: 0;`).

**Fix:** Removed `.view { overflow-x: hidden; }` from the mobile media query. Since the root cause of horizontal overflow (CC-03) was fixed by proper card and grid constraints, the band-aid `overflow-x: hidden` was unnecessary and was actively breaking the sticky behavior.

**Checked:** Scrolled down the dishes list at mobile widths (375px - 480px). The category and filter bar remains firmly sticky at the top of the viewport.

**Time:** about 15 minutes.



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
