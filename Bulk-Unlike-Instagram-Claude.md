# Instagram Bulk Unlike — Claude Automation Guide

## Purpose

This document instructs a Claude agent (running in Cowork mode with the Claude-in-Chrome browser extension) on how to bulk-unlike all liked posts on an Instagram account. Instagram caps bulk selection at 99 posts per action, so this is a repeating loop. The agent runs each round autonomously until no liked posts remain.

---

## Prerequisites

- Claude desktop app with **Cowork mode** enabled
- **Claude-in-Chrome** browser extension installed and connected to Claude
- Google Chrome open with the user's Instagram account already logged in
- The Claude-in-Chrome `mcp__Claude_in_Chrome__*` tools must be available (specifically `browser_batch`, `computer`, `javascript_tool`, and `navigate`)

---

## Starting Instructions for Claude

When the user asks you to bulk-unlike their Instagram likes, say:

> "I'll start removing your liked posts in batches of 99, oldest first. I'll keep going until the page is empty. Let me know if you want me to stop at any point."

Then begin the loop below. Do **not** ask clarifying questions — just start.

---

## The Per-Round Loop

Repeat the following sequence continuously. Each round removes up to 99 liked posts.

---

### Step 1 — Navigate fresh

```
Navigate to: https://www.instagram.com/your_activity/interactions/likes/
Wait 5 seconds.
Take a screenshot to confirm posts have loaded.
```

If the page shows a loading spinner for more than 15 seconds, wait another 10 seconds and take another screenshot. If posts still don't appear, navigate fresh again.

If the page grid is **completely empty** (no posts visible after waiting), the job is done — stop the loop and tell the user their likes have been cleared.

---

### Step 2 — Apply "Oldest to newest" sort

> **Critical:** Instagram resets the sort order to "Newest to oldest" on every page navigation. You MUST re-apply the sort every single round, or you will delete the newest posts first.

1. Click **Sort & filter** button (top-center of the page, next to the sort label).
2. Wait 2 seconds for the modal to open.
3. In the modal, click **"Oldest to newest"**.
4. Wait 1 second.
5. Click **Apply**.
6. Wait 5–10 seconds for the page to reload with the correct sort order and posts to appear.

If the modal closes immediately after clicking Sort & filter (because you double-clicked), click Sort & filter again — click it once only and wait for it to open.

---

### Step 3 — Activate Select mode

The Select button requires careful handling — it often needs multiple clicks before the "0 selected" bar appears.

1. Click the **Select** button (top-right corner, styled as a blue text link).
2. Wait 2 seconds.
3. Click **Select** again.
4. Wait 3 seconds.
5. Take a screenshot and check: is there a **"0 selected"** bar at the bottom of the page with an **Unlike** button on the right?
   - **Yes** → proceed to Step 4.
   - **No** → click Select once more, wait 3 seconds, check again. Repeat until "0 selected" appears.

---

### Step 4 — Run the JavaScript selection script

Use the `javascript_tool` to execute the following script in the browser tab. This script scrolls the page to load more posts via Instagram's lazy-loading, then clicks up to 99 checkboxes.

```javascript
(async () => {
  const delay = (ms) => new Promise(r => setTimeout(r, ms));
  const getCount = () =>
    document.querySelectorAll('[data-testid="bulk_action_checkbox"] [role="button"]').length;
  const scroller = [...document.querySelectorAll('div')]
    .find(el => el.scrollHeight > el.clientHeight && el.clientHeight > 300);
  if (!scroller) return console.log('Scroller not found');
  let lastCount = 0, stableRounds = 0;
  while (getCount() < 120 && stableRounds < 5) {
    for (let i = 0; i < 5; i++) { scroller.scrollTop += 800; await delay(400); }
    await delay(1500);
    const newCount = getCount();
    if (newCount === lastCount) stableRounds++;
    else { stableRounds = 0; lastCount = newCount; }
  }
  const checkboxes = [...document.querySelectorAll('[data-testid="bulk_action_checkbox"] [role="button"]')]
    .filter(el => el.offsetParent !== null);
  let selected = 0;
  for (const cb of checkboxes) {
    if (selected >= 99) break;
    cb.click(); selected++; await delay(90);
  }
  console.log('Selected ' + selected);
})();
```

**What this script does:**
- Finds the scrollable div container on the page (the one taller than 300px that scrolls).
- Scrolls it repeatedly in batches of 5 × 800px to trigger Instagram's lazy-loading.
- Stops scrolling when either 120+ checkboxes are loaded, or the count hasn't grown for 5 consecutive scroll attempts (meaning no more posts are available to load).
- Selects up to 99 visible checkboxes by clicking each one with a 90ms delay between clicks.
- Logs "Selected N" to the browser console when complete.

