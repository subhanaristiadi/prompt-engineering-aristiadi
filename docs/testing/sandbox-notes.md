# Sandbox and GitHub workflow notes

Practical notes on running a Git/GitHub workflow from inside a confined agent
sandbox — specifically DeepSeek Harness on Windows with `workspace-write`
permissions. These are environment observations, not prompt engineering advice,
but they cost real debugging time so they are recorded here.

## What works

Commands that talk to GitHub's API directly work normally:

```sh
gh pr create --head my-branch --base main --title "..." --body "..."
gh pr list --state all --json number,title,state
gh api repos/OWNER/REPO/branches --jq '.[].name'
```

`gh` handles its own authentication from the system keyring, so nothing about
sending requests is affected by the sandbox.

## What does not

A plain `git push` fails, even with a correct credential-helper configuration:

```text
fatal: could not read Username for 'https://github.com'
```

## Why

The sandbox blocks `sh.exe` from spawning:

```text
sh: fatal error - couldn't create signal pipe, Win32 error 5
```

Git invokes credential helpers through a shell, so:

1. Git tries to spawn the helper via `sh`
2. `sh` cannot start in the confined environment
3. Git never receives credentials
4. Git falls back to prompting, which is also unavailable

The deflection is subtle: the error names *credentials*, so the natural
investigation is auth configuration. The credential helper is fine. The system
credential store is fine. `gh auth status` reports a clean login. The failure is
in the mechanism git uses to *invoke* the helper.

## Diagnosis

Confirm the cause rather than assuming it:

```sh
sh -c "echo ok"                     # fails with Win32 error 5
gh auth status                      # reports a clean login
gh auth git-credential get          # returns the credential correctly
```

If `sh` fails but `gh` returns a credential, the helper is not the problem.

## Workaround

Pass the credential as an HTTP header for the push, bypassing helper invocation:

```sh
c=$(printf 'protocol=https\nhost=github.com\n' | gh auth git-credential get)
u=$(printf '%s\n' "$c" | sed -n 's/^username=//p')
p=$(printf '%s\n' "$c" | sed -n 's/^password=//p')
export GIT_CONFIG_COUNT=1
export GIT_CONFIG_KEY_0=http.extraheader
export GIT_CONFIG_VALUE_0="Authorization: Basic $(printf '%s:%s' "$u" "$p" | base64 -w0)"
git push -u origin my-branch
```

The header is passed via environment variables rather than written to git config,
so it does not persist to disk.

## Also: TLS backend

A separate failure that presents similarly. On Windows, git often defaults to the
`schannel` TLS backend, which failed here with:

```text
schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS
```

Fix per repository:

```sh
git config http.sslBackend openssl
```

This is unrelated to the credential problem above, but the two produce confusingly
similar errors and can occur together. Fix TLS first: if the connection cannot be
established, no credential attempt happens.

## Takeaways

- Prefer `gh` commands over `git push` in confined environments
- When a git failure names credentials, verify the *invocation mechanism*, not only
  the credentials
- Sandbox confinement can break process spawning generally, not just file writes —
  a wider blast radius than the policy name suggests
- These are environment conditions, not prompt defects; a working prompt and a
  failing push look alike only if you conflate them