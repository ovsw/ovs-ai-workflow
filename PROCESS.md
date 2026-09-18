# AI Website Build Process

This document describes how one human and AI agents build a client website, page by page, until Launch. Terms in capitals are defined in [CONTEXT.md](./CONTEXT.md).

The process has two parts. Part 1 prepares the project once. Part 2 repeats for every page. Publishing content in Sanity is separate from Launch.

## Actors

| Actor | Who | Does |
| --- | --- | --- |
| Human | Ovi | Plans, decides, reviews, publishes after Internal Review, launches |
| Agent | An AI coding agent session | Builds, matches, reports, fixes what it is told to fix |
| Client | One named person at the client | Supplies sources, reviews pages, accepts or rejects them |

Nobody else takes part.

## Where things live

| Thing | Lives in | Notes |
| --- | --- | --- |
| Page Records, Stages, client comments | Miro Page Table on the project Board | Kanban view groups by Stage. Tree view is the Sitemap. |
| Client-facing project, source files | Basecamp | The Human links files into the Page Table. |
| Code tasks and Section Gaps | GitHub Issues in the project repo | Agents use the `gh` CLI. |
| Product Brief, Shared References, design system | Project repo, `PRODUCT.md`, `DESIGN.md`, `docs/` | Agents read these on every build. |
| Content | Sanity dataset | Agents use the Sanity MCP. If it is not authorized, the agent stops and asks the Human. |
| Board id and Page Table id | Project repo agent instructions | Set once during setup. Agents never search Miro by name. |
| Development Build | Vercel, main branch, plus the deployed Sanity Studio | Studio is a separate app. `pnpm deploy:studio` after any merged schema change. |

## Page Table schema

One row per routable page. Navigation, footer, and site settings are Shared References, not pages. A collection such as blog posts gets one row for the listing page and one row for a representative detail page.

| Column | Type | Rule |
| --- | --- | --- |
| Page | Text | Internal name |
| Page brief | Text | Purpose and outcome of the page |
| Original address | Text | Address on the old site, or `new page` |
| Planned address | Text | Address on the new site |
| Parent | Row link | Parent page, or `top level`. Drives the Tree view. |
| Primary avatar | Select | Exactly one |
| Secondary avatars | Multi-select | Zero or more. Write `none` when zero. |
| Source | Links | Page-specific sources, or `none` |
| Prototype | Link | Approved Prototype that covers this page, or `none` |
| Stage | Select | Planning, Ready, Building, Internal Review, Client Review, Client Approved, Client Rejected |
| Blocked | Checkbox | Set by the Agent, cleared by the Human |
| Blocker | Text | Short reason, set with Blocked |

An explicit `none`, `new page`, or `top level` is a decision. An empty cell is not.

## Part 1: Prepare the project

Done once per website. The Human and the Agent work together.

1. **Set up the project.** Start from `ovsw/next-sanity-starter`. Create the repository, the Vercel project on main, and the deployed Studio. Write the project agent instructions with the Board id and Page Table id. Keep the global CodeRabbit and merge rules unchanged.
2. **Write the Product Brief.** Record the audience, purpose, constraints, brand direction, and intended outcomes in `PRODUCT.md`. Prototype work cannot start before this exists.
3. **Explore the design, Human present.** Build the Prototype with representative content. The Human directs and reviews its appearance. Record the outcome in `DESIGN.md`. Most pages will not have a Prototype. That is fine.
4. **Define the Shared References.** Write the Avatars with their jobs to be done, the navigation, the copy style guide, and the design system into `docs/`. Create the Page Table and fill one row per page. The Tree view becomes the Sitemap. These develop together until they are stable enough to build from.
5. **Brief the client.** Give the client editor access to the Board. Tell them two things. First, only pages in the Client Review column are ready for their eyes. Second, the Development Build changes whenever code merges, so a page they are reviewing can shift while they look.

## Part 2: Build one page

Every page goes through all seven Stages in order. There is no shortcut for pages built with the Human present.

### Planning

- The Human writes the Page Record. Every column has a value or an explicit `none`.
- The Client drops sources in Basecamp. The Human links them into the Source column.
- The Human moves the row to Ready when every column is filled and the Shared References are stable. Only the Human moves a row to Ready.

### Ready

- The row waits for an Agent. The Human launches at most one build Agent per row. An Agent only works on the row it was given.

### Building

The Agent runs an AFK Build. The Human is not present.

