# Page polish process

Shared process for `/page-polish` in Claude Code and `$page-polish` in
Codex. It follows `/page-draft` and takes a page whose
record sits in the polish-ready stage named in the project's
`docs/agents/pages.md`: the copy is written and every block is filled, but nobody has checked that each block is served by the
layout that fits it. One agent reviews the page as a senior web designer,
fixes what the existing Page Builder can fix, and reports the rest in chat.

## Scope and outcome

Content and layout choice only, inside the existing Page Builder. The agent
may move copy between blocks, swap a block for a better one, swap photos,
trim and dedupe copy, reorder blocks, and publish. It keeps the writer's
voice and the page's argument: no rewrite of what the page says, no new
claims, no removed confirmation markers (the marker is named in `pages.md`). Schema, GROQ, frontend, and
generated types stay untouched; a new block is a proposal in chat.

The result is a fresh page draft that differs from the newly published
baseline only by the polish changes, matching supporting documents
published, a before-and-after set of block screenshots, a change note in the
handover notes `pages.md` names, and a chat report with the block
verdicts, the proposals, and the links for Ovi's visual check. The page
record keeps its stage unless `pages.md` says otherwise; Ovi's own look at
the screen is the verdict.

Ovi's standing instruction for this skill: publish the page's current draft
and every draft-only document it references. Claims, offer details, and
placeholder quotes are the client's to verify in their collaboration pass,
where they also learn Studio; realness is not the agent's concern here.

## Take the page

Read `docs/agents/pages.md` first. Use only this project's facts; never copy
ids, names, or rules from another project. If a fact you need is missing,
stop and tell Ovi.

The input is a link to the page's record, or a page path as fallback. With
no input, take the first page in the polish-ready stage that no other run
holds, if `pages.md` allows untargeted runs. Resolve the record to the page
path and the Sanity page id before anything else: the record or its
handover notes carry both. If either is missing, find the page in Sanity by
its slug and record what you found in the handover notes.

Confirm the resolved page exists in Sanity (`*[_type=="page" && slug.current==$slug]`
in the `raw` perspective) before taking it. Take it with the method
`pages.md` gives. Read the page's plan and its draft handover notes: the
decisions, the layout proposals the writer recorded, and the checklist.
A plan serves one page. If it covers more than one page, or says another
page is "the same with small changes", stop and tell Ovi: each page needs
its own plan.
Read the page strategy comment on the page record too: the order it
explains is the order you judge against.
Read the draft as a reader with `pnpm page:text <slug>` and as structure
through Sanity MCP in the `raw` perspective: every block's type, key,
background, item counts, and image references.

## Publish the baseline

1. Verify the MCP project and dataset against `pages.md` and
   `studio/.env.local`. Create a
   timestamped raw export under `backups/` and run `gzip -t` on it. A failed
   export stops the run.
2. Publish, through MCP, every document the page references that exists only
   as a draft (page-owned FAQs, testimonials, team members, and shared drafts
   from other pages). Then publish the page draft itself.
3. From here on, every write creates or patches `drafts.<pageId>`, guarded by
   `ifRevisionId`. In the first patch, remove `_weak` from the page's
   references to the documents you published, so Studio stops showing
   "Reference strength mismatch".

Ovi compares the fresh draft with this published baseline in Studio, so the
baseline must be the page exactly as it was handed over.

## Look before judging

Start or reuse this worktree's servers with `pnpm dev:worktree` and run
`pnpm page:shots <slug>`. It writes one PNG per block at 1440 and 390 wide
to the folder it prints (under `/tmp`) and prints each block's height. Read every
image. Block heights and the page height are part of the evidence: a
block that is three times taller than its neighbours has to earn it.

## Judge each block

Work through the page in order. For every block, answer the fit questions
below, then give it one verdict. Write the verdict down before changing
anything; the list of verdicts is the plan and it goes in the chat report.

**Fit questions**

1. **Job.** What must the reader do or learn here: choose between options,
   understand a story, follow a sequence, check a price, gain trust, or ask?
2. **Mode.** Does the block's presentation mode serve that job? Use the
   catalogue below. A block's name is a layout, not a topic.
3. **Hierarchy.** Does the fact the reader came for get the biggest slot? A
   price hidden in a checkmark bullet fails. A one-line requirement in a
   full-width story block fails.
4. **Capacity.** Item counts sit inside the schema's minimum and maximum. Each
   slot holds what it was designed for: fact labels on one line, prices as
   short figures, timeline text under 180 characters, key details under
   eight. Read the schema descriptions in `studio/schemas/blocks/` when in
   doubt.
5. **Photos.** Each photo shows what its caption or copy says, and the people
   in it are the page's reader (for example, adults on a page written for
   adults). A photo that contradicts the copy is worse than no photo.
6. **Repetition.** A fact said in two blocks stays where the reader needs
   it and leaves the other. Hero facts, key-detail lists, and FAQ answers are
   the usual offenders.
7. **Order.** The page follows the order its strategy comment explains. A
   requirement comes before the action it gates. Decision blocks sit
   high on a page whose reader came to decide. A move against the
   strategy's order needs a stated reason.
8. **One ask at a time.** The page ends with one closing call-to-action
   block. No two call-to-action blocks (closing bands, nudges, director
   prompts) sit next to each other, and the block before the closing
   band is proof or content, never another ask. Two asks in a row read as
   a false ending and make the primary buttons compete. Verdict for the
   extra band: fold its line into the nearest content block and remove
   it.
