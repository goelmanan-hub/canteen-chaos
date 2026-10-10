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



## CC-04 — "The buttons don't work on my tablet"

**Reproduced:** Simulated tablet viewport between 761px and 900px (e.g. 800px width). Clicked "Add to Cart" or the star/favorite icon on any dish card. Neither button reacted to clicks, despite cursor and styles looking normal. Testing at < 760px (mobile) or > 900px (desktop) worked fine.

**Cause:** In `frontend/style.css`, inside `@media (min-width: 761px) and (max-width: 900px)`, `.dish-card::after` positioned an invisible pseudo-element (`height: 58px; z-index: 3;`) directly over the bottom Add to Cart area, and `.img-wrap::after` positioned an invisible pseudo-element (`width: 52px; height: 52px; z-index: 1;`) over the top-right favorite star. Because neither had `pointer-events: none`, they intercepted all mouse/touch click events.

**Fix:** Added `pointer-events: none;` to `.dish-card::after` and `.img-wrap::after` in the tablet media query in `frontend/style.css`, allowing clicks to pass directly through to the underlying action buttons.

**Checked:** Tested clicking Add to Cart and favorite stars at 768px, 800px, and 850px viewport widths. All buttons respond immediately to clicks.

**Time:** about 15 minutes.



## CC-05 — "The category bar scrolls away on my phone"

**Reproduced:** On mobile viewports (<= 480px), scrolling down the dishes list caused the sticky category/filter bar to scroll away out of sight instead of remaining pinned at the top.

**Cause:** In `frontend/style.css`, `@media (max-width: 480px)` contained `.view { overflow-x: hidden; }`. In CSS specification, applying `overflow: hidden` or `overflow-x: hidden` to any parent ancestor breaks `position: sticky` on all its descendant elements (in this case `.filters`, which has `position: sticky; top: 0;`).

**Fix:** Removed `.view { overflow-x: hidden; }` from the mobile media query. Since the root cause of horizontal overflow (CC-03) was fixed by proper card and grid constraints, the band-aid `overflow-x: hidden` was unnecessary and was actively breaking the sticky behavior.

**Checked:** Scrolled down the dishes list at mobile widths (375px - 480px). The category and filter bar remains firmly sticky at the top of the viewport.

**Time:** about 15 minutes.



## CC-06 — "I ordered more than they had"

**Reproduced:** Found a dish with stock of 2 (e.g. samosas). Placed an order with `qty: 5`. The order was accepted with HTTP 201, decrementing stock into negative values (-3).

**Cause:** In `backend/logic/validation.js`, `validateLine` checked `if (Number(dish.stock) <= 0) errors.push(...)` (sold out check), but never checked if the ordered quantity exceeded available stock (`qty > Number(dish.stock)`). Furthermore, `reserveStock` did not check if `dish.stock < item.qty` before subtracting.

**Fix:** Added validation in `validateLine`: `if (qty > Number(dish.stock)) errors.push(...)` and guarded `reserveStock` to ensure `dish.stock >= item.qty`.

**Checked:** Attempting to order quantity greater than current stock returns HTTP 400 with an explicit validation error, while ordering quantity within stock succeeds.

**Time:** about 15 minutes.



## CC-07 — "Cancelling makes it worse"

**Reproduced:** Placed an order for 2 plates of a dish with stock 4 (stock dropped to 2). Cancelled the order via `PATCH /api/orders/:token/status` with `status: 'cancelled'`. Instead of restoring stock to 4, stock dropped further to 0.

**Cause:** In `backend/logic/validation.js`, inside `releaseStock(items, menu)`, the function returned `{ ...dish, stock: dish.stock - line.qty }`, subtracting the quantity again on cancellation instead of restoring it.

**Fix:** Changed `-` to `+` in `releaseStock` so `stock: dish.stock + line.qty`.

**Checked:** Placed an order for 2 items and cancelled it; verified dish stock increased back by 2 to its original amount. Also verified `smoke.js` cancellation check now passes.

**Time:** about 10 minutes.



## CC-08 — "An old coupon still works"

**Reproduced:** Created an order meeting the minimum order value of Rs. 100 and applied the coupon `FRESHERS24` (which had expired on 2024-09-30). The coupon was successfully accepted and granted a Rs. 30 discount.

**Cause:** In `backend/logic/pricing.js`, `applyCoupon` checked `usesLeft`, `minOrder`, `vegOnly`, and meal `slots`, but completely omitted verifying the `coupon.expiresAt` timestamp against current date/time `now`.

**Fix:** Added an expiration check in `applyCoupon`: `if (coupon.expiresAt && new Date(coupon.expiresAt).getTime() <= new Date(now).getTime()) return { valid: false, discount: 0, reason: '${coupon.code} has expired' };`.

**Checked:** Tested cart quote with `FRESHERS24` on an order exceeding Rs. 100; it is now rejected with `"FRESHERS24 has expired"`, while valid unexpired coupons continue to apply normally.

**Time:** about 15 minutes.



## CC-09 — "The menu shows more dishes than it should"

**Reproduced:** Loaded the menu page or queried `/api/menu?limit=5&page=1` and `/api/menu?limit=5&page=2`. Instead of returning only 5 dishes per page, the API dumped all dishes in the database at once, and both page 1 and page 2 returned the identical full dataset starting from index 0.

**Cause:** In `backend/logic/search.js`, inside `paginate(list, page, limit)`, the sliced array `items = list.slice(start, start + perPage)` was correctly computed, but the return statement returned `items: list` (the entire un-sliced array) instead of the sliced `items`.

**Fix:** Changed `items: list` to `items` (or `items: items`) in the return object of `paginate` in `backend/logic/search.js`.

**Checked:** Queried `/api/menu?limit=5&page=1` and `/api/menu?limit=5&page=2`; each now returns exactly 5 dishes, and page 1 and page 2 contain different dishes. The frontend infinite scroll/pagination now correctly batches requests.

**Time:** about 15 minutes.



## CC-10 — "Sorting by price is backwards"

**Reproduced:** Selected "Price: low to high" from the sort dropdown or requested `/api/menu?sort=price-asc`. The most expensive dishes were returned first. Selecting "Price: high to low" (`/api/menu?sort=price-desc`) returned the cheapest dishes first.

**Cause:** In `backend/logic/search.js`, the comparator functions in `SORTERS` were inverted: `'price-asc'` was defined as `(a, b) => b.price - a.price` (descending), while `'price-desc'` was defined as `(a, b) => a.price - b.price` (ascending).

**Fix:** Corrected both comparators in `backend/logic/search.js`: `'price-asc': (a, b) => a.price - b.price` and `'price-desc': (a, b) => b.price - a.price`.

**Checked:** Verified that sorting by "Price: low to high" returns items in strictly increasing price order, and "Price: high to low" in strictly decreasing price order.

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
