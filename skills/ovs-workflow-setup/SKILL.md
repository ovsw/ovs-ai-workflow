---
name: ovs-workflow-setup
description: "Set up or check a site for the OVS Website Workflow (the page skills /page-plan, /page-draft, /page-polish on a Basecamp card table). Use when a starter site is new to the page workflow, when docs/agents/pages.md is missing or out of date, or when a page skill stops on a missing setup item. Usage: /ovs-workflow-setup"
---

# OVS Website Workflow setup

Follow `process.md` in this skill folder. It checks every item the page
skills need in this repository, fixes what an agent can fix, and ends with
a gap list: what only Ovi can supply.

The workflow is Basecamp-only. The general procedure and its reasons are on
the ops wiki page `/work/dev/ovs/ops/wiki/topics/website-page-workflow.md`.
Read it before the first run in a session.

The run is rerun-safe: run it on a new site, and again on a site that is
already set up. A second run changes only what is missing or wrong.
