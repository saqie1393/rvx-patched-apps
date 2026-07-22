# Contributing

## Signing your commits

Commits to this repo should be **signed** so they show a green **Verified** badge on
GitHub. We use **SSH commit signing** — you can reuse the same SSH key you already use to
push to GitHub; no GPG toolchain is needed.

> Signing is enabled but **not enforced** — unsigned commits are still accepted. Please sign
> anyway. CI-authored commits on the `update` branch (the daily `github-actions[bot]` version
> bumps) are intentionally left unsigned and are out of scope here.

### 1. Configure git to sign with SSH

Run inside your clone (drop the repo scope by adding `--global` if you'd rather sign in every
repo):

```bash
git config gpg.format ssh
git config user.signingkey ~/.ssh/id_rsa.pub   # your public key (any type: rsa/ed25519)
git config commit.gpgsign true
git config tag.gpgsign true                     # optional: also sign annotated tags
```

### 2. Register the key on GitHub as a *Signing Key*

A key added for **authentication** is **not** automatically usable for signing — add it again
with the signing type:

```bash
gh ssh-key add ~/.ssh/id_rsa.pub --type signing --title "commit-signing"
```

…or via **GitHub → Settings → SSH and GPG keys → New SSH key → Key type: _Signing Key_**.

> The `gh ssh-key add --type signing` call needs the `admin:ssh_signing_key` token scope:
> `gh auth refresh -h github.com -s admin:ssh_signing_key` (or just use the web UI above).

### 3. Make sure your commit email is verified on GitHub — common gotcha

GitHub only shows **Verified** when the commit's committer email is a **verified email on the
account that owns the signing key** (GitHub → Settings → Emails). If it isn't, the signature is
still cryptographically valid but shows **Unverified**. Either verify that email, or commit
with your account's `@users.noreply.github.com` address:

```bash
git config user.email <you>@users.noreply.github.com
```

### 4. (Optional) Verify signatures locally

`git log --show-signature` needs an `allowed_signers` file to print *Good signature* locally
(GitHub's badge doesn't need this):

```bash
mkdir -p ~/.config/git
printf '%s namespaces="git" %s\n' "$(git config user.email)" "$(cat ~/.ssh/id_rsa.pub)" \
  >> ~/.config/git/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers
```

Then:

```bash
git commit --allow-empty -m "test: signed commit"   # then: git reset --hard HEAD~1 to drop it
git log --show-signature -1                          # -> Good "git" signature for <you>
```
