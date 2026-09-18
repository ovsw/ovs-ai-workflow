# Miro AI prompt for the workflow diagrams

Paste everything below the line into Miro AI on the project Board. It asks for two diagrams. Review them against [PROCESS.md](./PROCESS.md).

---

Create two flowcharts on this board, side by side, with a title above each.

Use these rules for both diagrams:

- Rounded rectangles for actions.
- Diamonds for decisions, with "yes" and "no" labels on the outgoing arrows.
- Rectangles with a document icon for artifacts that get written.
- Arrows only in the direction of time. No arrow may point backwards unless the text below names it as a loop.
- Keep every label under ten words.
- Use one colour per actor: Human is blue, Agent is green, Client is orange.

## Diagram 1: Prepare the project

Title: "Part 1: Prepare the project (once per website)"

A single lane, left to right. All boxes are Human colour except where marked Agent.

1. Start node: "New website project".
2. Action: "Create repo from next-sanity-starter".
3. Action: "Create Vercel project on main and deploy Studio".
4. Action: "Write agent instructions with Board id and Page Table id".
5. Artifact: "PRODUCT.md: audience, purpose, constraints, brand, outcomes".
6. Action (Human and Agent together, split colour): "Explore design with representative content".
7. Decision: "Prototype approved?" No loops back to 6. Yes continues.
8. Artifact: "DESIGN.md".
9. Artifact: "Shared References in docs/: avatars, navigation, copy style guide, design system".
10. Action: "Create Page Table, one row per page".
11. Artifact: "Sitemap = Tree view of Page Table".
12. Decision: "Shared References stable?" No loops back to 9. Yes continues.
13. Action: "Give client editor access to the Board".
14. Action: "Tell client: only Client Review column is ready for your eyes".
15. End node: "Go to Part 2 for every page".

## Diagram 2: Build one page

Title: "Part 2: Build one page (repeats for every page)"

A swimlane diagram with three horizontal lanes, top to bottom: "Human", "Agent", "Client". Across the top, seven column headers, left to right: "Planning", "Ready", "Building", "Internal Review", "Client Review", "Client Approved", "Client Rejected". Every box sits in the lane of the actor who does it and under the Stage in which it happens.

### Planning column

- Client lane: "Drop source files in Basecamp".
- Human lane: "Fill every Page Record cell (or explicit none)". Arrow from the Client box into this box.
- Human lane: Decision "All cells filled and Shared References stable?" No loops back to the fill box. Yes goes to the next box.
- Human lane: "Move row to Ready".

### Ready column

- Human lane: "Launch one build Agent for this row". Arrow down to the Agent lane in the Building column.

### Building column (Agent lane unless marked)

1. Decision "Readiness Gate passes?" No goes to "Set Blocked and Blocker, stop" and an arrow up to the Human lane box "Fix Page Record, clear Blocked", which loops back to the gate.
2. Decision "Unpublished Draft exists?" Yes goes to "Resume from Draft". Both paths continue.
3. Action: "Write every section as a Scaffold: name, richText, proposedSectionShape".
4. Action: "Images from Sources or Sanity media library, else named placeholder".
5. Action: "Publish scaffold page".
6. Loop box labelled "For each Scaffold":
   - Decision "A library section fits?"
   - Yes: "Match content into section", then "Replace Scaffold, move it to Blocks Archive", then "Publish Checkpoint".
   - No: "Skip, note Section Gap". Continue the loop.
7. Action: "Write report as comment on the row".
8. Decision "Any Scaffold left?"
   - No: "Move row to Internal Review". Arrow to the Internal Review column. Label the arrow "Agent publishing ends here".
   - Yes: "Set Blocked with gaps, stay in Building". Arrow up to the Human lane box "Resolve Section Gap" (see below).

### Section Gap sub-flow (Human and Agent lanes, Building column, drawn below the main flow)

- Human: Decision "Use other section / improve section / new section?" Three arrows all lead to the Agent box.
- Agent: "Open GitHub issue; pnpm page-builder:new if new section".
- Agent: "Build, pnpm verify, one CodeRabbit review, merge".
- Agent: "pnpm deploy:studio if schema changed".
- Agent: "Match remaining Scaffolds". Arrow back to the "Any Scaffold left?" decision.
- Human: "Clear Blocked".

### Internal Review column

- Human lane: "Launch fresh Agent session".
- Agent lane: "Check links resolve and spelling; fix as Drafts; report the rest".
- Human lane: "Inspect page on Development Build and Drafts in Studio".
- Human lane: Decision "Changes needed?" Yes goes to the Agent box "Make changes as Drafts only" and back to Inspect. No continues.
- Human lane: "Publish all Drafts".
- Human lane: "Move row to Client Review".

Add a note under this column: "From here on, only the Human publishes."

### Client Review column

- Client lane: "Review published page on the Development Build".
- Client lane: Decision "Accept?"
  - Yes: "Move row to Client Approved".
  - No: "Comment on the row and move to Client Rejected".

Add a note under this column: "Every Scaffold resolved and all content published before a row arrives here."

### Client Rejected column

- Human lane: "Read client comments; direct Touch-up".
- Agent lane: "Content changes as Drafts; code through normal checks and merge".
- Agent lane: Decision "New section needed?" Yes goes to the Section Gap sub-flow with a note "Drafts only in this Stage". No continues.
- Human lane: "Publish; move row back to Client Review". Arrow back to the Client Review column, labelled "no Internal Review".

### Client Approved column

- Client lane: End node "Page accepted".
- Human lane note: "Signal, not a lock. Agents may still edit as Drafts when asked."

### Launch (draw as a small box to the right of the swimlanes, Human colour)

Title: "Launch, once all rows are Client Approved". Checklist inside the box:

1. Every row Client Approved (Agent computes)
2. No Scaffold renders (Agent computes)
3. No Draft remains (Agent computes)
4. Redirects Original to Planned, written in one pass (Agent writes)
5. Domain, analytics, indexing (Human)
6. Final visual pass (Human)

Arrow from the Client Approved column into this box.
