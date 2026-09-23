---
description: Generate multiple self-contained HTML prototypes of an idea so distinct design directions can be compared side by side.
argument-hint: "<what to prototype>"
---

Generate multiple self-contained HTML prototypes of whatever the user describes,
so they can compare distinct design directions side by side.

## Arguments

`$ARGUMENTS` is what to prototype, e.g. `/prototype a settings page` or
`/prototype 3 a login modal`.

- If the arguments begin with a number, that's the count of options to generate.
- Otherwise default to **5** options.
- The rest of the arguments is the thing to prototype.

If no description is given, ask the user what they want prototyped before doing
anything else.

## Steps

1. **Restate the brief.** In one or two sentences, confirm what you're
   prototyping and how many options. Note any constraints you can infer from the
   surrounding project (existing CSS, framework, brand colors) — but each
   prototype must be a **single self-contained `.html` file** (inline CSS/JS, no
   build step, no external deps except CDN links if truly needed).

2. **Make the options genuinely different.** Don't generate five recolors of the
   same layout. Vary the actual approach — layout, hierarchy, interaction model,
   density, tone. Each option should represent a defensible, distinct design
   direction a designer might argue for.

3. **Write the files.** Create `prototypes/<slug>/` in the current working
   directory (slugify the brief). Write `option-1.html` … `option-N.html`. Then
   write an `index.html` in that folder that links to and/or iframes all options
   with a short caption for each, so the user can browse them in one place.

4. **Recommend one.** Per the global rule, pick the option you'd ship and say so
   first — name it, and give the one trade-off that decided it. Then briefly
   characterize each of the others (one line each: what it optimizes for, who
   it's right for).

5. **Offer to open it.** Tell the user they can run `open
   prototypes/<slug>/index.html` (macOS) to view them, or offer to open it for
   them.

## Notes

- Keep each file readable — a designer or developer should be able to lift the
  one they like straight into the project.
- Prefer semantic HTML and modern CSS (flexbox/grid, custom properties). Accessibility
  basics (labels, contrast, focus states) count as quality, not extra credit.
- This command produces throwaway exploration artifacts. Don't wire them into the
  app or modify project source.
