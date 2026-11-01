# kb — the support knowledge base

**kb** = knowledge base. What the support desk knows about how our products break, written
down where the code can keep it honest.

One desk takes reports for every product we run: `sched-prod` (calendar invites for about
300k people) and the rest. When something
goes wrong, the reports reach the desk first. This kb is how the desk triages them, what each
incident turned out to be, and why we decided what we decided.

## What's here

```
src/routes/kb/      the pages (.svx: Markdown with components)
  triage/           how the desk handles incoming reports
  incidents/        one page per big ticket
  decisions/        numbered decision records
src/lib/kb/         components and the logic they run
src/lib/layouts/    the frame every page shares
src/lib/data/       exports from the ticket system
```

## Conventions

- Words in the page, logic in `src/lib` (imported as `#lib/…`). A number that can be computed belongs in a component.
- Every page starts with frontmatter: `title`, `owner`, `status`, `reviewed`.
- A decision is never edited after it is accepted. A new record replaces it.
- A page you add is signed: its frontmatter carries your `signature` and it ends with `<RenderStamp {signature} />`, so the desk knows who wrote it.

## Owners

Support desk. Decision records: whoever signed them.

---

Exported 2026-10-04 from the `support` repo, `apps/kb/` → `src/`.

## #4471, split (added 2026-11-01)

The desk merged 412 reports into #4471 on subject line. They hold three problems, each now an
issue in this repo with the reports it covers:

| problem | reports | issue |
|---|---|---|
| `parser`: shared parser reads day-first dates month first | 374 | [#1](https://github.com/marcinzawada/support-kb-4471/issues/1) |
| `hint`: booking-v3 form hints `MM/DD/YYYY`, reads day first | 33 | [#2](https://github.com/marcinzawada/support-kb-4471/issues/2) |
| `one-day`: dates land one day early (time zone / DST) | 5 | [#3](https://github.com/marcinzawada/support-kb-4471/issues/3) |

- The page: [`src/routes/kb/incidents/4471-split/+page.svx`](src/routes/kb/incidents/4471-split/+page.svx), counts from [`src/lib/kb/split.ts`](src/lib/kb/split.ts).
- As the desk sees it: [`rendered/kb/incidents/4471-split.html`](rendered/kb/incidents/4471-split.html) (static build; `npm install && npm run build` makes it again into `build/`).
