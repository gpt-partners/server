# server

The server layer: everything on the node below Kubernetes (OS and updates, SSH, Docker, wg-easy VPN,
dnsmasq split DNS, k3s, restic backups, the one-time Argo CD bootstrap). Applied by `ansible-pull` on the
node from `main`, within 15 minutes of a merge. The README has the details; this file holds the rules.

## Rules
- **This repository is public.** No secrets and no per-node values (addresses, hostnames, keys, tokens)
  in git; they stay on the node in `/etc/node-infra/*.env`, mapped in `group_vars/all.yml`.
- **Pull, not push.** No CI job or tool gets access to the node; the node applies `main` itself.
- **Nothing above k3s.** Kubernetes manifests of apps or platform services belong in gpt-partners/platform
  or the app repos. After the bootstrap, Ansible touches only the host Services (`host_services`).
- Pinned versions (k3s, restic, packages where it matters); Renovate proposes updates.
- Roles are idempotent; a change must be safe to apply on a running node every 15 minutes.
- Claude proposes changes here; the user reviews and merges them. Don't push.