**After calling javascript_tool:**

- If it returns `undefined` immediately → the script finished fast (within the 45-second timeout). Wait 10 seconds, then take a screenshot.
- If it returns a **CDP timeout error** after ~45 seconds → the script ran longer than the timeout but is **still executing in the browser**. Wait an additional 20–30 seconds, then take a screenshot.

Do **not** click anything while waiting — clicks during script execution can deselect posts.

---

### Step 5 — Verify selection count

Take a screenshot. The bottom bar should read **"99 selected"** (or a smaller number if near the end of the list). Confirm the **Unlike** button is visible and red on the right side of the bar.

If the count is 0 (script failed), go back to Step 3 and re-activate Select mode, then re-run the script.

---

### Step 6 — Click Unlike

Click the **Unlike** button at the bottom-right of the screen (in the selection bar, not the one in the header).

Wait 2 seconds for the confirmation modal to appear.

---

### Step 7 — Confirm the modal

A modal dialog will appear with the text:
> "Unlike posts? Are you sure you want to unlike these posts?"

Click the red **Unlike** button inside the modal (approximately centered on the screen, upper option — not Cancel).

---

### Step 8 — Wait, then start the next round

Wait 10 seconds for Instagram to process the unlike actions server-side.

Then go back to **Step 1** and repeat.

---

## Coordinate Reference (for `mcp__Claude_in_Chrome__computer`)

These are approximate coordinates for the standard Instagram activity page layout at 1060×1148 viewport. Take a screenshot first to verify before clicking — Instagram's layout may shift slightly.

| Element | Approximate coordinate |
|---|---|
| Sort & filter button | (531, 114) |
| "Oldest to newest" option in modal | (453, 452) |
| Apply button in modal | (530, 721) |
| Select button (top-right) | (947, 112) |
| Unlike button in selection bar | (946, 922) |
| Unlike button in confirmation modal | (530, 460) |

> Always take a screenshot before clicking to confirm the element is where you expect it.

---

## Troubleshooting

### Sort resets to "Newest to oldest"
This happens on every fresh navigation — it is expected. Always re-apply "Oldest to newest" at the start of each round.

### Select button doesn't activate the "0 selected" bar
Click Select one more time and wait 3 seconds. Repeat until the bar appears. The pattern of "click, wait 2s, click again, wait 3s" works most reliably. Occasionally a third or fourth click is needed.

### Script times out (CDP 45-second error)
This is not a failure. The script is still running in the browser background. Wait 20–30 more seconds and take a screenshot. The "99 selected" count will appear once it finishes.

### Page shows a spinner but no posts load
The Instagram server is slow to respond after a bulk unlike. Wait up to 30 seconds total. If posts still don't appear, navigate fresh.

### Fewer than 99 posts selected
This means you are near the end of your liked posts. There are fewer than 99 remaining. Proceed normally — unlike however many are selected.

### Page is empty after navigation
The account's likes have been fully cleared. Tell the user the job is complete.

---

## Full Round Summary (quick reference)

```
1. Navigate to https://www.instagram.com/your_activity/interactions/likes/
2. Wait 5s
3. Click Sort & filter → Oldest to newest → Apply → Wait 5–10s
4. Click Select → Wait 2s → Click Select again → Wait 3s → Verify "0 selected" bar
   (repeat Select clicks if bar doesn't appear)
5. Run JS script via javascript_tool → Wait 10–30s
6. Verify "99 selected" in bottom bar
7. Click Unlike (bottom-right)
8. Wait 2s → Click Unlike in confirmation modal
9. Wait 10s
10. Repeat from step 1
```

---

## Notes on Instagram's Behavior

- **Lazy-loading:** Instagram only loads ~27–30 posts into the DOM initially. The JS script scrolls to load more before selecting, which is why it takes time.
- **99-post cap:** Instagram enforces a maximum of 99 posts per bulk-select action. The script respects this limit.
- **Sort persistence:** The "Oldest to newest" sort does NOT persist across page navigations. It must be re-applied every round without exception.
- **Rate limiting:** Instagram may briefly throttle requests if rounds are performed too quickly back-to-back. The 10-second wait after each unlike batch is intentional. If you encounter errors or empty pages mid-session, wait 60 seconds before the next round.
- **Progress indicator:** The content visible in the top-left post after sorting "Oldest to newest" tells you what era of likes you're in. As rounds progress, the dates advance forward toward the present.
