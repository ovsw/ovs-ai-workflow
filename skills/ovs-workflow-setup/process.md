# OVS Website Workflow setup process

Shared process for `/ovs-workflow-setup` in Claude Code and
`$ovs-workflow-setup` in Codex. Run it from the root of a site repository
built from the OVS Next.js + Sanity starter.

## Outcome

Every check below ends in one state:

- **pass**: in place and correct;
- **fixed**: this run set it up or corrected it;
- **gap**: only Ovi can supply it (a login, an invitation, a decision, a
  fact). Each gap names the exact thing Ovi must do.

The run is done when every check has a state, the repository changes are
on one branch, and Ovi has the report. Keep a running checklist with the
check numbers below and fill in each state as you go.

Ovi makes the decisions; finding facts is the run's job. Look up ids,
URLs, schema types, scripts and file paths yourself. Ask Ovi only for
decisions and for facts no system holds, and ask them together in one
message.

## Ground rules

- Read the repository's `AGENTS.md` first. Its rules for branches,
  commits, PRs, reviews, datasets and public content apply to every write.
- Use Ovi's default Basecamp login for setup writes (missing columns,
  folders, cards). Clark (`--profile claude`) is for page work, not setup.
- Change only what is missing or wrong. Keep everything Ovi wrote in
  `pages.md`, keep extra card-table columns, and keep cards where they are.
- Secrets stay in ignored env files. Check that a token is present; never
  print it.

## 1. Project and logins

1.1 **Basecamp login.** `basecamp auth status` shows the `ovi` profile as
    the default. `basecamp --profile claude me` answers as Clark. Record
    both person ids.
1.2 **Basecamp project.** Take the project id from `pages.md` if it exists;
    otherwise ask Ovi for the project. Clark must be a member
    (`basecamp people list --project <id>`); if not, it is a gap: adding a
    person sends an email, so Ovi does it.
1.3 **Sanity target.** `studio/.env.local` holds `SANITY_STUDIO_PROJECT_ID`,
    `SANITY_STUDIO_DATASET` and `SANITY_AUTH_TOKEN`, and the project and
    dataset match `pages.md` and `docs/agents/sanity-cli.md`. A read with
    the token works (`pnpm --dir studio exec sanity documents query
    'count(*[_type=="page"])'`).
1.4 **Sanity MCP uses the API token, not OAuth.** The `sanity` server in
    `.mcp.json` has `"headersHelper": "node scripts/sanity-mcp-headers.mjs"`
    and the script exists. If not, copy `templates/sanity-mcp-headers.mjs`
    from this skill folder to `scripts/` and add the line.
    `node scripts/sanity-mcp-headers.mjs > /dev/null` exits 0. The new
    connection loads in the next session; say so in the report.
