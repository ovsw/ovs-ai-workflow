# Process: implement the sub-issues of one parent issue

Shape of the work: the parent gets one integration branch, BRANCH. Each
sub-issue N is built on its own short branch, SUB = `BRANCH-issue-N`, gets
its own pull request into BRANCH, one CodeRabbit review on that PR, and is
merged into BRANCH before the next sub-issue starts. At the end one draft PR
from BRANCH into DEFAULT waits for Ovi's visual check and merge. That final
PR gets no review: every file in it was reviewed already, and CodeRabbit
skips PRs over 150 files on Ovi's plan (RBP parent #5, PR #21, 192 files,
2026-10-16).

VALUES, filled in once at the start and used everywhere below:

- PARENT: the parent issue number from the argument.
- MODEL: unless Ovi said otherwise, workers, checkers and integration agents
  use `gpt-6-astra` at reasoning effort `medium`; the preview agent uses
  `gpt-6-luna` at `medium`. Say the model and effort in each spawn request.

Rules that hold for the whole run:

- One subagent at a time. Wait for each one before you spawn the next.
- Every subagent works in the current folder. No worktrees.
- A subagent's answer must be JSON in the shape its prompt asks for. If it
  is not, or the agent returned nothing, treat the step as FAILED with the
  reason "output did not match the shape".
- After each subagent returns, close its thread, then write one line in chat:
  the issue, the result, the commit or PR. Ovi follows the run from these
  lines.
- Sub-issue PRs are merged into BRANCH by the integration agent. You and the
  subagents never merge anything into DEFAULT, never mark the final PR ready,
  never close or edit an issue.
- When a step says STOP, stop the whole run and report the reason.

In the `gh api` commands, `{owner}` and `{repo}` are literal text. Type them
exactly as shown, including the braces; `gh` fills them in.

## PART 1: Setup (you do this yourself; read-only until step 1.5)

1.1 Run `gh api "repos/{owner}/{repo}/issues/PARENT" --jq '{title, state}'`.
    Call the title PARENT_TITLE. If it fails, STOP: "cannot read issue
    #PARENT".
1.2 Run `gh api "repos/{owner}/{repo}/issues/PARENT/sub_issues" --paginate`.
    This is SUBS, the full set of sub-issues. If SUBS is empty, run
    `gh api "repos/{owner}/{repo}/issues/PARENT/parent" --jq '.number'`.
    If that prints a number P, STOP: "issue #PARENT has no sub-issues; it is
    itself a sub-issue of #P". Otherwise STOP: "issue #PARENT has no
    sub-issues linked in GitHub". Do not create a branch or a PR for an
    issue without sub-issues.
1.3 Run `git status --porcelain`. If it prints anything, STOP: "uncommitted
    changes in folder". Do not touch those changes.
1.4 Run `gh repo view --json defaultBranchRef`. Call that branch DEFAULT.
1.5 Run `git fetch origin`, then `git branch --show-current`. Call the output
    CURRENT; if empty, STOP: "no branch checked out". Decide BRANCH:
    a) CURRENT ends in `-issue-<number>`: it is a sub-issue branch left by an
       earlier run. BRANCH = CURRENT without that ending. Run
       `git switch BRANCH` (or `git switch -c BRANCH origin/BRANCH` if there
       is no local BRANCH), then `git pull --ff-only origin BRANCH`; if that
       fails, STOP: "local branch has diverged from origin".
    b) CURRENT is not DEFAULT: BRANCH = CURRENT. If it exists on origin, run
       `git pull --ff-only origin BRANCH`; if that fails, STOP: "local branch
       has diverged from origin". If it does not exist on origin, run
       `git push -u origin BRANCH`.
    c) CURRENT is DEFAULT: pick a new branch name for the work on #PARENT by
       the project's naming convention, not yet on origin, not ending in
       `-issue-<number>`. Run `git switch -c BRANCH origin/DEFAULT`, then
       `git push -u origin BRANCH`.
1.6 Run the project's checks as the project instructions define them. If
    they fail, STOP: "checks already fail on BRANCH before any work". Do not
    fix them.
1.7 A sub-issue is DONE if its state is "closed", or if
    `git log origin/DEFAULT..origin/BRANCH -E --grep="#<number>([^0-9]|$)"`
    prints at least one commit (a sub-issue merged into BRANCH leaves a
    squash commit whose subject names it). Every other sub-issue is TODO.
1.8 For each TODO issue, run
    `gh api "repos/{owner}/{repo}/issues/<number>/dependencies/blocked_by" --paginate`.
    If and only if that is empty, search the issue body (any case) for
    "blocked by #N" and "depends on #N" and take those numbers as blockers.
