# AGENTS.md

Guidance for AI agents working in `ansible-nsm`.

## Overview

Ansible role (Galaxy: `jahrik.nsm`) that builds a Network Security Monitoring stack -- Zeek (Bro),
Snort, PulledPork, and Barnyard2 -- from source on Arch Linux, plus an optional Conky dashboard.
`playbook.yml` at the repo root is a thin wrapper that `include_role`s this role by directory, so
`ansible-playbook playbook.yml` keeps working without the repo being checked out under a
`roles/` path.

This repo was converted from a 5-role playbook (`bro, snort, pulledpork, barnyard2, conky`) into a
single role. Each sub-role's `tasks/main.yml` folded into `tasks/<tool>.yml`; `defaults`,
`templates`, and `handlers` merged up into the top-level role dirs (no name collisions found). The
custom `library/yaourt` AUR-install module is unchanged.

## Task Flow

`tasks/main.yml` dispatches each tool via `include_tasks` with the `apply:` tag form, so a single
tool can be re-applied with `--tags nsm:<tool>`. Each dispatch is guarded by
`when: ansible_facts['os_family'] == 'Archlinux'` -- there is no non-Arch path.

| File | Tag | Notes |
|---|---|---|
| `tasks/bro.yml` | `nsm:bro` | Builds Zeek from source, configures `broctl`, 5-minute cron. |
| `tasks/snort.yml` | `nsm:snort` | Creates the `snort` user, builds from source, systemd unit. |
| `tasks/pulledpork.yml` | `nsm:pulledpork` | Downloads PulledPork, daily rule-update cron. |
| `tasks/barnyard2.yml` | `nsm:barnyard2` | Builds Barnyard2 from source with MySQL support. |
| `tasks/conky.yml` | `nsm:conky` | Optional dashboard configs for the sensor host. |

## Conventions

- **Idempotency:** build/install `command` tasks use `creates:`/`changed_when:` to stay
  re-runnable. Handlers that shell out (`broctl install`/`restart`, `pulledpork.pl -l`) use
  `ansible.builtin.command` with `changed_when: true` -- fixed from the original playbook, which
  left them without a `changed_when` at all.
- **Dependencies:** use `uv` for Python package management; do not use `pip` directly.
- **OS constraint:** all tasks are guarded by `when: ansible_facts['os_family'] == 'Archlinux'` in
  `tasks/main.yml`; nothing here supports another distro.
- **Facts:** use `ansible_facts['<fact>']`, never the bare `ansible_<fact>` magic vars --
  `ansible.cfg` sets `inject_facts_as_vars = False`. This includes template-side facts
  (`ansible_facts['hostname']`, `ansible_facts['default_ipv4']['interface']`,
  `ansible_facts['os_family']`) in `templates/*.j2`.
- **FQCN:** all built-in modules fully qualified (`ansible.builtin.*`). The custom `yaourt` module
  in `library/` is a local role module, not a collection member, so it's referenced bare.
- **remote_src:** PulledPork's `copy` tasks (`tasks/pulledpork.yml`, and the Snort handlers that
  copy build artifacts) set `remote_src: true` -- the files being copied live on the target host
  (staged there by an earlier `git`/build task), not on the Ansible controller. The original
  playbook omitted this, which meant those tasks would fail or silently no-op on a controller that
  didn't happen to have matching local paths.
- **Secrets:** `barnyard2.db_pass` defaults to `CHANGEME` in `defaults/main.yml` -- override via
  Ansible Vault in `secrets/{{ inventory_hostname }}.yml` (gitignored, see `playbook.yml`) or a
  real `group_vars`/`host_vars` file, never commit a real value.
- **General rules:** abide by the global `AGENTS.md` guidelines (no hardcoded secrets or IPs).

## Testing

```bash
uv sync
uv run ansible-galaxy collection install -r requirements.yml
uv run yamllint .
uv run ansible-lint
uv run ansible-playbook playbook.yml --syntax-check
```

## CI/CD

- **lint** -- `yamllint` + `ansible-lint` (profile `production`).
- **syntax** -- `ansible-playbook playbook.yml --syntax-check`. No Molecule/converge job: this role
  builds Zeek and Snort from source (kernel packet-capture libraries, `libpcap`/`libdaq`) and
  installs AUR packages via `yaourt` -- nothing here safely converges in a container. Lint +
  syntax-check is the CI gate for this repo.
- **release** -- `needs: [lint, syntax]`, publishes to Ansible Galaxy on merge to `main`.