1. **Readiness Gate.** The Agent reads the Page Record and the Shared References. If any cell is empty or any Shared Reference is missing, the Agent refuses to start, sets Blocked with a Blocker reason, and stops. It never moves a row to Ready.
2. **Resume check.** If the page already has an unpublished Draft, the Agent continues from it. Drafts during Building belong to Agents, so no human diff is lost.
3. **Scaffold the whole page.** The Agent writes every section as a Scaffold with `name`, `richText`, and `proposedSectionShape`. Content comes first. Images come from the Source links or from assets already in the Sanity media library. When neither has a usable image, the Agent places a named placeholder and lists it in its report. When the last Scaffold is written, the Agent publishes the page.
4. **Match each Scaffold.** For each Scaffold the Agent picks a section from the project's Section Library, distributes the content into its fields, places the images, turns links into buttons where that fits, and configures the section. A plain rich-text section is a valid final choice.
5. **Checkpoint each match.** The Agent replaces the Scaffold, moves it into the page's Blocks Archive, and publishes the page. Every replacement is a saved Checkpoint. Archived Scaffolds never render.
6. **Skip Section Gaps.** When no section fits, the Agent leaves that Scaffold in place and continues with the next one. It does not stop at the first gap.
7. **Report.** When the pass is done, the Agent writes a short comment on the row: what it built, placeholders it left, and every Section Gap. Then:
   - No Scaffold remains: the Agent moves the row to Internal Review. Agent publishing for this page ends here.
   - One or more Scaffolds remain: the Agent sets Blocked, writes the gaps in Blocker, and leaves the row in Building.

### Resolving a Section Gap

Human present. Starts from Building or from Client Rejected.

1. The Human decides: use a different existing section, improve an existing section, or build a new one.
2. For code, the Agent opens a GitHub issue, runs `pnpm page-builder:new` for a new section, and builds against the Human's direction.
3. Code goes through `pnpm verify` and one CodeRabbit review on the PR, posted as a trigger comment because auto-review is off.
4. The Agent merges when the global merge conditions pass. Merging does not wait for the Client, and it is not a visual review. After a schema change, the Agent runs `pnpm deploy:studio`.
5. The Agent matches the remaining Scaffolds. In Building it may publish and it moves the row to Internal Review when no Scaffold remains. In Client Rejected it works in Drafts only.
6. The Human clears Blocked.

### Internal Review

The Human is present. Agents no longer publish this page.

1. The Human launches a fresh Agent session. It checks two things only: every link resolves, and spelling is correct. It fixes those as Drafts and reports anything else.
2. The Human inspects the rendered page on the Development Build and the Drafts as inline diffs in Sanity Studio.
3. Changes to meaning, structure, or section choice are decided by the Human and made by the Agent as Drafts.
4. The Human publishes all Drafts and moves the row to Client Review. Both by hand.

### Client Review

- Every Scaffold is resolved and all content is published before the row arrives here.
- The Client looks at the page on the Development Build. They answer on the row: a comment, a move to Client Approved, or a move to Client Rejected. They never edit other columns and they never publish.
- Other pages keep merging to main during this Stage. The Client was told at kickoff.

### Client Rejected

Touch-up, Human present.

1. The row stays in Client Rejected while the work happens.
2. The Agent makes content changes as Drafts. Code follows the normal checks and merge rules. A missing section follows the Section Gap route above.
3. The Human publishes and moves the row back to Client Review. No Internal Review in between.

### Client Approved

- A signal, not a lock. Agents may still change the page as Drafts when the Human asks. Only the Human publishes.
- Code changes can still change how an approved page renders. The Human re-checks approved pages before Launch.

## Launch

The Human owns Launch. It happens once, after every row is Client Approved.

| # | Check | Who |
| --- | --- | --- |
| 1 | Every row is in Client Approved | Agent computes, Human confirms |
| 2 | No Scaffold renders on any page | Agent computes, Human confirms |
| 3 | No unpublished Draft remains | Agent computes, Human confirms |
| 4 | Redirects from every Original address to its Planned address, written in one pass from the Page Table | Agent writes, Human confirms |
| 5 | Domain, analytics, and search indexing set | Human |
| 6 | Final visual pass over the whole site | Human |

Redirects are written only here, never during Building, because Planned addresses can change until Launch.

## Rules that hold everywhere

- **Publishing.** Agents publish on their own only in Building. From Internal Review onward, only the Human publishes.
- **Drafts.** A Draft has one purpose: to let the Human inspect an Agent's change as an inline diff in Studio. Agents never publish over a Draft the Human has not seen.
- **Approval.** Publishing is never how the Client approves. The Client approves by comment or by moving the row.
- **Reviews.** Code reviews follow the global routing table: one CodeRabbit review per PR for schema, data, or architectural changes; none for looks or copy. A code review is never a visual check. Visual checks are the Human's.
- **Merging.** The Agent merges when the global merge conditions pass. Merging never waits for the Client.
- **Parallel work.** Several pages can build at once. One Agent per row. Separate git worktrees for code.
- **Blocked.** A row is in exactly one Stage. Blocked is a flag on the row, never a Stage.
- **Tools that are not authorized.** If Sanity MCP or Miro MCP is not authorized, the Agent stops and asks the Human. It does not work around it.
- **Nothing is frozen.** No Stage stops an Agent from doing work the Human asks for. Stages control who publishes, not who may edit.

## Follow-up deliverables

1. Workflow diagram in Miro, built by Miro AI from [MIRO-PROMPT.md](./MIRO-PROMPT.md), reviewed by the Human.
2. Agent instructions and skills that enforce this process.
3. The Miro Page Table template.
4. Starter changes: [ovsw/next-sanity-starter#50](https://github.com/ovsw/next-sanity-starter/issues/50).
