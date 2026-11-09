# Page content work: <Site name> facts

The page skills (`/page-plan`, `/page-draft`, `/page-polish`) hold the
process. This file holds this project's facts. Use only these; never copy
ids, names, readers or rules from another project. `/ovs-workflow-setup`
wrote this file on <YYYY-MM-DD> and checks it on every run.

## Release scope (<source>, <YYYY-MM-DD>)

<What this release is, its launch date, and which pages are in it. Name
what is out of scope (features, lead magnets, later pages). Say what a page
does when a fact is unconfirmed, and what a button does when its target
page is not in this release.>

## Stage use (<source>, <YYYY-MM-DD>)

<Which stages the pages go through, named as in "Stages" in the page
record below. For example: "Rebuild of the live site. Existing pages
start in To Build; a new page starts in To Plan.">

<!-- ovs-workflow-setup:page-record:start -->
## Page record

The template in `/ovs-workflow-setup` owns this part. A setup run replaces
it with the template's current version; edit the template, not this part.

- **System:** Basecamp project <project name> (`<projectId>`), card table
  "<card table name>" (`<cardTableId>`):
  <card table URL>
  One card per page. The card title is the page path.
- **Stages** are the board's columns; a card's column is its page's stage.
  All of them, each with its name as the board shows it (emoji included)
  and its `id`: <Backlog name> (`<id>`), <To Plan name> (`<id>`), <To
  Build name> (`<id>`), <Internal Review name> (`<id>`), <Client Review
  name> (`<id>`), <Accepted name> (`<id>`), <Rejected name> (`<id>`).
  Move cards by `id`. The rest of this file and the page skills name a
  stage without its emoji.
- **Logins:** page-card work uses Clark, the agents' Basecamp user (person
  `<clarkPersonId>`): add `--profile claude` to every `basecamp` command for
  this work. Ovi is person `<oviPersonId>`.
- **Clients see the card table.** The card holds
  only a short description (the plan link, what the page is, what to
  check, the page link), subtasks for the facts the client confirms, the
  page strategy comment, and the client's comments. Plan text and agent
  notes stay in the plan document.
- **Taking a page:** read the card's assignees. If Clark is assigned,
  another run has the page: stop and tell Ovi. Otherwise assign Clark,
  read the card again, and start only if Clark is now assigned.
- **Assignment shows whose turn it is:** Clark while a run works; at
  handover, remove Clark and assign Ovi; at Client Review, the client's
  reviewer.
- **Stage moves:**
  - `/page-plan`: the card moves to To Build once its plan document exists.
  - `/page-draft`: the card stays in To Build while the draft is written;
    at handover it moves to Internal Review, assigned to Ovi.
  - `/page-polish`: the card keeps its stage; at the end, assign Ovi.
  - Only Ovi moves a card to Client Review. With the move: publish the
    page and every document it references, write the card's short
    description and page link below the plan link, add the facts to
    confirm as subtasks, and assign the client's reviewer.
  - The client moves an approved card to Accepted. An approval by comment
    or email counts too.
- **Subtasks:** a subtask assigned to Clark is work for a run; a subtask
  assigned to the client is for the client.
- **Comments on a card:** Clark posts the page strategy comment at
  `/page-draft` handover, a short "Updated:" note when `/page-polish`
  changes the page's order, and short status notes ("Fixed: …") after Ovi
  asks. Every Clark comment is written for the client: no questions or
  promises to the client.
- **Plan documents:** one Basecamp document per page in the team-only
  folder "Page plans" (`<plansFolderId>`), titled with the page path. The
  page facts come first (reader, the page's job, status new / rewrite /
  keep / merge, old pages, launch), then the plan.
- **Plan link on the card:** the first line of every card's description
  is `<div><strong>Plan (team only):</strong> <a href="<plan document
  URL>">Open the plan document</a></div>`. Each card links to its own
  plan; pages never share one. Clients cannot open the link; Ovi tells them
  that is normal. Keep this line when you edit a description; `basecamp
  cards update --body` replaces the whole description, so read it first.
- **Handover notes:** comments on the page's plan document, headed "Draft
  notes" (`/page-draft`) or "Polish notes" (`/page-polish`).
- **Page link for clients:** the hosted Studio's Presentation view of the
  published page, never the website directly:
  `<hostedStudioUrl>/presentation?preview=<URL-encoded page path>&perspective=published`
  Clients need a Sanity login to open it; Ovi sends the invitations.
  For a draft review link, leave out `perspective`: Studio shows drafts by
  default. Never write `perspective=drafts`; Studio reads it as a release
  name and the document panel shows an error.
<!-- ovs-workflow-setup:page-record:end -->

## Plan

- **Topic map:** `docs/content/topic-map.md` (`/page-plan` writes it
  before the first plan): which page owns each topic, and where each old
  page's content goes.

## Readers and sources

- **Readers:** <where the reader profiles live (segments, avatars), and
  which part an agent reads for a page.>
- **Marketing plan:** <path or link, or "none">.
- **Old site content:** <how to read an old page, for example
  `pnpm legacy:page <path>`, a folder, or the live site>.
- **Facts about the client:** <PRODUCT.md or another source>.

## Message rules

- <Rules for what pages say and never say.>
- **Confirmation marker:** "<(Client to confirm)>". The client confirms
  these facts during Client Review.

## Shared wording (<source>, <YYYY-MM-DD>)

Every plan and every draft uses these exact words.

- <Who the client serves, the short description, names.>
- **Buttons (label → target):**
  - "<label>" → <path>.

## Client review

- **Client's reviewer(s) in Basecamp:** <name> (person `<id>`). In a
  public repository, write a name only if the public site shows it;
  otherwise write the person id alone.

## Sanity

- Project `<projectId>`, dataset `<dataset>`; commands in
  `docs/agents/sanity-cli.md`.
- Hosted Studio: <hostedStudioUrl>. Local Studio for agents: the port in
  `.worktree-ports.json` after `pnpm dev:worktree`.

## Sections (<source>, <YYYY-MM-DD>)

Read this part before you choose sections for a page. Where it differs
from the section catalogue in the shared page skills, this part wins.

- **Use freely:** <block types>.
- **Use only for their named job:** <block type → page or job>.
- **Closing section:** <the block type, and variant, that ends a page>.
- **Do not use:** <block types, with the reason>.
- **Missing photos:** <what a draft does when no fitting photo exists>.