8. **Weight.** Block height matches its importance to the page's purpose,
   and the whole page matches its tier. Atmosphere blocks do not outweigh
   the offer.
9. **Rhythm.** Page-level rules: one hero, one FAQ, one team block, the
   page ends with the closing block the Blocks part names, a nudge sits
   mid-page, and three blocks with the same background in a row read as
   one long band.
10. **Placeholders.** Invented quotes and sample data are visibly labelled,
    and the label survives the layout.

**Verdicts**

- **keep**: fits; leave it.
- **trim**: right block, too much or repeated content; cut and dedupe.
- **re-house**: some of the content belongs in another block on the page;
  move it.
- **replace**: another existing block serves this content better; rebuild
  the block with that type and carry every idea across.
- **propose**: no existing block fits without loss; keep the closest
  fallback in the draft and describe the missing block in chat.

Read the Blocks part of `pages.md` first (older sites head it "Sections").
It names the site's own blocks, the blocks not to use, and the page-level
rules, such as the block that closes a page. Where it differs from the
catalogue below, the Blocks part wins. If `pages.md` has no Blocks part,
stop and tell Ovi.

**Block catalogue** (only the blocks every site of this starter family
shares, and the job each serves. The project's `studio/schemas/blocks/`,
its Studio preview images, and the renderer in
`frontend/components/blocks/` are the source of truth for what exists and
how each looks)

| Block | Serves | Watch |
| --- | --- | --- |
| Inner Hero | Opening: heading, one-line body, two buttons, up to four facts | Facts are a short figure plus a one-line label |
| Image and Text (`storyFeature`) | A story or point of view with one photo, a pull quote, key details, buttons | Adjacent ones alternate sides; three paragraphs is the ceiling |
| Image Feature Cards (`featureCards`) | Options the reader picks between: courses, sessions, next pages; two to six linked photo cards per group | Every card needs a photo and a link |
| Feature Grid (`benefitCards`) | Text-only cards with an icon: reasons, features, rules | No links, no photos |
| Stacked Feature Rows | Short headings with supporting points and an optional link | Points are explanations, not prices or comparisons |
| Timeline (`stackedTimeline`) | Ordered steps or a schedule as numbered cards with a time or label, two to eight | Text under 180 characters; photos optional |
| Big Image List | Ordered stops where every stop deserves a large photo, two to eight | Very tall; only when each photo shows the caption |
| Large Slides | Places, moments, or steps with one pinned photo per slide | Photo-led; copy is short |
| Image Collage Feature | A story with supporting points and two overlapping photos | Needs two good photos |
| Heading and Image | A centred heading and paragraph over one full-width photo | A pause, not an argument |
| FAQ hub, and any specialised block the project adds | Their named purpose | Keep to that purpose |
| Quote Wall | Quotes from Testimonial documents as a wall of cards | Long quotes open in a dialog; nine cards show before "Show more" |
| Team Members | Selected Team Member documents, profiles or roster | |
| FAQ Section | One accordion of FAQ documents | One per page |
| Call to Action (`ctaBanner`) | `nudge` between blocks for a small aside; `closing` ends a page unless the Blocks part names another closing block | A nudge never ends the page |
| Rich Text Block, Latest Posts | Starter blocks without a finished design | Avoid; pick a designed block or propose one |

## Apply the verdicts

Build the new `blocks` array in a small node script from the raw draft, so
keys, marks, references, icons, and image hotspots carry over untouched
where content is unchanged. New items get new keys. Check the schema fields
of any block you add (`studio/schemas/blocks/<type>.ts`): required fields,
minimum and maximum counts, the `_type` of nested items and links. Reuse
row icons from existing rows or the lucide render recipe. For photos, query
`sanity.imageAsset` by `description` and choose by what the description
says; write alt text; drop crop and hotspot values that belonged to the old
photo.

Patch `drafts.<pageId>` through MCP with `ifRevisionId`. Then run
`pnpm page:check <pageId>` where the project has it; else read the draft
back to ndjson and run `pnpm --dir studio exec sanity documents validate -y
--file <ndjson> --format ndjson --level warning`. Only
pre-existing markers, such as the legacy `migration` field, may remain.
Run `pnpm page:shots <slug> --out <the printed folder>-after` and read
every image again. Each changed block must look like its verdict said,
at both widths. Fix what does not, and re-shoot that width.

## Report

Add one change note to the handover notes `pages.md` names: the changes
block by block, the ids you published, the backup path, and the new
page height beside the old one. When the polish changed the order of the
page, also post a short "Updated:" comment on the page record, written
for the client like the page strategy comment (see `/page-draft`): what
moved and why. Nothing else in the record changes; the layout proposals
stay in chat.

Then tell Ovi, in chat, the short report by default: at most six lines
with the outcome, what changed for the reader, anything unverified or
open, and the review link for his visual check (the hosted Studio
Presentation link on the card, "Page link on the card" in `pages.md`;
add it to the card if it is missing, never a localhost link), for both
widths, with the blocks to look at first. Say that the look is
unverified until he has seen it. No table.

Give the full report only when Ovi asks for it ("full report", "details",
"more"):

- the verdict for every block and what changed, as a table;
- each **propose** verdict: the content, the fallback used, what the
  fallback loses, and the block that would serve it;
- anything left open;
- the visual-check links again, one for each width.

Ovi decides on each proposal afterwards and asks, when he wants, for a note
in the page record or a GitHub issue.
