# MeatFest site — agent digest

Static HTML5 UP "Arcana" template site, no build step, hosted via GitHub Pages
(custom domain `meatfest-ec.com` per `CNAME`). Flat HTML files at repo root,
`images/`, `assets/{css,js,sass,webfonts}`. Edit HTML directly — there is no
templating layer.

This file is a running log of *decisions*, not a feature list. Keep entries
short. Update it when a new convention gets established; don't let it drift
from what's actually in the code.

## Forms (un-static.com)

- All forms POST to `https://forms.un-static.com/forms/<id>` — a free static-site
  form backend. **Each form has its own id/endpoint** — don't reuse the footer
  "Get In Touch" form's id for a new form.
- The RSVP form's submissions feed a local n8n workflow → Airtable. The
  post-submit redirect (e.g. to `thank-you.html`) is configured in the
  un-static dashboard per form, not in the HTML.
- Honeypot pattern (per un-static's own docs): a field named `website`,
  hidden via `style="display:none"` on a wrapper (not `type="hidden"` —
  some bots skip those), `tabindex="-1"`, `autocomplete="off"`,
  `aria-hidden="true"` on the wrapper. See `6d656174/rsvp.html`.

## RSVP page: `6d656174/rsvp.html`

- Intentionally unlisted: not in nav, `<meta name="robots" content="noindex, nofollow">`.
  `6d656174` is the ASCII bytes of "meat" in hex — a cheeky obfuscation, not
  real access control. Since it's one directory deep, all its local links/asset
  paths are `../`-prefixed.
- Yes/No questions are pill buttons: `<label class="choice">` wraps a radio
  input that's visually clipped (not `display:none`) so it stays keyboard/AT
  accessible; JS toggles a `.selected` class per radio-name group on `change`.
- Conditional follow-up questions use `.conditional-field` > `.conditional-field-inner`,
  animated with a CSS grid `grid-template-rows: 0fr → 1fr` transition (not
  `max-height`) so blocks of any height — including a whole nested stack of
  questions — animate open/closed without a hardcoded height guess.
- `required` on gated fields is **never static in HTML** — it's toggled by JS
  only while the field's own `.conditional-field` is visible, via a
  `.gated-required` marker class + `setVisible()`'s
  `f.closest('.conditional-field') === el` scoping check. This matters: a
  `required` field that's visually hidden but still in the DOM makes Chrome
  silently block submission with no visible tooltip. The scoping check also
  stops an outer wrapper opening from prematurely requiring inner
  not-yet-revealed sub-questions.
- Current gating chain: `attending` (Yes/No) gates everything else → within
  that, `bringing_dish` gates a dish-name field, `coming_early` gates
  `own_apparatus`, which gates `transport_support`.
- Field names collected: `name`, `email`, `attending`, `party_size`,
  `bringing_dish`, `dish`, `coming_early`, `own_apparatus`,
  `transport_support`, `allergies` (plus the `website` honeypot). If you add/
  rename a field, whatever consumes the un-static webhook (n8n workflow) needs
  the matching field name.

## Images

- Filenames often carry a size suffix from how they were exported —
  `-Small` (~640×480), `-Medium` (~1600×1200), `-Large` (bigger). New photos:
  EXIF-rotate, resize to ~1600px on the long edge, save as JPEG quality ~85,
  and follow the `<stem>-Medium.jpg` naming pattern.
- `.image.featured img` is `width: 100%` (responsive, no forced crop/aspect),
  so differing aspect ratios between photos are expected and fine.
- `<mark>` is reset to transparent background / inherited color by the base
  stylesheet (`assets/css/main.css`) — a bare `<mark>` won't visually
  highlight anything. Use a scoped class (e.g. `mark.callout`, see
  `parking.html`) with explicit `background-color`.

## Verification

No automated test suite. For anything touching JS/animation/validation
logic, verify with a real browser rather than reading the code and assuming:
spin up `python -m http.server` from the repo root and drive it (headless
Edge/Playwright via a background agent has worked well), checking console
errors, screenshots, and native HTML5 validation behavior specifically.

## Workflow

- Only commit/push when explicitly asked ("commit and push"). Commit messages
  end with `Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>`.
- HTML files are tab-indented. If an `Edit` `old_string` match fails, suspect
  whitespace — check with `python3 -c "print(repr(open(f).readlines()[n]))"`
  rather than guessing at indentation.
