# ADR 0001: Kernel updates on the node

- Status: accepted
- Date: 2026-11-06
- Context: [incident 2026-11-04](../incidents/2026-11-04-kernel-7.0.0-42.md)

## Context

The node installs security updates with unattended-upgrades and reboots by itself at 04:00 UTC when
an update needs it (`roles/base`). On 2026-11-04 this booted Ubuntu kernel 7.0.0-42, which made the
Prisma processes in LiteLLM hang at start. LiteLLM was down for about 60 hours. The node is a single
k3s server, so a bad kernel takes down whatever it breaks until someone notices.

## Decision

1. Security updates, kernels included, stay automatic. Skipping them leaves known holes open, and
   manual updates get forgotten.
2. Every reboot is checked: the alerts `NodeKernelChanged` and `PodsNotReadyAfterReboot`
   (gpt-partners/platform, `alerts/node-infra.yaml`) mail info@wake-u.com.
3. A kernel known to be bad goes into `base_unattended_upgrades_blacklist`
   (`roles/base/defaults/main.yml`), so unattended-upgrades does not install it.
4. If a bad kernel is already running, the node boots the previous one by hand and keeps it:
   `apt-mark hold` on its `linux-image` and `linux-modules` packages,
   `/etc/default/grub.d/99-pin-kernel.cfg` with `GRUB_DEFAULT=saved`, `grub-set-default` on its menu
   entry, `update-grub`, reboot.
5. When Ubuntu ships a fixed kernel, the pin is removed: `apt-mark unhold`, delete
   `99-pin-kernel.cfg`, `grub-set-default 0`, `update-grub`, drop the blacklist entry.

## Alternatives considered

- **Disable kernel updates.** Rejected: leaves kernel vulnerabilities open indefinitely.
- **Update by hand.** Rejected: depends on someone remembering, for every update.
- **Several k3s nodes.** Rejected for now: the same automatic update would reach every node, so it
  does not help against a bad kernel unless updates are staggered; it also needs shared storage and
  doubles the cost. Revisit when an app promises availability.

## Later

- Boot a new kernel once on trial (`grub-reboot`) and make it the default only if pods come up.
- Canonical Livepatch, to apply critical kernel fixes without a reboot.
- Watch [Ubuntu Security Notices](https://ubuntu.com/security/notices) and
  [Launchpad: linux](https://bugs.launchpad.net/ubuntu/+source/linux) for kernel regressions.
