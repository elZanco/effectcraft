# EffectCraft — contribution workflow (elZanco)

> **Never commit this file.** `WORKFLOW.md` is personal working notes for
> elZanco's agents, not project documentation. It is listed in
> `.git/info/exclude` (and matched by `.gitignore`) precisely so it can never
> be staged by accident and **must never end up in any PR or in upstream**.
> The only place it may ever be committed is the private sync branch
> `local/workflow` on the fork (via `git add -f WORKFLOW.md`), purely so other
> machines can `git fetch fork && git show fork/local/workflow:WORKFLOW.md`.
> On every other branch it stays an untracked file.

Personal working agreement for this fork, written 2026-10-19. Where this file
disagrees with AGENTS.md / CONTRIBUTING.md of upstream, those win for anything
that lands in a PR; this file only governs *how* we work here and how much we
spend per change.

## Why this workflow (short)

Upstream merges ~10 external PRs per day, but review happens opportunistically
(not first-in-first-out) and the repo has **no pull-request CI**. Full-day
`cargo xtask ci` runs per PR protected nothing that the short gates plus the
community reviews did not catch. Meanwhile `main` moves fast: any PR older than
2–3 days risks merge conflicts. So: keep every PR cheap to produce and cheap to
keep alive, and keep a personal integration branch so the improvements are
visible in the app immediately, regardless of when upstream merges.

## Per-PR quality gates (~15 minutes, NOT the full xtask ci)

1. `cargo fmt` / `cargo fmt --check`
2. Tests of only the crates the change touches, e.g.
   `cargo test -p effectcraft-engine --release --lib tests_shortcut_cmds`
   (widen to the whole crate when the change is cross-cutting)
3. `cargo clippy -p <crate> --release --lib -- -D warnings`

Known pre-existing environment failures on machine A (the
`effectcraft-export` ffprobe 8.0.1 oracle, the `effectcraft-gpu` posterize
test) are not ours and never block a PR. Reserve the full `cargo xtask ci`
for release-sized changes, if at all.

## PR policy

- One task per commit; English commit messages; see CONTRIBUTING.md.
- **Issue-first**: search open issues before starting non-trivial work; file a
  good issue when the change is behavioural. Upstream fixes issues fast (our
  #1685 was fixed by someone else in 1.5 days, PR #1785).
- Keep PRs small and single-concern (roughly < 300 changed lines). No big
  UI-behaviour reworks without a maintainer showing interest first.
- Branch off a fresh `origin/main`. When a PR conflicts: merge `origin/main`
  into the branch **the same day** (merge commit, never force-push), rerun the
  short gates, push.
- After ~5 days without any reaction: **one** friendly "no pressure, just
  checking" comment. Never more than one.
- PR body: what, why, AE-parity framing, how it was tested. Say explicitly
  that something is "After Effects' original behaviour" when that is the
  justification.

## Local bundle branch (`ae-parity`) — see everything now

- `fork/ae-parity` = fresh `origin/main` + **all** open PR branches merged.
- Build and run the app from it so the AE-parity work is visible immediately,
  even while the PRs wait for review.
- After opening a new PR branch: merge it into `ae-parity`. After upstream
  moves: `git merge origin/main` into it, resolve the (usually small)
  conflicts, rebuild.
- Never open a PR from `ae-parity`; it is the personal nightly.

## Environment notes (machine A: Windows + WSL)

- Cargo and gh only work inside WSL:
  `wsl -e bash -lc 'cd /mnt/c/Users/Egon/effectcraft && …'`, with
  `PATH=$HOME/.cargo/bin:$HOME/.local/bin:$PATH` and
  `CARGO_TARGET_DIR=/home/egon/.cache/effectcraft-target`.
- Windows `git push` hangs on the credential manager: always push via WSL git
  (`git push fork <branch>`).
- Avoid quotes/brackets inside `wsl -e bash -lc '…'` (PowerShell and wsl.exe
  eat them). For anything complex, write a `.sh` under
  `C:\Users\Egon\AppData\Local\Temp\opencode\` and run
  `wsl -e bash /mnt/c/Users/Egon/AppData/Local/Temp/opencode/<file>.sh`.
- Long builds/tests: launch the script detached
  (`Start-Process wsl.exe -ArgumentList '-e','bash','/mnt/c/…/script.sh' -WindowStyle Hidden`),
  then poll its log for the `*_EXIT=` marker.
- The app is a Linux ELF (`/home/egon/.cache/effectcraft-target/debug/effectcraft`)
  shown via WSLg. Run it in a *held* WSL session
  (`exec …/effectcraft >> /mnt/c/…/app.log 2>&1` inside a detached wsl.exe):
  WSL tears down background processes when the launching wsl.exe exits, and a
  missing `.exe` suffix is normal — it is not a Windows binary.
- A hung WSL: `taskkill /F /IM wsl.exe`, then start fresh.
- **Line endings:** this repo is checked out with CRLF on Windows. The repo's
  `.git/config` now has `core.autocrlf=true` so the WSL clone sees a clean
  tree too; without it *every* file looks modified and `git checkout` of any
  other branch aborts. After a fresh clone on a mixed Windows/WSL setup run
  `git config core.autocrlf true` once.
- **Long jobs: run them foreground, not detached.** A `Start-Process wsl.exe`
  child dies with its PowerShell parent when a tool call times out, and that
  tears the whole WSL VM down mid-build (the VM reboots, cargo is gone, the
  log just stops). Instead run `wsl -e bash script.sh` as the single blocking
  call with a generous timeout; the WSL VM survives and the job finishes.
- **`WORKFLOW.md` is deleted on branch switches** while it is tracked on
  `local/workflow`; restore it with
  `git show local/workflow:WORKFLOW.md > WORKFLOW.md` (it stays untracked).

## Open PRs (status 2026-10-19)

| PR | Branch | Topic | State |
|----|--------|-------|-------|
| #1653 | `menu-bar-native-behaviour` | native press-and-browse menus, edge auto-scroll | large; waits for maintainer signal |
| #1654 | `tools-double-click-create` | double-click a Tools-bar creation tool (#1072) | clean |
| #1655 | `dev-line-tables-only` | line-tables-only debug info in dev profile | clean |
| #1732 | `i18n/german-effect-names` | German effect catalog uses AE's own names (#821) | clean |
| #1737 | `fix/trim-out-keeps-playhead-frame` | Alt+] keeps the playhead's frame (original AE behaviour) | clean, approved by nanox333 |
| #1761 | `ui/shape-group-elided-contents` | shape group's inner Contents row elided, as in AE | small |

## Outcomes we accept

- Our PRs may sit for days; that is upstream's call. The bundle branch keeps
  us unblocked and the work visible.
- Large reworks (e.g. #1653) get no further investment until a maintainer
  signals interest; the local branch is the better host meanwhile.
- Contributor credit stays `@elZanco` (user chose the default; no
  `contributors/people.toml` entry).
