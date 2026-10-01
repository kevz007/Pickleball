# Prompt for Claude Code — Cebu Pickleball Court booking picker

Copy everything below the line into Claude Code, run from the root of your booking plugin.

---

I have an existing booking plugin in this repo. Update **only its front-end** (markup, CSS and client-side JS) so the booking UI's **layout, colours, typography and interactions match the spec below exactly**.

**Scope: front-end only.**
- Do not touch the back-end: PHP/server logic, database, REST/AJAX endpoints, admin settings, payment or availability logic.
- Keep the plugin's existing build tooling, file structure and naming conventions.
- If the UI needs data (courts, prices, booked slots), read it from whatever the front-end already receives. Where nothing exists yet, use a single clearly marked JS config object at the top of the front-end script.

Before editing anything, explore the repo and tell me in a short plan which front-end files render the booking UI. Then implement.

## 1. Design tokens

Add these as CSS custom properties, scoped to the plugin root (e.g. `.cpc-booking`), so they don't leak into the host theme.

```
--cpc-cream:       #FBF4EC   /* page bg, unselected chips */
--cpc-white:       #FFFFFF   /* picker card */
--cpc-ink:         #141414   /* text, selected date */
--cpc-muted:       #555555   /* helper text */
--cpc-body:        #3A3A3A
--cpc-teal:        #0E7C86   /* summary panel, selected court, evening slots */
--cpc-orange:      #F0592A   /* accents, weekend labels, today dot */
--cpc-yellow:      #F4B63A   /* selected daytime slot, sun icon */
--cpc-lime:        #C6DC3A   /* primary CTA (Messenger), labels on teal */
--cpc-peach:       #FDE3D8   /* weekend calendar cells */
--cpc-grey:        #EEEAEA   /* unavailable slot */
--cpc-line:        rgba(20,20,20,.10)
--cpc-line-strong: rgba(20,20,20,.12)
```

Fonts (Google Fonts): **Outfit** 500/600/700/800/900 for headings, labels, buttons and numbers; **DM Sans** 400/500/700 for body copy; **Yellowtail** for the script accent word in the section title. Enqueue them the way the plugin already loads assets.

## 2. Layout

Section wrapper: `max-width:1320px`, centred, horizontal padding `clamp(16px,4vw,48px)`.

**Section header**
- Eyebrow "STEP 01": Outfit 600, 14px, letter-spacing .16em, orange.
- Title "Pick your *slot.*": Outfit 800, `clamp(44px,6vw,92px)`, line-height .95, letter-spacing -.035em. The word "slot." is Yellowtail 400, orange, letter-spacing 0.

**Two-column grid** (gap 20px, align start)
- ≥ 900px: `grid-template-columns: minmax(0,1fr) minmax(300px,380px)`. The picker card is on the left and the summary panel on the right, with `position:sticky; top:96px`.
- < 900px: a single column, with the summary stacked below the picker.

### 2a. Picker card
- Style: white, radius 32px, padding `clamp(22px,3vw,40px)`, shadow `0 30px 70px -40px rgba(0,0,0,.18)`.
- Contents: three stacked groups with gap `clamp(28px,3vw,40px)`. Each group has a label in Outfit 800 20px: "1. Date", "2. Court", "3. Timeslot".

**1. Date — month calendar**
- **Header row:** the label on the left. On the right, a prev button (←), the month label and a next button (→).
  - Buttons: 40px circles, border 1.5px `--cpc-line-strong`, bg cream, hover white. Disabled state: opacity .35, `not-allowed` cursor.
  - Month label: e.g. "October 2026", Outfit 800 17px, min-width 150px, centred.
- **Calendar panel:** full width, cream bg, radius 24px, padding `clamp(10px,1.4vw,14px)`.
  - **Weekday header:** 7-column grid, gap 4px. Labels "Sun…Sat" in Outfit 700 11px, letter-spacing .1em. Sun/Sat in orange, weekdays in `--cpc-muted`.
  - **Day grid:** 7 columns (`repeat(7,minmax(0,1fr))`), gap 4px, with leading blank cells up to the first weekday of the month.
  - **Day cell (button):**
    - Size and type: height `clamp(36px,4vw,46px)`, radius 12px, border 1.5px, Outfit 700 14px.
    - Weekday cell: white bg, border `rgba(20,20,20,.08)`.
    - Weekend cell (Sat/Sun): peach bg and peach border.
    - Selected: ink bg, ink border, white text.
    - Past dates, and dates beyond the booking window: disabled, opacity .3, `not-allowed`.
    - Today: a 4px dot, centred 4px from the bottom, orange (lime when today is selected).
- **Legend** below the grid: Outfit 600 12px, `--cpc-muted`, gap 18px.
  - "WEEKDAY RATE": a 12px white square with a border.
  - "WEEKEND RATE": a 12px peach square.
  - "TODAY": a 6px orange dot.
- **Behaviour:**
  - Default selection is today.
  - The booking window runs from the current month to 2 months ahead. Make the number of months a config value.
  - Prev is disabled on the current month, and next is disabled on the last allowed month.
  - When the month changes, slide the grid in 30px from the direction of travel (fade in, 0.5s, expo-out) and nudge the month label 12px vertically.

**2. Court**
- Grid `repeat(auto-fit,minmax(120px,1fr))`, gap 10px, with one button per court (Courts 1–4, from the front-end config).
- Button: height 88px, radius 20px, border 1.5px, column layout with gap 6px.
  - Content: an outline court icon above "Court N" (Outfit 800 15px).
  - Icon: an SVG 48×28, drawn in `currentColor`. It has an outer rect, a centre line at 2px, and two kitchen lines at x=17 and x=31 at 1.2px.
  - Unselected: cream bg, `--cpc-line` border, ink text.
  - Selected: teal bg, teal border, white text.
  - Transition: all .25s.