1.5 **Screenshot browser.** `pnpm page:shots` uses Playwright Chromium from
    the frontend's `@playwright/test`. If `pnpm --dir frontend exec
    playwright install --dry-run chromium` shows it missing, install it
    with `pnpm --dir frontend exec playwright install chromium`.

## 2. Basecamp board

2.1 **Card table.** Find the project's page card table, "Website Pages
    Workflow". If there are several, ask Ovi which one. If there is none,
    it is a gap: Ovi creates it from his Basecamp project template, so do
    not create a plain one.
2.2 **Columns.** Column names may start with an emoji ("📘 To Plan");
    compare names with the emoji and spaces removed. Read the columns
    from `basecamp api get /buckets/<p>/card_tables/<id>.json` (`lists`):
    - Backlog, Rejected and Accepted are Basecamp's fixed columns. Find
      them by `type`: `Kanban::Triage` (Backlog), `Kanban::NotNowColumn`
      (Rejected), `Kanban::DoneColumn` (Accepted). Their names do not
      matter, and they cannot be added, deleted or moved.
    - To Plan, To Build, Internal Review and Client Review are
      `Kanban::Column`. Create only a column that is missing, with its
      plain name. Do not rename or move existing columns. Report other
      columns; keep them.
    Record each column's id and its name exactly as the board shows it
    (emoji included) for `pages.md`; the page skills use the ids.
2.3 **Clients see the board.** The card table's `visible_to_clients` is
    true (`basecamp api get /buckets/<p>/card_tables/<id>.json`). Fix it
    with `basecamp recordings visibility <id> --visible`. If the project
    has client access turned off, that is a gap for Ovi, not a fix.
2.4 **Team-only plan folder.** A folder named "Page plans" sits at the root
    of Docs & Files, and its `visible_to_clients` is false. Create it at
    the root without `--visible-to-clients`, then read it back to confirm.
    Documents in it inherit its visibility.
2.5 **One card per page.** The page list is the release scope in
    `pages.md`; when that gives no list, use the site's page inventory or
    sitemap, and confirm the list with Ovi. A column's card list comes 50
    cards at a time: read `cards.json?page=2` and on until a page is
    empty, or you will create duplicates. Create a card for each page
    path that has none; new cards go to Backlog with the path as the title.
    Leave existing cards, their columns and descriptions alone, with one
    exception: when a card's page has a plan document and the card has no
    plan link, add it as "Plan link on the card" in `pages.md` says. Report
    cards whose path is not in the list.

## 3. Project files

3.1 **`docs/agents/pages.md`.** If it is missing, build it from
    `templates/pages.md` in this skill folder:
    - Fill every id, URL and path you can find: the Basecamp ids from
      part 2, Clark's and Ovi's person ids, the Sanity project, the hosted
      Studio URL (3.4), the source paths.
    - Draft the Blocks part from `studio/schemas/blocks/`,
      `docs/agents/page-builder.md`, and the renderer in
      `frontend/components/blocks/`. Propose the closing block.
    - Then ask Ovi, in one message, for what only Ovi can decide: the
      release scope, which stages the pages go through, the readers source,
      the message rules and confirmation marker, the shared wording and
      button targets, the client's reviewer(s), and approval of your
      Blocks draft. Give your recommended answer for each.
    - Also ask about any rule in `AGENTS.md`, the spec or `GLOSSARY.md`
      that stops the page skills: a content freeze, a planned re-import,
      or "no copy changes". Remove a rule only where Ovi says so.
    If it exists, check that every part of the template is present and
    that its ids match part 2. Replace the part between the
    `ovs-workflow-setup:page-record` markers with the template's version,
    filled with this project's ids. A file without the markers gets the
    marked part in place of its "Page record" part, after Ovi sees the
    difference. Add missing parts with the same interview.
3.2 **Agent instructions.** `AGENTS.md` has a "Page content work" block
    that names the three page skills, this setup skill, and
    `docs/agents/pages.md`. Add it if missing. If the repository keeps
    `CLAUDE.md` identical to `AGENTS.md`, change both.
3.3 **Supporting files.** These exist: `docs/agents/page-builder.md`,
    `docs/agents/sanity-cli.md`, `GLOSSARY.md`, and `DESIGN.md` (root or
    `frontend/`). A missing one is a gap. The topic map
    (`docs/content/topic-map.md`) is needed only when pages go through To
    Plan; `/page-plan` writes it, so report its state only.
3.4 **Hosted Studio and Presentation.** Find the hosted Studio URL (the
    `studioHost` in `studio/sanity.cli.ts`, `docs/deployment.md`, or the
    Sanity project). It answers over HTTPS. Studio has `presentationTool`
    in `studio/sanity.config.ts`. Build one Presentation link for a
    published page with the pattern in `pages.md`; Ovi opening it once is
    a gap until Ovi confirms it shows that page. If the project's
    `pages.md` has a link pattern with `perspective=drafts`, that is a gap:
    Studio reads `perspective` as a release name, and "drafts" breaks the
    document panel. Draft links leave `perspective` out.

## 4. Page scripts

4.1 These root `package.json` scripts exist: `dev:worktree`, `dev:stop`,
    `page:text`, `page:shots`. `/page-draft` also uses an old-page lookup
    (for example `legacy:page`) when the site has old content; it is
    optional.
4.2 A missing script is code work: open one GitHub issue for it, following
    the repository's issue rules. Search the open issues first and link an
    existing one instead of opening a second.

## 5. Repository changes

Put every file change of this run (`pages.md`, `AGENTS.md`, `CLAUDE.md`,
`.mcp.json`, the helper script) on one branch and commit it, following the
repository's rules. Open the PR when the repository's rules allow an
agent to; otherwise give Ovi the branch name.

## 6. Report

Tell Ovi, in at most six short lines:

- whether the site is ready for the page skills;
- what this run fixed;
- the gap list, each with the exact thing Ovi must do;
- the links: the card table, the PR or branch, the Presentation link to
  check.

Give the full checklist with every state only when Ovi asks for details.
