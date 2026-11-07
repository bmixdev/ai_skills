# User-global rules

These rules apply to every session and every project on this machine. They
override a project's convenience, not its explicit content.

## Never submit anything to an upstream repository

Do not open pull requests, push branches, file issues, or post comments to any
repository that this machine does not own. This applies to the DeepSeek Harness
upstream (`deepseek-ai/deepseek-harness`) and to every other third-party or
open-source project.

Work belongs in the user's own fork:

- Fork: `bmixdev/deepseek-harness`
- Remote `origin` in a harness worktree points at the fork.
- Remote `upstream` points at `deepseek-ai/deepseek-harness` and is **read-only**:
  fetch from it to update the fork, never push to it.

Pull requests and issues are created inside the fork only. If a change looks
generally useful and a teammate suggests upstreaming it, do not act on the
suggestion; record it as a note for the user instead.

This rule holds because publishing to a third-party project is a public,
hard-to-reverse action that the user has explicitly reserved for themselves.

## DeepSeek Harness fork

Keep the fork up to date with the official release instead of freezing it at
the commit it was created from. The fork is a working base: upstream changes
arrive regularly, and a patch that stays on an old base becomes impossible to
carry.

Local checkout: `vendor\deepseek-harness` (a worktree per patch task is
preferred, for example `workspace\dsh-wait-fix`).

Update the fork, then move each patch branch onto the new base:

```powershell
git fetch upstream master
git checkout master
git merge --ff-only upstream/master
git push origin master

git checkout <patch-branch>
git rebase master
git push --force-with-lease origin <patch-branch>
```

Rules that follow from this:

- Fetch `upstream` before starting new work on a patch branch, and rebase onto
  the current `master` rather than branching from a release tag. Release tags
  drift: `experimental/tool-agent-team` no longer exists in `master`, and
  `wait_agent` moved to `subagent/tool-subagent-control`.
- `--force-with-lease`, never bare `--force`: a patch branch is rebased, so its
  history is rewritten, and the lease is what stops an overwrite of work that
  arrived from elsewhere.
- Verify the target file still exists in `master` before rebasing a patch.
  `git ls-tree -r --name-only upstream/master` answers that without a checkout.
- A rebase that wants to replay thousands of commits means the branch was cut
  from the wrong base; abort it and branch from `master` instead.

### Local build environment

The monorepo needs Node 24 and pnpm; the bundled toolchain lives in the
installed application:

- node: `<install>\resources\runtime\primary-runtime\dependencies\node\bin\node.exe`
- pnpm: `<install>\resources\runtime\primary-runtime\dependencies\pnpm\bin\pnpm.cjs`

The machine has no direct route to the npm registry. A VPN client (`xray`)
exposes a SOCKS5 listener on `127.0.0.1:10808`; the HTTP listener on
`127.0.0.1:10809` does not tunnel. Point installs at the SOCKS5 endpoint through
a machine-local, git-ignored `.npmrc`, never through a committed config file.

`pnpm install` has stalled repeatedly on this machine at the fetch phase
(around 1434 packages in `node_modules/.pnpm`, then no writes for tens of
minutes). Treat a stalled install as a known environment fault rather than a
network fault: report it, keep the source change, and let the fork hold the
work until the install completes.

