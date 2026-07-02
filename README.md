# jahrik.nsm

[![CI/CD](https://github.com/jahrik/ansible-nsm/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-nsm/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.nsm-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/nsm/)

Builds a Network Security Monitoring (NSM) stack -- Zeek (Bro), Snort, PulledPork, and Barnyard2 --
from source on Arch Linux, plus an optional Conky dashboard for the sensor host.

**Target platform: Arch Linux only.** Everything here builds from source or pulls AUR packages via
the bundled `yaourt` module; there is no equivalent for other distros.

## Usage

Include the role in your playbook:

```yaml
- hosts: all
  become: true
  roles:
    - jahrik.nsm
```

Or run the bundled thin wrapper directly:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Each tool can be re-applied on its own with tags: `bro`, `snort`, `pulledpork`, `barnyard2`,
`conky` -- e.g. `--tags nsm:snort`.

## Components

| Tool | Notes |
|---|---|
| Zeek (Bro) | Builds from source, configures `node.cfg`/`networks.cfg`/`broctl.cfg`, cron job every 5 minutes. |
| Snort | Builds from source, creates the `snort` user/dirs, systemd unit, `snort.conf`. |
| PulledPork | Downloads from GitHub, stages rule-management confs, daily 09:00 cron rule update. |
| Barnyard2 | Builds from source with MySQL support, systemd unit, writes alerts to `barnyard2.conf`. |
| Conky | Optional dashboard (`~/.conkyrc_left`, `~/.conkyrc_right`) for the sensor host. |

## Variables

Key variables (see `defaults/main.yml` for the full list):

| Variable | Default | Description |
|---|---|---|
| `bro_interface` | `{{ ansible_facts['default_ipv4']['interface'] }}` | Interface Zeek/broctl monitors. |
| `bro_log_dir` | `/var/log/bro` | Zeek log directory. |
| `nsm_home_net` | RFC1918 ranges | CIDRs treated as local/trusted by Zeek and Snort. |
| `snort.interface` | `{{ ansible_facts['default_ipv4']['interface'] }}` | Interface Snort listens on. |
| `snort.version` | `2.9.11` | Snort source version to build. |
| `barnyard2.db_pass` | `CHANGEME` | Override via Ansible Vault -- never commit a real password. |
| `pulledpork.version` | `master` | PulledPork git ref to build. |

Set real values (credentials, hostnames) in your own `group_vars`/`host_vars` or
`secrets/{{ inventory_hostname }}.yml` (gitignored -- see `playbook.yml`), and put every
`CHANGEME` in Ansible Vault -- never commit a real password.

*Note: Zeek binaries are installed to `/usr/local/bro/bin`. Add this to your shell `$PATH`.*

## Testing

```bash
uv sync
uv run ansible-galaxy collection install -r requirements.yml
uv run yamllint .
uv run ansible-lint
uv run ansible-playbook playbook.yml --syntax-check
```

CI runs lint and a syntax check only -- see [AGENTS.md](AGENTS.md) for why.

## Proposed follow-ups

- Add a Molecule scenario if/when a safe way to converge kernel packet-capture setup and AUR
  builds in a container is found -- see [AGENTS.md](AGENTS.md) for why there isn't one today.
- Snort 2.9.11 and the Zeek `master` branch are dated; consider pinning to a maintained Zeek
  release and evaluating Snort 3 as a follow-up.

## License

GPL-3.0-or-later

## Author

jahrik@gmail.com
