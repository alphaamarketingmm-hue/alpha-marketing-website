ALPHA MARKETING M.M — Website Bundle (ULTRA-PREMIUM EDITION)
==========================================================

WHAT'S NEW IN THIS VERSION
This is the elevated, ultra-premium build. On top of the existing
full-page 3D gold-crystal engine and cinematic hero, it adds:

  - HERO: studio eyebrow, "Available For Select Brands" status, an
    animated "Ads | Content | Brand Growth" badge, and two floating
    glass cards (capabilities + the ordinary -> authority conversion line).
  - AI STUDIO ("Inside The AI Studio"): four interactive AI capability
    cards plus a live, self-animating control-room panel that cycles a
    Concept -> Script -> Visual -> Ad -> Growth pipeline.
  - INTERACTIVE BRAND SCORE QUIZ: five tappable questions, a gold
    progress bar, and an animated score ring with a tailored result and
    a "Get My Free Brand Audit" call to action.
  - CASE-STUDY MODALS: every Results card now opens a full-screen luxury
    case study (challenge, creative direction, deliverables, metrics,
    result, CTA). Click, Enter/Space, Esc and backdrop all work.
  - FLOATING "FREE BRAND AUDIT" CTA that scrolls to the quiz (auto-hides
    while the quiz is on screen).
  - TASTEFUL EXIT-INTENT modal ("Before You Go - Want A Free Brand
    Audit?"), shown at most once per browser session.

EVERYTHING IS STILL ONE SELF-CONTAINED HTML FILE
All CSS, JavaScript, the hand-written WebGL crystal engine, the optional
Three.js hero scene, and the logo images are embedded. It works fully
offline - just open index.html.

THE 3D (unchanged, still real and offline)
- A fixed full-viewport <canvas id="crystals"> sits behind all content.
- A custom hand-written WebGL renderer draws faceted gold icosahedron
  crystals that drift and rotate with scroll, with mouse parallax.
- Section backgrounds are semi-transparent so the crystals glow through
  while text stays crisp.
- Respects "reduced motion"; falls back to a soft gold sheen with no WebGL.
- The hero also loads Three.js on demand for the orbiting brand-core
  scene, and silently falls back to the 2D network canvas if the CDN is
  blocked or motion is reduced.

WHAT CHANGED IN THIS REVISION
- Redesigned the About section into an 'AI Brand Control Room':
  manifesto + interactive ALPHA Brand Engine (hover modules with
  tooltips), Brand Signal Scanner (scroll-activated qualitative
  meters), Creative Intelligence Matrix (2x2), floating microcopy
  labels, and a technology CTA.
- Removed the 'Selected Work' section entirely (per request).
- Fixed the custom cursor so it tracks the real pointer exactly.

FILES
- index.html         Complete website, fully self-contained. Works offline.
- logo-original.jpg  Your original uploaded logo (full resolution).
- logo-lockup.png    Full logo lockup, transparent background.
- logo-monogram.png  AM monogram only, transparent background.

TO VIEW
Double-click index.html (Chrome, Edge, Safari, or Firefox).

BEFORE GOING LIVE - QUICK CHECKLIST
- WhatsApp number: the contact link and the contact form both use
  wa.me/0000000000. Replace 0000000000 with your real number (country
  code, digits only). Once set, the contact form and every CTA route a
  pre-filled enquiry to WhatsApp; with no number set it falls back to
  email so leads are never lost.
- Case-study / quiz / impact numbers are representative examples - swap
  in your real figures when you have them (in the CASES, PACKAGES and
  impact data near the top of the in-page script).

PERFORMANCE NOTES
- 16 crystals on desktop, 9 on mobile (auto). Pixel ratio capped at 2x.
- The AI Studio panel and counters animate only while on screen and pause
  off-screen to save battery.
