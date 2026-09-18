# AI Website Workflow

The process by which a human and AI agents prepare, build, review, and publish a Next.js and Sanity website page by page.

## Language

### Planning

**Product Brief**:
The PRODUCT.md document that records audience, purpose, constraints, brand direction, and intended outcomes. Prototype work cannot start without it.
_Avoid_: Project brief, discovery doc

**Prototype**:
The exploratory design, built with representative content, that the human approves. It becomes a reference for the pages it covers. Most pages have none.
_Avoid_: Mockup, comp, design

**Avatar**:
A named audience member with jobs to be done. Each page has one primary avatar and any number of secondary avatars.
_Avoid_: Persona, user type

**Shared References**:
Project-level documents every page draws on: avatars, sitemap, navigation, copy style guide, and design system.
_Avoid_: Project docs, global docs

**Page Record**:
One row in the Page Table. It owns the page brief, primary and secondary avatars, original and planned addresses, parent-page relationship, page-specific sources, Prototype reference, Stage, and client comments.
_Avoid_: Page card, page ticket, page doc

**Page Table**:
The Miro table on the Board with one Page Record per row and a fixed schema of columns. Its Kanban view and Tree view show the same rows.
_Avoid_: Card table, tracker, spreadsheet

**Stage**:
The column value of a Page Record that says where the page is. Exactly one of: Planning, Ready, Building, Internal Review, Client Review, Client Approved, Client Rejected. The Kanban view groups by it.
_Avoid_: Status, phase, state

**Board**:
The Miro board for one website project. It holds the Page Table and all client collaboration. Basecamp tracks the project for the client, GitHub Issues track code tasks, and neither holds the build.

**Sitemap**:
The Tree view of the Page Table, drawn from each Page Record's parent relationship. It is a view, never a separate source.

**Readiness Gate**:
The check an agent runs before an AFK Build. It refuses to start when any Page Record field is empty or any Shared Reference is missing. An explicit "none" passes; an empty field does not. Only the human moves a Page Record to Ready.

### Building

**AFK Build**:
A build session that an agent runs without the human present. It starts only when the Page Record and Shared References pass a readiness check.
_Avoid_: Autonomous build, unattended build

**Scaffold**:
A placeholder section in Sanity holding a name, the complete intended rich text (including images and links), and a proposed section shape. Content comes first. Images come from the page sources or from assets already in the Sanity media library; a named placeholder stands in when neither has one.
_Avoid_: Draft section, placeholder, stub

**Section Library**:
The set of section types in one website's codebase that a Scaffold can be matched to. The starter ships only the Scaffold section; every other section belongs to its own project. There is no central library.
_Avoid_: Block library, components, shared library

**Matching**:
Selecting a Section Library section for a Scaffold and distributing its content into that section's fields.
_Avoid_: Conversion, upgrade

**Checkpoint**:
A published page state after one Scaffold has been replaced by a matched section and archived.

**Blocks Archive**:
The per-page store of replaced Scaffolds. Archived Scaffolds never render on the frontend.
_Avoid_: Trash, history

**Section Gap**:
A Scaffold for which no Section Library section fits. The agent stops on it and brings it to the human.

### Reviewing

**Internal Review**:
The Stage every page enters when its last Scaffold is resolved. The human launches a fresh agent session that checks only that links resolve and spelling is correct, then inspects the rendered page and the Drafts in Sanity Studio. No page skips it.
_Avoid_: QA, self-review

**Client Approved**:
The Stage a client moves a page to when they accept it. It is a signal, never a lock: agents may still edit the page as Drafts when the human asks.

**Client Review**:
The Stage where the client inspects the fully published page on the Development Build and answers on its Page Record: a comment or a move to Client Approved or Client Rejected. Every Scaffold must be resolved and all content published before it starts.
_Avoid_: Client approval, sign-off

**Development Build**:
The always-on deployment of the main branch, plus the latest deployed Sanity Studio, that the client sees during the whole build. There are no preview branches before Launch.
_Avoid_: Staging, preview

**Touch-up**:
A revision made with the human present while a Page Record sits in Client Rejected. The page returns straight to Client Review afterwards, with no Internal Review.

**Draft**:
An unpublished Sanity change. Its only purpose is to let the human inspect an agent's change as an inline diff in Sanity Studio. From the start of Internal Review, agents change drafts only and never publish.

**Publish**:
Making Sanity content live. Agents publish on their own during Building; from Internal Review onward only the human publishes. Publishing is never how a client approves anything.
_Avoid_: Launch, release, approve

**Launch**:
Making the whole website public. Separate from publishing any page's content.
_Avoid_: Go-live, publish