1.9 Classify each blocker: closed, or in SUBS and DONE → satisfied, drop it;
    in SUBS and TODO → keep it; open and not in SUBS → STOP: "issue #X is
    blocked by #Y, which is outside parent #PARENT".
1.10 Build ORDER: repeatedly take the lowest-numbered TODO issue whose kept
    blockers are all already in ORDER. If issues remain and none can be
    taken, STOP: "dependency loop among: <numbers>".
1.11 If ORDER is empty, report "no open sub-issues to implement", with the
    DONE issues, and stop. Otherwise write one line in chat: BRANCH, and
    ORDER as "#a, #b, #c".

## PART 2: One issue at a time: worker, checker, integration

Walk ORDER from first to last. Keep a status for each issue: done, failed,
skipped. For each issue N, with SUB = `BRANCH-issue-N` and TITLE = its
title:

2.1 If any kept blocker of N is failed or skipped, mark N skipped with the
    reason "blocker #B is <status>" and go to the next issue.
2.2 Spawn one worker subagent (MODEL) with the WORKER PROMPT below, BRANCH,
    SUB and N filled in, and NOTES empty. Wait for it. Close its thread.
2.3 Read its JSON. Then verify the claim yourself, read-only: after
    `git fetch origin`, `git log origin/BRANCH..origin/SUB` must print at
    least one commit, and `git status --porcelain` must be empty. If the
    worker says "done" but this check fails, treat it as FAILED with the
    reason "worker reported done but no commit is on origin/SUB".
2.4 If the worker is "done", spawn one checker subagent (MODEL) with the
    CHECKER PROMPT. Wait, close. If `pass` is false, the issue has GAPS.
2.5 Retry rule, one time only: if the worker FAILED for a reason that does
    not begin with PUSH_FAILED, or the checker found GAPS, spawn one more
    worker for N with NOTES = the failure reason or the gaps list. Wait,
    close, verify as in 2.3, and run the checker again if the worker says
    done. After this second attempt there is no third.
2.6 If the last worker is done and the last checker passed: spawn one
    integration subagent (MODEL) with the INTEGRATION PROMPT. Wait, close.
    If its JSON is not usable, spawn it once more (the step is safe to
    repeat: it finds the open or merged PR). Then verify yourself:
    `git fetch origin`, and
    `git log origin/DEFAULT..origin/BRANCH -E --grep="#N([^0-9]|$)"` must
    print the merge commit it names. If the agent says merged but this
    fails, treat it as FAILED: "integration agent reported a merge that is
    not on origin/BRANCH".
2.7 Final status for N: done if the integration agent merged; otherwise
    failed, with the last reason. Keep the PR number, the review kind and
    the findings for the report.
2.8 Stop rules: if a reason begins with PUSH_FAILED or REVIEW_TIMEOUT, end
    PART 2 now (a REVIEW_TIMEOUT PR stays open for a later run). If two
    issues in a row end as failed, end PART 2 now. Skipped issues do not
    reset that count. Issues not reached are "not started".
2.9 Write one line in chat for N: status, PR, review, number of findings,
    merge commit, reason.

### WORKER PROMPT

```
Your task is one GitHub issue in this folder's repository. Its number is on
the last line of this message. Do the work the way this project's
instructions tell you to. This message only covers how your work is handed
over.

Read your issue in full, and read its parent issue #PARENT for the context
and the shared decisions. Do only what your issue asks; the other sub-issues
have their own workers.

NOTES

Your work goes on your own branch, SUB, which branches from the integration
branch BRANCH. Before you start:
1. Run: git fetch origin
2. Run: git ls-remote --heads origin SUB
   If it prints a line, the branch exists from an earlier attempt: run
   git switch SUB (if there is no local SUB, run git switch -c SUB
   origin/SUB), then git pull --ff-only origin SUB. If that pull fails,
   return status "failed" with reason "branch SUB has diverged from origin"
   and change nothing.
   If it prints nothing: run git switch -c SUB origin/BRANCH. If a local SUB
   exists already, run git switch SUB and then git reset --hard
   origin/BRANCH instead. Then run git push -u origin SUB.
3. Run: git status --porcelain
   If the output is not empty, a previous agent left unfinished changes.
   Discard them: git reset --hard HEAD, then git clean -fd.

Your work counts as done only when all of these are true:
- The project's checks pass.
- Your work is committed on SUB, and every commit message you wrote
  contains "#<your issue number>".
- git status --porcelain prints nothing.
- git push origin SUB has succeeded.

If the push fails, try it once more. If it fails again, return status
"failed" with a reason that begins with the word PUSH_FAILED, and do not
undo your commits.

If you cannot finish for any other reason, run git reset --hard HEAD, then
git clean -fd, and return status "failed" with the reason. Keep the reason
concrete: what you tried, what blocked you.

Never do any of these: commit on BRANCH or on the default branch, switch to
any branch other than SUB, create a worktree, open a pull request, merge
anything, close or edit or comment on any issue, force push, amend or
rewrite commits that are already pushed, or skip hooks or checks.

Return JSON only, in this shape:
{ "issue": integer, "status": "done" or "failed",
  "commit": string or null, "summary": one sentence,
  "reason": string or null }
"commit" is the head of SUB after your push.

Your issue number: N
```

