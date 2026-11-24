# Page draft process

Shared process for `/page-draft` in Claude Code and `$page-draft` in Codex.
One capable writer owns the complete page. The page record (named in the
project's `docs/agents/pages.md`) holds the page's stage and handover
notes; the page plan holds the brief; Sanity holds the content; GitHub holds
separately approved code work.

## Project facts come from `pages.md`

Every project built from this starter has `docs/agents/pages.md`. It names:

- **Page record:** the Basecamp card table and its link, how to find a
  page's record, the stage names, and which stage a draft moves to at
  handover.
- **Taking a page:** the project's claim method, or the rule that Ovi or a
  coordinating session assigns each page.
- **Handover notes:** where the brief summary, checklist, proposals and
  backup path are recorded.
- **Plan:** where each page's plan lives.
- **Readers and rules:** the audience sources, the scope of the current
  release, the message rules, the shared wording, the confirmation marker,
  and the topic map.
- **Sanity:** the project id and dataset (or a pointer to
  `docs/agents/sanity-cli.md`).
- **Blocks** (when present): which blocks to use, which not to use,
  which are being built, and how to treat missing photos.

Use only this project's facts. Never copy ids, names, readers, or rules from
another project. If a fact you need is missing from `pages.md`, stop and
tell Ovi.

## Scope and outcome

Write a complete, ambitious proposed page from its plan, then map it to
existing presentation layouts. Content comes first. Freely propose the
promises, details, numbers, and experiences appropriate to the page, inside
the scope of the current release in `pages.md`. Factual accuracy is a later
client pass: mark each unconfirmed claim with the project's confirmation
marker and put it on the page's checklist. Existing source facts are useful
inputs, not a limit on what the draft can propose. The page is not
published; only its page-owned supporting documents, such as FAQs, are.

The result is a complete Sanity draft, its page record moved to the
handover stage, and a traceable confirmation checklist. A temporary layout
limitation is acceptable when recorded. Missing content and failed writes
are not. Ovi is the editorial reviewer; do not create a review agent or
review loop. Content drafting is content-only work, so it earns no code
review either.

Content work requires no dedicated branch or worktree. Read repository
instructions and block definitions from an available checkout, including
main, without changing it. No schemas, queries, frontend, generated types,
code commits, pushes, GitHub plans, PRs, or release checks belong to this
task. Propose block changes in the handover notes for Ovi to decide on
later.

## Start and take one page

Accept a page path, an old-site URL, or a link to the page's record.
Resolve the record, path, title, plan, and Sanity document id. Normalize
the path without a leading slash for the slug. A missing page document can
be created as a draft after the backup.

Before taking the page, confirm that Sanity MCP can read and mutate the
target dataset, and that the page record's system is reachable with the
tools `pages.md` names. If a required connection is unavailable, report the
specific missing capability and stop. Do not replace MCP mutations with
local seeds or an invented CLI write path.

Take the page with the method `pages.md` gives. Work on one page per run.
Never take a page another run holds, and never infer ownership from a
branch name or replay a cached mutation. Resume only a page this session
holds, or one Ovi hands over to you; inspect the record and the current
Sanity draft to establish what remains.

## Brief and selective research

Read the page's plan in full: reader, job of the page, sources to merge,
outline, proof and media needed, links, search, and open questions. Read
the reader sources, message rules, shared wording and topic map that
`pages.md` names. The plan replaces writing a separate brief; record only
decisions that differ from the plan. A plan serves one page. If it covers
more than one page, or says another page is "the same with small changes",
stop and tell Ovi: each page needs its own plan.

Read the old page where helpful. `pnpm legacy:page <slug>` prints legacy
content stored on the page document, when the project migrated it; the
project's other old-content sources are listed in `pages.md`. Choose
further research for this page, not from a fixed roster. Delegate only
substantial independent questions and keep the answers concise. Do not
repeat research to verify proposed claims. Defer layout and asset selection
until the content is written.

The unit of delivery is this page. A topic another page owns (per the topic
map) gets at most a short summary and a link. Ask Ovi before searching
other pages for matching defects, claim occurrences, or wider coherence
problems. Do not start a site audit.

## Set the page strategy

Settle the page strategy before writing. Start from the plan's Reader and
Outline. Work out any part the plan lacks here, and record it in the
handover notes as a decision that differs from the plan.

- **Awareness.** For each way readers arrive (a QR card, a search, a link
  from another page), name their stage of awareness: unaware,
  problem-aware, solution-aware, product-aware, or most aware. The largest
  group sets the lead.
- **Lead.** The stage decides what the page opens with:
  - unaware: a situation the reader recognizes as their own;
  - problem-aware: the problem in the reader's words, and proof that we
    understand it;
  - solution-aware: why this kind of solution works, and why this one;
  - product-aware: recognition of what they heard about, their hope, their
    worries answered, proof, and an easy first step;
  - most aware: the offer and the action.
  A reader who lives with the problem every day gets the hope and the
  answers, not an explanation of the problem.
- **Hope and worries.** Name the reader's main hope in their own words.
  Rank their worries, including unspoken ones, such as the fear that the
  offer is out of their financial reach. Answer each worry where the reader
  meets it.
- **Cost.** On a short page, or for a reader who arrives cold, answer the
  cost worry early with what reassures: the lowest price or the help
  available, as the message rules in `pages.md` allow.
- **Good impression.** Every reader, including the one who does not act,
  leaves respected and welcome to come back. The page treats the offer as
  within reach of the families it serves.
- **Order.** Derive the block order from the lead, the ranked worries and
  the proof. The usual order: hope, worries answered, a small first step,
  proof, what happens next, the main request, then links that lead away.

The strategy is done when it names the stage for each arrival route, the
lead, the hope, the ranked worries, and the order.

## Write the complete page

Write without regard to available layouts. Follow the page strategy: serve
the reader's questions in its order with complete headings, body copy, and
calls to action.
Every button points to a page or feature that exists in the current
release. Use the shared wording exactly where it applies. Run `unslop` on
the writing; Ovi's later review is not a reason to leave known repetition,
fragments, or missing material.

Clearly mark each unconfirmed promise wherever it appears. If a field
cannot carry a visible marker, place one immediately beside its displayed
content and name the exact field in the checklist. Metadata claims also
need markers and checklist locations. Invented example quotations must be
visibly identified as proposed placeholders, not attributed as genuine
testimony. Preserve useful proposed detail instead of deleting it for lack
of a source.

## Match presentation layouts

After writing, read the Blocks part of `pages.md` when it has one (older
sites head it "Sections"). It names the blocks to use, the blocks not to
use, and the blocks being built, and it overrides the general rules in this step. Then read
`docs/agents/page-builder.md` and inspect only the existing block
definitions needed to select layouts
(`studio/schemas/blocks/`). General blocks are content-agnostic
presentation patterns; their names are not topics. Specialized blocks
keep their actual purpose.

Choose by presentation mode, fields, capacity, and existing rendering
support. Aim to reuse existing blocks nearly all the time. Preserve the
complete copy with multiple instances or the closest flexible layout when
necessary. Do not shorten, omit, or rewrite ideas merely to fit a block.
Preserve meaningful emphasis, links, ordering, and required fields.

Choose existing assets after the copy. Use supplied or stored descriptions
for alt text; record uncertain image suitability for Ovi. Leave an optional
image slot empty and note the gap rather than create a broken asset link.

A new block or schema modification is a proposal only. Record the
affected content, current fallback, specific presentation limitation, and
benefit of the proposed change in the handover notes. Ovi decides whether
to start separate design or development work. Make no schema or frontend
changes here, and do not create a GitHub issue for an unapproved proposal.

## Save directly to Sanity

### Rules for working in parallel

The dataset is shared. Only the operator mutates the page it holds and that
page's own supporting drafts. Keep deterministic, page-scoped ids for new
FAQs or other supporting documents. Read shared references, but do not
overwrite them. Leave `homePage`, `navigation`, `settings`, `footer`, other
pages, asset documents, and global configuration unchanged.

Before the first dataset write, name and verify the project and dataset from
`pages.md` against the MCP target; verify rather than assume. Create a
timestamped export under `backups/` and run `gzip -t` on that exact archive.
Follow `docs/agents/sanity-cli.md` for the CLI and its token. Use
`sanity datasets export <dataset> <absolute-backup>.tar.gz --raw` for
document-only work. If the task changes asset references, use a full export
including assets. A failed export or verification stops writes. Backups are
recovery archives only.

Use Sanity MCP to read current document ids and revisions, then apply
scoped mutations to drafts. For an existing published-only page, create the
draft from its current content without altering the published version.
Preserve fields outside the task. Use revision preconditions when patching
existing drafts; create new drafts without replacing a concurrently created
document. On a conflict, re-read the document and reconcile the scoped
change. Do not blindly retry or overwrite another writer's work.

Use registered block types and actual schema fields. Portable Text
blocks, spans, list items, and marks need valid keys and references.
Page-owned FAQs and other supporting documents are published as soon as
they are created, in the same session, so the page draft can hold normal
references to them. Never hand over with supporting drafts left for Ovi to
publish by hand. The page itself stays a draft. If MCP lacks the operation
needed for a safe write, stop and report it instead of using an unsafe
replacement.

Write and fix the dataset directly. No seed files, repeated imports,
content artifacts, or local progress files are required or created. The
Sanity draft is the content result.

## Check storage and hand over

Read the affected drafts back through MCP, including referenced page-owned
documents. Check that all intended copy survived in order, required fields
and keys are present, references resolve in the draft perspective, and
every proposed claim has its marker and checklist entry. Then read the
block order as a shape: the page ends with one closing call-to-action
block, and no two call-to-action blocks (closing bands, nudges,
director prompts) sit next to each other. The block before the closing
band is proof or content, never another ask. Two asks in a row read as a
false ending and make the primary buttons compete; fold the extra prompt's
line into the nearest content block and remove its band before you hand
over. Then run Studio's own
validation on the draft, because MCP writes skip it: `pnpm page:check
<document id>` where the project has it, else export the draft to ndjson
and run `pnpm --dir studio exec sanity documents validate -y --file
<ndjson> --level warning`. Hand over at zero errors and zero warnings. Fix mutation or structural errors; distinguish
pre-existing warnings from new failures. This is a completeness check, not
a factual or independent editorial review.

No browser, screenshot, server-render test, typecheck, or build gate is part
of content drafting. The review link is the hosted Studio's Presentation
view of the draft, built as "Page link on the card" in `pages.md` says; it
needs no local server. Never hand over a localhost link: Ovi reviews from
the card, often with no local server running. If the project has no hosted
Studio, or its Presentation view cannot show the draft, stop before
handover and tell Ovi; do not claim a working link.

Record the handover notes where `pages.md` says:

- Decisions that differ from the plan, and the Sanity page id.
- The complete confirmation checklist.
- Layout proposals, temporary presentation limitations, and media gaps.
- Backup path and ids of documents changed.
- The review link (the same one the card carries).

Post the page strategy comment on the page record, where clients read it
(see "Comments on a card" in `pages.md`):

- **Who arrives here:** the arrival routes and what these readers already
  know.
- **What they need from the page.**
- **The order of the page:** each part and why it is there.
- **Special choices:** for example, why the lowest price shows early.
- **Please check:** the facts that carry the confirmation marker.

Write it for the client in plain words: the reasoning in the page's own
terms, with methods applied and never named, and only figures the page
itself shows. Ids, agent notes, questions and promises belong in the
handover notes. Post it once and read it back. A later run that changes the
strategy posts a short "Updated:" comment with only the change.

Move the page record to the handover stage only after the content,
checklist and strategy comment are saved and the review link is on the
record ("Page link on the card" in `pages.md`; read the description first,
since an update replaces it). Release the page if the project's claim method needs
it. Tell Ovi the record link, the review link, and any presentation
limitations. Content ready and presentation work proposed are
separate outcomes; a layout proposal does not block handover. The next
skill is `/page-polish`.

## Pauses, failures, and later corrections

Honor a requested stop after reading the plan or after writing. Record
progress in the handover notes on a deliberate pause. A later session reads
the notes and current draft, then takes the page again before writing. On
an unexpected failure, record the step, cause, saved document ids, and next
action in the handover notes when possible, and keep the page's stage. If
the page record's system itself fails, report the failure directly to Ovi.
Never report incomplete content as handed over.

When Ovi reports a quality failure, correct this page and identify the
cause. Ask before searching other pages for the same failure. Broader
coherence, client fact confirmation, and block development are separate
passes. Do not start them as cleanup for this run.
