# QA Report

Use exact pass/fail evidence. "Looks fine" is not a result.

## Pages Checked
- index.html, services.html, contact.html — all checked live via local static server (`python3 -m http.server`) at mobile (375x812), tablet (768x1024), and desktop (1280x800).

## Audit Results
| Check | Result | Evidence |
|---|---|---|
| Contrast audit | PASS (after fix) | `.pipeline/qa/contrast-audit.js` run on all 3 pages. Initial run on index.html found 10 real violations: `.brand-text small`, `.section-head .kicker`, `.team-role`, `.quote-card cite` text using `var(--tan)`/`var(--brass)` on cream/white backgrounds (ratios 2.80–3.65, threshold 4.5). Fixed by adding darker text-only variables `--tan-ink: #7a4f26` and `--brass-ink: #6b5220` (kept original `--tan`/`--brass` for icons/borders/scrollbar) and repointing all 5 failing text-color declarations in `assets/css/style.css`. Re-run after fix: index.html 0 violations/51 checked, services.html 0/79, contact.html 0/41. |
| Upscale mobile | PASS | `.pipeline/qa/upscale-audit.js` at 375px: index.html 0 violations/4 imgs, services.html 0/10, contact.html 0/1. No brokenImages, no aspectMismatches. |
| Upscale tablet | PASS | Same script at 768px: index.html 0/4, services.html 0/10, contact.html logo only (native 263px, rendered 88px — no upscale). |
| Upscale desktop | PASS | Same script at 1280px: index.html 0/4, services.html 0/10, contact.html logo only, no upscale. |
| Broken images | PASS | `brokenImages: []` on every page/breakpoint combination checked above. |
| Aspect mismatch advisory | PASS | `aspectMismatches: []` on every page/breakpoint combination — no `object-fit: fill` distortion anywhere. |

## Manual Checks
| Check | Result | Notes |
|---|---|---|
| Text on photo | PASS | Hero (index.html) text sits over `linear-gradient(rgba(20,12,8,.15), rgba(20,12,8,.86)) + rifle-vintage.jpg`, bottom-aligned into the darkest part of the scrim, plus `text-shadow` on h1/lede — confirmed legible by screenshot at mobile width. Flagged by the automated script as `needsManualCheck` (gradient/background-image) per its documented limitation, not a violation. |
| Gradient/::before backgrounds | PASS | `.team-avatar` (white "CT"/"L"/"A" initials on `linear-gradient(135deg, var(--tan), var(--oxblood))`) also flagged as `needsManualCheck`. Font is bold ~25.6px so WCAG large-text threshold (3.0) applies; white text against the lightest corner of the gradient (tan, contrast ≈4.5) and the darkest corner (oxblood, much higher) both clear 3.0. |
| Image/content match | PASS | Every asset manifest entry in BUILD_BRIEF.md matches its adjacent copy (shooting photos next to shooting copy, equestrian photos next to equestrian copy, etc.). No image reused in a mismatched context. |
| Fabricated claims | PASS | Re-verified chris@saddlersden.co.uk and the "Copyright © 2018" / broken base64 team-photo placeholders live on https://www.saddlersden.co.uk/ during this QA pass (2026-07-11) — still present, pitch angle still valid. All facts trace to BUILD_BRIEF.md's Allowed Facts table; no new claims added during QA/fix pass. |
| Mobile layout | PASS | Mobile nav toggle opens/closes correctly (`nav-links.open` + `aria-expanded` toggle verified). Course table on services.html scrolls horizontally inside `.scroll-x` wrapper (640px content in a 327px viewport slot) without breaking page-level layout (`document.body.scrollWidth` stayed at viewport width). Contact page's longest real strings (full address, email) fit their boxes with no `scrollWidth > clientWidth` overflow. |

## Blocking Issues
| Issue | Evidence | Required fix |
|---|---|---|
| (none remaining) | | |

## Advisory Issues
- None.

## Fixed Verification
| Issue | Fix | Recheck result |
|---|---|---|
| Text-color contrast failures on `.brand-text small`, `.section-head .kicker`, `.team-role`, `.quote-card cite`, `.contact-line .label` (var(--tan)/var(--brass) on cream/white, ratios 2.80–3.65 vs 4.5 required) | Added `--tan-ink`/`--brass-ink` darker text-safe variables in `assets/css/style.css` `:root`, repointed the 5 failing declarations to use them; left `--tan`/`--brass` in place for icons, borders, and the scrollbar thumb (non-text, not subject to the text-contrast gate) | Contrast audit re-run on all 3 pages: 0 violations, 0 needsManualCheck beyond the two expected gradient/background-image cases |

## Verdict
PASS