NOTES is empty on the first attempt. On the retry it is:
"A previous attempt at this issue did not pass. Its result: <reason or gaps>.
Read the current state of branch SUB before you continue; commits from the
earlier attempt may already be on it."

### CHECKER PROMPT

```
You are the acceptance checker for GitHub issue #N in this folder's
repository. A worker reports that it implemented the issue on branch SUB,
which branches from BRANCH. Decide whether the work meets the issue. You
change nothing: no edits, no commits, no comments, no git commands that
write. The one exception: if git branch --show-current is not SUB, run
git fetch origin and git switch SUB so that the checks run on the worker's
code.

1. Read issue #N in full: its description, acceptance criteria, and the
   comments. Read parent issue #PARENT only for context.
2. Read the worker's work: git fetch origin, then
   git log origin/BRANCH..origin/SUB and git diff origin/BRANCH...origin/SUB
3. Run the project's checks as the project instructions define them.
4. Judge against the issue only. A gap is a requirement of the issue that
   the commits do not meet, a check that fails, or a change outside the
   issue's scope that breaks something. Taste, style, and things the issue
   did not ask for are not gaps.

Return JSON only: { "issue": N, "pass": boolean, "gaps": [string] }
"gaps" is empty when "pass" is true. Each gap is one concrete sentence that
a worker can act on.
```

### INTEGRATION PROMPT

```
You are the integration agent for GitHub issue #N ("TITLE") in this folder's
repository. A worker implemented it on branch SUB, and an acceptance checker
passed it. BRANCH is the integration branch for parent issue #PARENT. Your
job: open the pull request from SUB into BRANCH, get it reviewed, close the
findings, wait for green checks, and merge it into BRANCH. Nothing goes to
DEFAULT. In gh api commands, {owner} and {repo} are literal text; type them
exactly, including the braces.

1. Run: git fetch origin
   Run: gh pr list --head SUB --base BRANCH --state all --json number,url,state
   If an open PR is listed, use it and call its number PR. If a merged PR is
   listed, skip to step 6 with that PR. Otherwise open one, ready for
   review, not a draft:
   gh pr create --head SUB --base BRANCH --title "TITLE (#N)" --body "<one
   short paragraph of what changed>

   Closes #N
   Part of #PARENT"
   Call its number PR.
2. Decide the review. Read the PR diff. If the change is looks, copy or
   content only (the review-routing rules in the user's instructions), the
   review is "none": go to step 4. Otherwise the review is "coderabbit".
3. Look at the PR's comments and reviews for a review by coderabbitai. If
   there is none, post the trigger comment one time:
   gh pr comment PR --body "@coderabbitai review"
   Then wait: check every 60 seconds, up to 30 minutes. A comment from
   coderabbitai that says the review was skipped because of too many files
   is not a review: in that case run the CodeRabbit CLI on SUB checked out,
   coderabbit review --committed --base BRANCH, treat its findings as the
   review, and report the review as "cli". A comment that says the review is
   rate limited means keep waiting. If after 30 minutes there is neither a
   review nor a skip comment, return merged false with a reason that begins
   with REVIEW_TIMEOUT, and leave the PR open.
   Check every finding against the current code. Fix each valid finding on
   SUB: run the project's checks, commit with a message that contains "#N",
   then git push origin SUB. For each finding you do not fix, reply on the
   PR thread with the reason. Never ask for a second review, and never post
   the trigger comment twice.
4. Wait for the PR's checks: gh pr checks PR --watch. If a check fails, read
   its log and fix the cause once on SUB (checks, commit with "#N", push),
   then watch again. If a check still fails, return merged false with the
   reason, and leave the PR open.
5. Merge the PR into BRANCH with a squash commit whose subject names the
   issue: gh pr merge PR --squash --delete-branch --subject "TITLE (#N)"
   Then run git switch BRANCH and git pull --ff-only origin BRANCH; delete
   the local SUB if it still exists. The new head of BRANCH is the merge
   commit.
6. Comment on issue #N one time, if no such comment exists yet:
   "Implemented in PR #<PR>, merged into BRANCH as <merge commit>."
7. Never merge anything into DEFAULT, never close or edit any issue, never
   force push, never mark anything ready on the default branch.

Return JSON only:
{ "issue": N, "pull_request": integer or null, "url": string or null,
  "review": "coderabbit" or "cli" or "none", "review_reason": one sentence,
  "findings": [ { "summary": string, "action": "fixed" or "answered",
                  "commit": string or null } ],
  "merged": boolean, "merge_commit": string or null,
  "reason": string or null }
"reason" is null when merged is true.
```

