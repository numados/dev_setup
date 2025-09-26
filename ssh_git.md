[Index](README.md)

# Git Multi‑Repo Setup with SSH (Per Folder)

This reference explains how to configure multiple Git identities with
dedicated SSH keys.\
It uses per‑folder Git configs so each working directory applies the
correct identity automatically.

------------------------------------------------------------------------

## 1. Generate SSH Key per Identity

Create an **ed25519** key for each identity (personal, work, etc.).

``` bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519_identity
```

Add the key to the macOS agent and Keychain:

``` bash
# Older macOS OpenSSH
ssh-add -K ~/.ssh/id_ed25519_identity
# Newer OpenSSH
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_identity
```

Optional global convenience (`~/.ssh/config`):

``` ssh-config
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

------------------------------------------------------------------------

## 2. Register Key with GitHub

Copy your public key:

``` bash
pbcopy < ~/.ssh/id_ed25519_identity.pub
```

Add it in **GitHub → Settings → SSH and GPG keys**.\
Test:

``` bash
ssh -T git@github.com
```

Expected: `Hi <username>! You’ve successfully authenticated…`

------------------------------------------------------------------------

## 3. Directory‑Based Git Config (Recommended)

Use `includeIf` to apply identity and key by folder path.

### Global config (`~/.gitconfig`):

``` ini
[includeIf "gitdir:~/Work/project/**"]
    path = ~/.work.gitconfig

[includeIf "gitdir:~/Personal/identity/**"]
    path = ~/.identity.gitconfig
```

### Example identity config (`~/.identity.gitconfig`):

``` ini
[user]
    name = your-identity
    email = your_identity@example.com
[core]
    sshCommand = ssh -i ~/.ssh/id_ed25519_identity
```

Repos placed under `~/Personal/identity/` will use this config
automatically.

Verify inside a repo:

``` bash
git config user.email
git config --show-origin --get user.email
```

------------------------------------------------------------------------

## 4. Alternative: SSH Host Aliases

Instead of per‑folder configs, set host aliases in `~/.ssh/config`:

``` ssh-config
Host github-identity
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_identity

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
```

Use alias in remote URL:

``` bash
git remote set-url origin git@github-identity:owner/repo.git
```

------------------------------------------------------------------------

## 5. Per‑Repo Override (Ad Hoc)

Inside a single repo:

``` bash
git config user.name "Your Name"
git config user.email "your_identity@example.com"
git config core.sshCommand "ssh -i ~/.ssh/id_ed25519_specific"
```

------------------------------------------------------------------------

## 6. Troubleshooting

Show active configs:

``` bash
git config --show-origin --list
```

Debug SSH key usage:

``` bash
ssh -vT git@github.com
```

Fix stale host entries:

``` bash
ssh-keygen -R github.com
ssh -T git@github.com
```

[Index](README.md)
