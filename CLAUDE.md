# CLAUDE.md

## Working in this repo

- when writing or editing code, or running build / test / git commands, follow
  `.claude/skills/coding/SKILL.md` — prompt-free command shapes (write a script rather than inline a
  heredoc; one action per Bash call; Read/Grep/Edit over shell pipelines), and staying autonomous
  instead of stopping for input.
- always invoke scripts in the repository by relative path from the repo root (e.g.
  `devops/scripts/prod-spot-check.py`, `.claude/skills/prod-release/scripts/wait-rollout.sh`),
  never by absolute path.
- **branch only for long-running work.** the deciding question is how long the work will stay in
  progress. a couple of days: commit straight to `main` — a feature branch is overkill while the
  team is so small, and this overrides the default habit of branching before committing when the checkout is
  on the default branch. weeks: use a feature branch, so the work can be checked in and set aside
  while something else goes on `main`. `main`'s history is linear and intentionally so, so a
  feature branch lands by rebasing onto `main` and fast-forwarding, not by a merge commit. when it
  is unclear which case a piece of work is, ask rather than guess. never open a pull request, or
  suggest opening one, unless I ask for it in so many words.

## Hard constraints

- **one replica until EventHub is distributed.** the app runs `replicas: 1` and must keep doing so
  until `EventHub` is backed by Kafka or an equivalent cross-process bus. it is an in-process ZIO
  Hub today, so a second replica delivers events only to its own subscribers — sockets stay
  healthy, nothing errors, nothing reconnects, and live updates just silently stop crossing
  replicas. never propose or apply a change that scales the deployment past one replica, and treat
  any design that assumes multiple replicas as blocked on the EventHub work.

## Build

- backend (Scala): `sbt "~ reStart"` (dev mode), `sbt compile` (compile check)
- frontend: `cd web && ./start-dev.sh` (dev mode), `cd web && npm run compile` (compile check)
- run `npm run compile` in web/ when editing frontend code to verify it compiles

## Code Style

- **Scala**: 2-space indent, explicit types on public methods, ZIO for effects/errors
- **TypeScript**: 2-space indent, strong typing, immutable data patterns
- **React**: functional components, explicit state management
- **naming**: PascalCase (classes/components), camelCase (variables/methods)
- **imports**: group by external/internal, organized logically. no wildcard imports in Scala (unless
  contained in a small enclosed block). no unused imports
- **architecture**: domain-driven, MVC (frontend), ZIO layers (backend)
- **API error handling** (thought-required, not rote): always log any *unexpected* error coming back from the
  service layer to an API handler before it becomes a response — never swallow it. catching *specific expected*
  errors to build a 4xx is fine (`mapError` / `flatMapError` / pattern-match); the catch-all path must log the
  rest — e.g. `case _ => logCause(t) *> …`, or wrap the service call in `logAndRespond` (which `taskHandler`
  does). `UserApiImpl` is the model. this is not the rote rule "never `mapError` in a `logResponseIO`"
- always import Scala classes at top of file. no naming classes with package prefixes mid code
- a class or trait and its companion object share a file (object last). everything else gets its own
  file. sealed trait variants stay with their parent trait
- trait/impl pairs (e.g. `MatchApi`/`MatchApiImpl`) must be in separate files
- do not write whitespace on otherwise empty lines
- never leave trailing whitespace on a line
- always add a newline at the end of a file
- comments and commit messages do not start a sentence with a capital letter. capitals are reserved for
  proper names and code identifiers
- when about to add a comment, consider an extract-method refactor instead. the method name can often
  replace the comment, and the code is usually cleaner for it
- **markdown**: line limit is 120 characters. always a blank line between a paragraph and a following
  bullet list
- write Scala try/catch statements without {} parens
- when Scala method arguments are listed one per line, include a trailing comma after the last
  argument before the closing paren
- when moving a class to a new file as part of a refactoring, remove the old version immediately —
  leaving it around creates compiler errors
- aggressively remove dead code. don't keep deprecated files or unused classes around "temporarily"
  — delete them as soon as they're replaced

## Testing

- backend: ZIO Test framework. frontend: Cypress (e2e)
- the Scala test suite has two independent parts: `sbt test` (unit) and `sbt sequentialSuite`
  (sequential/repo). both are required to pass
- run specific test: `sbt "testOnly package.path.TestName"`
- on any repo test failure, reset the db before running tests again:
  `src/main/resources/sql/recreate_schema.sh -u variationalgo -d variationalgo_test`
- four test levels: repo, service (for logic inappropriate to test through the API), api
  (service+api class pairs together), and functional (Cypress). service classes without a matching
  api class are tested individually
- every passing repo test must clean up after itself
- every passing Cypress test must clean up after itself be cancelling open invites and resigning
  from incomplete games.
- in Scala tests, include a trailing comma after the last test in a suite and blank lines between
  tests. in Cypress tests, use a semicolon instead of a comma. this makes it easy to reorder,
  comment out, and cut/paste
- RepoStubs are primed by `retrieveThese` and act as mini in-memory database tables — only call
  `retrieveThese` once per stub per test
- Cypress: `cd web && npm run cypress_run` (headless) or `cd web && npm run cypress_open`
  (interactive). the app is usually already running in dev mode

## Specs

- `doc/` is the working desk: specs being implemented right now.
- `doc/open/` holds everything else that is still live — paused work, planned features, and
  long-lived active documents such as epic scopes and epic plans, which stay relevant across many
  implementation cycles without being the current task. Living in `doc/open/` says nothing about
  whether a document is settled or in force; an epic scope there is signed and binding.
- When a spec is completed, it moves to `doc/closed/` with a date-stamp filename prefix.
- Use regular English capitalization and punctuation in both functional and technical specs. this is
  a new rule so you might find older specs that break this convention.
- When implementing a spec, stay on spec. if anything is unclear or seems wrong, ask before
  improvising.

## Domain

- Any counting dispute can be resolved by returning to the game and filling in more stones.
  Eventually, the players will reach a fully provable result.
- A player cannot lose unless they have played a move — it's a forfeit instead.
- There is no immediate ko in Badux. There are only impractical super-kos.
- There are no ladders in Badux.
- Radius-3 boards are too small to be representative: a game on a board that small does not behave
  like a real game, so draw no conclusions from r3 games. Do not run games on boards smaller than r5
  anywhere — r5 is the smallest real board.