TITLE goes in with double quotes replaced by single quotes.

## PART 3: Finish (only when every issue in ORDER is done)

Skip this part and PART 4 if any issue is failed, skipped or not started;
go to PART 5. Otherwise spawn one subagent (MODEL) with the FINISH PROMPT.
Wait, close. Check with `gh pr view <number> --json isDraft,url` that the PR
exists and is a draft.

### FINISH PROMPT

```
You are the finish agent of a workflow that implemented the sub-issues of
GitHub issue #PARENT ("PARENT_TITLE") on the integration branch BRANCH, in
the current folder and its repository. Each sub-issue was merged into BRANCH
through its own pull request, each with its own review. In gh api commands,
{owner} and {repo} are literal text; type them exactly, including the
braces.

1. Run: git fetch origin
   Run: gh pr list --head BRANCH --base DEFAULT --state open --json number,url
   If a pull request is listed, return its number and url and do nothing
   else.
2. Otherwise open a DRAFT pull request from BRANCH into DEFAULT, following
   the project's pull request conventions. The title comes from the parent
   issue's title. The body contains:
   - one short line per issue, from this list:
     SUMMARIES
   - one line "Closes #<number>" for every issue in SUBS. SUBS is the full
     set of sub-issues of #PARENT; get it with:
     gh api "repos/{owner}/{repo}/issues/PARENT/sub_issues" --paginate
   - one line "Part of #PARENT". Do not write "Closes #PARENT".
   - one line "Reviewed per sub-issue on the PRs listed above; no review on
     this PR."
3. Do not request any review on this pull request. Do not merge it. Do not
   mark it ready for review. Do not close or edit any issue.
Return JSON only: { "pull_request": integer, "url": string }
```

SUMMARIES is one line per issue in ORDER:
"- #N <title>: <worker summary> (PR #<sub-issue PR>)".

## PART 4: Preview links

Spawn one preview subagent (`gpt-6-luna`, `medium`) with the PREVIEW PROMPT.
Wait, close. Write the preview links in chat as soon as it returns: one
clickable link per line, with viewport and what to look for.

### PREVIEW PROMPT

```
You are the preview agent for the integration branch BRANCH in the current
folder. Your job is to give the user one clickable link for each thing to
look at, for the visual check of these issues:
ISSUE_LIST

1. Make sure the folder is on BRANCH and up to date: git fetch origin,
   git switch BRANCH, git pull --ff-only origin BRANCH.
2. Start the project's dev servers for this folder on the project's
   standard ports, or reuse servers that already run for it. Keep the
   standard ports: third-party keys (maps, auth) are often allowed only on
   them.
3. For every page and every Studio input that these issues changed, make
   one link in the form the user's instructions give for a visual check.
   For a Next.js and Sanity project that is the Studio Presentation link
   open on that page, with its document panel when a document is involved.
   Give each link the viewport to use and what to look for there, in one
   line.
4. Open each page yourself in a headless browser and confirm that the
   changed content is on it. Never open a window or tab on the user's
   screen. If content that the check needs is applied only at merge (for
   example a data migration that waits for the merge), do not apply it to
   published content and do not write drafts on your own: list that page as
   "not checkable before merge", with the reason.
5. Do not commit, push, or change code. Do not stop any server.

Return JSON only:
{ "links": [ { "what": string, "url": string, "viewport": string,
               "look_for": string } ],
  "not_checkable": [ { "what": string, "reason": string } ] }
```

ISSUE_LIST is one line per issue in ORDER: "- #N <title>".

## PART 5: Report

Close every subagent thread that is still open. Then report in this order
and nothing else:

1. One table: Issue, Title, Status (done, failed, skipped, not started),
   Attempts, PR (clickable), Review (coderabbit, cli, none), Findings
   (fixed/answered counts), Reason.
2. BRANCH, and the final draft PR as a clickable link, or the reason the run
   ended early.
3. The preview links, one per line, clickable, with viewport and what to
   look for; then the "not checkable before merge" items.
4. The last line: "Not merged. The final PR waits for your visual check."
