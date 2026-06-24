# ansible-nsm

Ansible configuration to deploy Network Security Monitoring (NSM) tools on Arch Linux.

## Components

- **Zeek (formerly Bro)**: Installs dependencies, builds from source, configures, and manages via `broctl`. Sets up a 5-minute cron job for `broctl cron`.
- **Snort / Pulledpork / Barnyard2**: Scaffolding for intrusion detection.
- **MySQL**: Local database for storing alerts.

## Configuration

Configure variables in your inventory or group/host vars (e.g., `vars/localhost.yml`):

```yaml
# Example variable overrides
bro_interface: '{{ ansible_default_ipv4.interface }}'
bro_log_dir: /var/log/bro
```

*Note: Zeek binaries are installed to `/usr/local/bro/bin`. Add this to your shell `$PATH` (e.g., `~/.bashrc` or `~/.zshrc`).*

## Usage

```bash
# Apply the playbook locally
ansible-playbook playbook.yml -i inventory.ini --ask-become-pass
```

## Testing

Run linting and testing in the managed Python virtual environment:

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```
