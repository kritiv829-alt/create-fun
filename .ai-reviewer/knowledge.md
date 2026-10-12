# create-fun reviewer notes

## Architecture

This is a Vite React 18 application written in strict TypeScript, with `src/App.tsx` acting as a large application/gallery composition root. Domain widgets live in `src/components`, editing flows in `src/editors`, alternate presentations in `src/views`, reusable UI primitives in `src/ui`, and domain-specific charts in `src/charts`. The package is private and has no runtime dependencies beyond React and React DOM.

## Conventions

- Use functional React components with named TypeScript interfaces and exports; chart files such as `src/charts/AccountChart.tsx` define `AccountPoint`, `AccountChartProps`, and `AccountChartMode` before the component.
- Components commonly provide both named and default exports, as shown by `AccountChart` and `AlbumChart`.
- Domain files are organized by feature noun and presentation role: e.g. `OrderCard`, `OrderEditor`, and `OrderGrid` are imported from separate `components`, `editors`, and `views` directories in `src/App.tsx`.
- Keep reusable UI components under `src/ui` and import them through relative paths. `src/ui/README.md` documents the available primitives and their expected usage.
- Controlled state belongs to the consuming widget. `src/ui/README.md` specifically requires consumers to own state for controls such as `SearchInput`, `Pagination`, and `ToggleButton`.
- Use native HTML semantics and explicit button types. Shared `Button` defaults to `type="button"`; callers must set `type="submit"` for form submission (`src/ui/README.md`). Charts also use explicit `type="button"` in their mode toggles.
- Accessibility is part of the shared-component contract: use descriptive labels, accessible headings, live loading/error status, and status wording rather than color alone (`src/ui/README.md`).
- TypeScript must satisfy strict checks, including `noUnusedLocals` and `noUnusedParameters` (`tsconfig.json`). The supported validation command is `npm run build`, which runs `tsc --noEmit` followed by `vite build`.

## Intentional non-standard choices

- The application deliberately has a very large, explicit import/composition surface in `src/App.tsx`; do not flag the absence of dynamic discovery or route-based lazy loading without a demonstrated requirement.
- Chart implementations are intentionally repeated per domain (`src/charts/AccountChart.tsx`, `AlbumChart.tsx`, `AlertChart.tsx`, etc.), with domain-specific point types, seeds, defaults, and CSS class names rather than a shared generic chart abstraction.
- Charts use deterministic local pseudo-random series generators, not external data or a randomness package. Their `generate*Series` functions use a fixed seed and linear-congruential calculation.

## Watch out for

- New code must not introduce unused imports, locals, or parameters; strict compiler settings will fail the build (`tsconfig.json`).
- Preserve stable keys and unique identifiers when rendering lists. `src/ui/README.md` requires unique select option values and definition-list item IDs; chart points use `p.label` as their key.
- Do not make controlled shared inputs internally stateful; pass value and change handlers from the consuming widget (`src/ui/README.md`).
- Check accessibility for new interactive or visual-only UI. SVG charts currently expose `role="img"` and `aria-label`, but hover-only data disclosure in files such as `src/charts/AccountChart.tsx` is not keyboard accessible; similar additions should provide an accessible equivalent.
- Keep form buttons from accidentally submitting forms by using the shared default or explicit `type="button"`; only submit actions should use `type="submit"`.
- Validate chart changes against empty or single-point data and unusual dimensions; chart geometry in `src/charts/*Chart.tsx` assumes numeric dimensions and uses series-derived scaling.