## Independent Reviewer Re-Verification (2026-07-11)
Re-ran both shared audit scripts (`.pipeline/qa/contrast-audit.js`, `.pipeline/qa/upscale-audit.js`) live in-browser, independently, against all 3 pages at all 3 required breakpoints (375/768/1280) — 9 page/breakpoint combinations total, images force-loaded past `loading="lazy"` before each measurement.

| Check | Result |
|---|---|
| Contrast, all 9 combinations | 0 violations each. Only `needsManualCheck` items: hero eyebrow/h1/lede/CTA text over `section.hero`'s gradient+photo background (index.html), page-hero h1/p over `section.page-hero`'s gradient (services.html, contact.html), and the 3 `.team-avatar` initials over their gradient — all expected, all previously flagged. |
| Upscale/broken/aspect, all 9 combinations | 0 violations, 0 brokenImages, 0 aspectMismatches, 0 notYetLoaded (after forcing lazy images to load) on every page/breakpoint. |
| Manual math check on the `needsManualCheck` items | Computed contrast by hand for the worst case (scrim blended over an assumed light `rgb(200,200,200)` patch of the hero photo, not just the darkest corner): white h1 ≈14.8:1, tan-light eyebrow ≈6.8:1, cream lede ≈12.0:1 — all clear the 4.5:1/3:1 thresholds with large margin. Page-hero gradient (oxblood-dark→oxblood) white text: 11.3–15.1:1. Team-avatar white-on-gradient: 3.65:1 at the lightest (tan) corner, clears the 3.0:1 large-bold-text threshold; oxblood corner far higher. |
| `--tan-ink`/`--brass-ink` fix verification | Computed by hand: `--tan`/`--brass` (icon-only uses remaining in the CSS: `.credential-strip .item svg`, `.contact-line svg`, `details.faq summary::after`) are non-text UI elements (3:1 gate, both ≈3.1–3.2:1, passes) — confirmed these are icons via source inspection, not leftover text-color regressions. `--tan-ink`/`--brass-ink` (the actual text-color fix) measure 6.18:1 and 6.44:1 against cream — comfortably clears 4.5:1. The fix holds. |
| Mobile layout | Nav toggle open/close verified via direct `.click()` + `aria-expanded`/`.open` class check — works. Course table `.scroll-x` wrapper: 640px content in a 327px slot, `document.body.scrollWidth` stays at 375 (no page-level overflow). Contact page's three `.contact-line` blocks (phone, email, full address) all show `scrollWidth === clientWidth` — no text overflow. |
| Image manifest vs. disk | Re-measured all 10 image files independently (PIL) — every native size matches BUILD_BRIEF.md exactly (e.g. bridle-horse.jpg/saddle-horse.jpg both 302×331, rifle-vintage.jpg 1157×768). `.small-native img` CSS caps the two small equestrian shots at max-width 340px against a 302px native width — 1.126x, under the 1.3x gate. |
| Image/copy match | Re-read every `<img>`+heading pairing across all 3 pages directly from source — every image sits next to matching copy (shooting photos under shooting headings, equestrian under equestrian, accessories under accessories). No mismatches. rifle-vintage.jpg is deliberately reused (Home hero + Services shooting card), consistent with BUILD_BRIEF's note, not a new mismatch. |
| Course pricing vs. live source | Re-fetched https://www.saddlersden.co.uk/course-information independently — all 5 course names/durations/prices (£255/£475/£490/£520/£130) match the demo exactly. |
| Pitch-angle facts vs. live source | Re-fetched https://www.saddlersden.co.uk/ independently — footer still reads "Copyright © 2018 Saddlers Den - All Rights Reserved", team section still shows the broken `data:image/gif;base64,...` placeholders for Chris/Louise/Amy, and chris@saddlersden.co.uk / 01704 228370 / 60 Norwood Avenue both still shown. Pitch hook in BUILD_BRIEF.md is accurate. Demo's own footer correctly self-identifies as "Concept redesign — not the business's live site" with a live `current-year` script (not a hardcoded stale year), so the demo does not inherit the 2018 bug it is pitching against. |
| Fabricated-facts scan | Grepped all 3 pages for founding-year, opening-hours, and named-testimonial patterns — none present, matching BUILD_BRIEF's Do Not Claim list. `course-testimonials` quotes remain unattributed ("Course evaluation form"), consistent with the source. |

**Environment note, not a site defect:** mid-review, a screenshot briefly rendered a different business's stylesheet (navy/karate palette) because a concurrent session's `python -m http.server` grabbed the same local port out from under this review's server. Re-confirmed via `curl`/`ps` that this was a local port collision in the shared review sandbox, not a bug in Saddlers Den's code; re-served on a clean port and re-ran the full 9-combination audit plus both screenshots against the confirmed-correct server, with identical (passing) results to the first pass. No action needed on the site itself.

**Reviewer verdict: PASS.** No blocking issues found. No advisory issues found beyond what the builder already logged (none). This is the second independent verification of this build (build-agent QA + this review) and both agree.