- Behaviour: single select, defaulting to Court 1.

**3. Timeslot**
- Header: the label on the left. On the right, "Tap one or more hours" in Outfit 600 13px, `--cpc-muted`.
- Grid `repeat(auto-fill,minmax(118px,1fr))`, gap 8px.
- Hourly slots start from 6 AM, with the last one 11 PM – 12 AM. Show them as "6AM – 7AM" … "11PM – 12AM".
- Slot button: padding 12px 10px, radius 16px, border 1.5px, left-aligned.
  - Line 1: the time label, Outfit 700 14px.
  - Line 2: the price, e.g. "₱600", Outfit 600 12px at opacity .75.
- Slot states:
  - Available: cream bg, `--cpc-line` border.
  - Selected, daytime (before 5 PM): yellow bg and border, ink text.
  - Selected, evening (5 PM onward): teal bg and border, white text.
  - Unavailable (4 PM – 5 PM): grey bg, opacity .55, `not-allowed`, disabled. The price line reads "Unavailable".
- Behaviour: multi-select toggle, keeping the selection sorted.
  - Prices update immediately when the date changes (weekday ↔ weekend).
  - Expose one front-end function, e.g. `isSlotUnavailable(date, court, hour)`, that returns `true` only for the 4 PM hour for now. That leaves room to plug in real availability later without changing the UI.

### 2b. Summary panel (right column)
- Style: teal bg, white text, radius 32px, padding `clamp(24px,3vw,36px)`, flex column with gap 22px, overflow hidden.
- Decoration: a 200px circle of `rgba(255,255,255,.07)` at the top right, offset -60px/-60px.
- **Header row:**
  - "YOUR BOOKING": Outfit 600 13px, letter-spacing .16em, lime.
  - A 40px icon on the right. It is a yellow sun (a circle). When any selected hour is 5 PM or later, it morphs into a lime crescent moon (0.6s), and morphs back when there isn't. Use GSAP MorphSVG if the plugin already has GSAP; otherwise cross-fade two SVGs.
- **Detail rows** (Outfit): each row has the label on the left in `rgba(255,255,255,.8)` weight 500, and the value on the right in weight 700. Rows are separated by 1px `rgba(255,255,255,.18)` lines with 12px bottom padding.
  - Date: e.g. "Sat, Oct 3 · Weekend".
  - Court: "Court N".
  - Time: consecutive hours merged into ranges, e.g. "5PM–7PM, 9PM–10PM", or "Pick a timeslot" when empty.
  - Hours: a count.
- **Total:**
  - Label "ESTIMATED TOTAL": Outfit 600 13px, letter-spacing .12em.
  - Amount: Outfit 900 `clamp(48px,5vw,64px)`, letter-spacing -.03em, formatted as "₱1,600". On every change it pulses from scale 1.12 to 1 (0.5s, back-out), with the transform origin at the left.
  - Note below it, DM Sans 14px: "Partial payment to confirm via Messenger. Remaining balance at the counter."
- **Actions** (column, gap 10px):
  - Primary: "CHAT ON MESSENGER", a full-width pill with a Messenger glyph.
    - Style: lime bg, ink text, Outfit 800 14px, padding 17px 22px, hover yellow.
    - Disabled until at least one slot is picked: bg `rgba(255,255,255,.35)`, opacity .7, no pointer events.
    - Opens `https://m.me/<PAGE_USERNAME>?text=<encoded message>` in a new tab. The page username is a value in the front-end JS config.
  - Secondary: "COPY BOOKING DETAILS", a transparent pill with a 1.5px `rgba(255,255,255,.35)` border.
    - It copies the same message to the clipboard, then shows "COPIED ✓" for 2s.
- **Message format:** `Hi Cebu Pickleball Court! I'd like to book Court {n} on {Ddd, Mmm D}, {time ranges}. Estimated total ₱{total}.`

## 3. Pricing rules

Per court, per hour, in Philippine pesos. Keep them in the front-end JS config object, not scattered through the markup. These are display and estimate only; don't add server-side pricing.

| Day type | 6:00 AM – 4:00 PM | 5:00 PM – 12:00 MN |
|---|---|---|
| Weekday (Mon–Fri) | ₱600 | ₱700 |
| Weekend (Sat–Sun) | ₱700 | ₱800 |

- A slot is "daytime" if its start hour is before 17, and "evening" if it is 17 or later.
- The 4 PM – 5 PM hour has no rate, so it is unavailable.
- Total = the sum of each selected hour's rate for the selected date.

## 4. Responsive and accessibility
- < 900px: single column, and the summary is no longer sticky.
- Calendar cells shrink to 36px tall. Court and slot grids reflow through `auto-fit` / `auto-fill`.
- Nothing may cause horizontal overflow at 360px wide.
- Every control is a real `<button type="button">`.
- Use `aria-pressed` for selected court and slot buttons, and `aria-selected` / `aria-current="date"` on calendar days.
- Use `aria-disabled` plus the `disabled` attribute for unavailable items.
- Keep a visible `:focus-visible` ring: 2px teal outline, 2px offset.
- Respect `prefers-reduced-motion` by skipping slide, pulse and morph animations.

## 5. Deliverables
1. The plan described at the top, before any edits.
2. The front-end implementation (markup, scoped CSS, client-side JS) in the plugin's existing structure. No back-end changes.
3. A short note on where in the front-end config to change the prices, the booking window, the court list and the Messenger username.
4. A quick manual test checklist covering:
   - weekday vs. weekend pricing
   - the 4 PM gap
    - merged time ranges in the summary
   - the Messenger link text
   - copy-to-clipboard
   - mobile layout at 375px
