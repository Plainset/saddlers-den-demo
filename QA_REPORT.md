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
