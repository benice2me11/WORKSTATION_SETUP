# Workstation Setup

Copy-paste friendly Ubuntu workstation bootstrap guide for local development and agent-assisted setup.

The main guide is here:

- [WORKSTATION_SETUP.md](WORKSTATION_SETUP.md)

## What It Covers

- Base CLI packages: `git`, `curl`, `wget`, `build-essential`, `make`, `zsh`
- Git identity and SSH key setup
- Oh My Zsh installation
- Zsh plugins:
  - `git`
  - `zsh-autosuggestions`
  - `zsh-syntax-highlighting`
- Go installation under the user's home directory
- Docker CE and Docker Compose from Docker's official APT repository
- Final verification commands
- Agent safety checklist

## Quick Start

1. Open [WORKSTATION_SETUP.md](WORKSTATION_SETUP.md).
2. Follow the sections in order.
3. Replace placeholders such as `YOUR_NAME` and `YOUR_EMAIL`.
4. Run the final verification block.
5. Restart the terminal session after changing the login shell or Docker group membership.

## Safety Notes

- Do not overwrite an existing SSH key unless the user explicitly asks for a new key.
- Print only public SSH keys. Never print or commit private keys.
- Treat membership in the `docker` group as root-level host access.
- If `sudo` requires an interactive password, stop and ask the user to run the command locally.
- Use placeholders in documentation instead of personal emails, usernames, hostnames, tokens, or absolute local paths.

## Repository Privacy

This repository is intended to stay safe for public or shared use. Documentation should use:

- `YOUR_NAME` and `YOUR_EMAIL` for Git identity examples
- `~` and `$HOME` for user paths
- generic host and account references

Do not commit:

- private SSH keys
- access tokens
- real passwords
- machine hostnames
- personal email addresses unless intentionally published
- local absolute paths; prefer `$HOME/...` or `~/...`
