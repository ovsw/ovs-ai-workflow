# Page plan process

The first page skill. Each page on the board gets a content plan in its
own Basecamp document. Then `/page-draft` writes the Sanity draft from that
plan, and `/page-polish` fixes block fit and photos.

## Project facts come from `pages.md`

Read `docs/agents/pages.md` in the repository. It names the page record
(the Basecamp card table and its stages), the release scope, where plan
documents live, the readers (segments and avatars), the marketing plan,
the old-site content, the message rules, the shared wording, the
confirmation marker, and the topic map. Use only this project's facts. If a
fact you need is missing, stop and tell Ovi.

## Scope

- Plan only the pages in the current release, as `pages.md` defines it.
- Treat the current readers as correct and complete, unless `pages.md`
  says otherwise.
- A plan uses the content the release allows, rewritten for the reader. It
  never links to a page or feature outside the release. A later
  improvement may be named in one line under "Later"; it is not planned.
- Unconfirmed facts never block a plan. Mark them with the project's
  confirmation marker and list them under "Open questions".
- Keep each plan short enough that Ovi can review it in a few minutes.

## Topic map first

Before the first plan, check that the topic map named in `pages.md`
exists and covers the release. If it does not, write it first, as one
pass over the whole release:

1. **Topic ownership:** topic → the one page that holds the full detail →
   pages that only summarize it in one or two lines and link to the owner.
2. **Old page destinations:** every old page → destination page(s), what to
   keep, what to drop. Flag conflicting facts between old pages instead of
   copying them.
3. **Gaps:** topics a reader needs that no page owns, with a suggested owner
   from the release's pages. Do not invent new pages.

Plans follow the map: a topic another page owns gets at most a short
summary and a link.

## Plan document template

Write in plain, short sentences.

1. **Page:** title, path, parent, section, page type, status (new, merge,
   rewrite, keep).
2. **Reader:** the segment and avatar. What they feel and want when they
   land here, the questions they bring, and their doubts. Where they come
   from (search, a QR card, another page). Then the page strategy, by the
   rules in "Set the page strategy" in `../page-draft/process.md`: the
   stage of awareness for each arrival route, the lead it sets, the main
   hope in the reader's words, and the ranked worries, unspoken ones
   included.
3. **Job of the page:** from the record, sharpened. The one action we want
   (main button) and the second choice. Which funnel stage it moves them to.
4. **Marketing plan steps served:** each code and what the page does for it.
5. **Sources to merge:** each old page, what to keep, and what to drop, as
   the topic map says. Content that moves to another page names that page.
6. **Outline:** the blocks in the order the page strategy sets. For
   each: its purpose (the hope or the worry it serves), the key points or
   draft headline, and the proof it needs (a story, a number, a photo, a
   quote).
7. **Proof and media needed:** photos, video, stories, numbers, and who can
   supply them.
8. **Links:** pages that link here, and pages this page links to. Every
   button uses the shared wording's label and target.
9. **Search:** the main search query (if any), page title, meta description.
10. **Open questions:** the card's open subtasks, plus every unconfirmed
    item.

One plan document serves one page. We do not use shared plans: never write
one plan for several pages, and never write "the other pages are the same
with small changes" (Ovi, 2026-10-07).

A path with `{…}` is not a template because of its shape. It is a
template only when every instance has the same reader, the same job and
the same layout, and only its data changes, like `/blog/{post}`: one
plan, for the template. Pages that share a URL prefix but serve different
readers, channels or stages of awareness, like landing pages
(`/go/events`, `/go/doctor`), are separate pages: each has its own card,
its own plan and its own page strategy. The `{…}` path is only a URL
pattern; it gets no card and no plan. If the record shows such a path as
one card, stop and tell Ovi.

Legal pages get a short plan: what the page must cover and who writes or
checks it.

## Rules every plan follows

- The message rules and shared wording in `pages.md`, exactly.
- No criticism of the client's current site or past work; Ovi gives that on
  a call first.
- Never name the experts or authors whose marketing methods we use.

## How to make the document

Write the plan as HTML in a temp folder (never in the repo), in the shape
the "Plan documents" rule in `pages.md` gives: page facts first, then the
plan. Basecamp keeps `h1` to `h3`, `div`, `strong`, `em`, links, nested
`ul` / `ol` lists, tables, `blockquote`, `hr` and `pre`. Create it as
Clark in the plan folder, titled with the page path:

```sh
basecamp --profile claude files documents create "<page path>" - \
  --in <projectId> --folder <plansFolderId> --json < plan.html
```

First list the folder and match titles, so no page gets two plans:
`basecamp --profile claude files documents list --in <projectId> --folder
<plansFolderId> --all --json`. Run each create once; after an unclear
result, list the folder again before you retry. Read the document back
(`basecamp --profile claude files show <id> --in <projectId> --json`) to
check that the text is there.

To revise a plan, replace its body in place, which keeps its link and its
comments: `basecamp --profile claude files update <id> --content - --in
<projectId> < plan.html`. The new body replaces the whole old one, so read
the document first and start from its current text; Ovi may have edited
it.

## Check, then update the record

After all plans in a batch exist, check them as one set before reporting:
release scope, topic ownership, message rules, shared wording, buttons that
point outside the release, and contradictions between plans. Fix what you
find, in place.

Then update each page's card: add the plan link as "Plan link on the
card" in `pages.md` says, if the card does not have it yet, and move the
card as "Stage moves" says. One session updates the cards, not each plan
writer.
