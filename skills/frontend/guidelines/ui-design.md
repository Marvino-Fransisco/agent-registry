# UI Design Best Practices

This skill covers the full spectrum of visual and interaction design. When answering design questions, critiquing interfaces, or building UI, internalize these principles and apply them with intentionality.

---

## Quick Reference

| Topic | Reference File |
|---|---|
| Color theory, palettes, accessibility | `references/color.md` |
| Typography — fonts, scale, rhythm | `references/typography.md` |
| Spacing, layout, grids | `references/spacing-layout.md` |
| Visual hierarchy & composition | `references/hierarchy-composition.md` |
| Aesthetic styles & design language | `references/aesthetics.md` |
| Components & interaction patterns | `references/components-patterns.md` |
| Accessibility & inclusive design | `references/accessibility.md` |

Load the relevant reference file(s) before answering. For broad design questions, load all of them.

---

## Core Design Philosophy

### The Three Unbreakable Rules

1. **Intentionality over accident** — every visual decision must have a reason.
2. **Contrast drives clarity** — hierarchy, readability, and emphasis all come from contrast: size, weight, color, spacing, texture.
3. **Constraints create quality** — good design uses a limited, consistent system. Restrict yourself to a palette, a type scale, and a spacing scale.

### Design Process

1. **Understand context** — Who is the user? What is the goal? What emotion should this evoke?
2. **Choose a clear aesthetic direction** — commit fully. Half-hearted design looks unfinished.
3. **Build a system** — define colors, type, and spacing tokens before designing components.
4. **Design for the hardest case first** — long content, small screens, low contrast environments.
5. **Audit relentlessly** — zoom out and ask: does every element earn its place?

---

## The Most Common Mistakes

- **Too many font sizes** — use a strict scale (12/14/16/20/24/32/48px). Never deviate.
- **Inconsistent spacing** — use a spacing unit (4px or 8px) and only use multiples of it.
- **Low-contrast text** — WCAG AA requires 4.5:1 for body text. Most designs fail this.
- **Overusing color** — color should accent, not dominate.
- **Center-aligning body text** — center-align headings only; always left-align body copy.
- **Generic font choices** — Inter, Roboto, Arial make interfaces forgettable.
- **No visual hierarchy** — when everything is the same size/weight, nothing stands out.
- **Border-itis** — using borders everywhere instead of spacing/background to separate zones.
- **Icon without label** — icons are often ambiguous; pair with labels for clarity.
- **Cluttered layouts** — whitespace is not wasted space. It creates focus.
- **Inconsistent border-radius** — pick one radius style and apply it uniformly.
- **Shadow overuse** — one shadow style per elevation level. Never mix.

---

## Design Tokens: The Foundation

```css
/* Spacing (base-8 grid) */
--space-1: 4px;   --space-2: 8px;   --space-3: 12px;
--space-4: 16px;  --space-5: 24px;  --space-6: 32px;
--space-7: 48px;  --space-8: 64px;  --space-9: 96px;

/* Type scale (Major Third ratio ~1.25) */
--text-xs: 12px;  --text-sm: 14px;  --text-base: 16px;
--text-lg: 20px;  --text-xl: 24px;  --text-2xl: 32px;
--text-3xl: 40px; --text-4xl: 48px; --text-5xl: 64px;

/* Border radius */
--radius-sm: 4px; --radius-md: 8px; --radius-lg: 12px;
--radius-xl: 16px; --radius-full: 9999px;

/* Shadows (max 3 elevation levels) */
--shadow-sm: 0 1px 2px rgba(0,0,0,.06), 0 1px 3px rgba(0,0,0,.1);
--shadow-md: 0 4px 6px rgba(0,0,0,.07), 0 10px 15px rgba(0,0,0,.1);
--shadow-lg: 0 10px 25px rgba(0,0,0,.1), 0 20px 48px rgba(0,0,0,.12);

/* Transitions */
--transition-fast: 150ms ease;
--transition-base: 250ms ease;
--transition-slow: 400ms ease;
```

---

## Responsive Design

- **Mobile-first**: design the constrained case first, then progressively enhance.
- Breakpoints: 320px / 768px / 1024px / 1280px / 1536px.
- Touch targets: minimum 44×44px for any interactive element on mobile.
- Body font: never below 16px on mobile (prevents iOS zoom on focus).
- Test at 320px width — if it works there, it works everywhere.

---

## Design Critique Checklist

- [ ] Clear visual hierarchy (one dominant element per section)?
- [ ] Text contrast ≥ 4.5:1 body, ≥ 3:1 large text?
- [ ] Spacing consistent and grid-aligned?
- [ ] Font sizes from a defined scale?
- [ ] Color palette limited and purposeful?
- [ ] Interactive elements look interactive (affordance)?
- [ ] Enough whitespace for breathing room?
- [ ] Layout responsive at small sizes?
- [ ] Border radii consistent throughout?
- [ ] Aesthetic feels intentional and cohesive?

---

## Reference Files

Load these for deep guidance:

- `references/color.md` — Color theory, palettes, dark mode, emotion
- `references/typography.md` — Font pairing, scale, rhythm, readability
- `references/spacing-layout.md` — Grid systems, spacing rules, layout patterns
- `references/hierarchy-composition.md` — Visual weight, focal points, flow
- `references/aesthetics.md` — Aesthetic styles, design languages, mood
- `references/components-patterns.md` — Buttons, cards, forms, navigation, modals
- `references/accessibility.md` — WCAG, contrast, keyboard, screen readers