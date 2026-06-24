# AGENTS.md

This file provides guidance to AI agents (such as Claude Code and Antigravity) when working with this repository.

## Overview

`ansible-nsm` provides Ansible configuration to deploy Network Security Monitoring (NSM) tools for Arch Linux, such as Zeek (Bro), Snort, PulledPork, Barnyard2, and MySQL.

## Conventions & Rules

- **Testing Toolchain**: Ensure the virtual environment is activated before running tests:
  ```bash
  uv sync
  source .venv/bin/activate
  yamllint .
  ansible-lint
  molecule test
  ```
- **Idempotency**: All Ansible tasks must be idempotent and safely re-runnable.
- **Variables**: Never hardcode IP addresses, internal hostnames, or interfaces. Use Ansible facts (e.g., `ansible_default_ipv4.interface`) or standard variables.
- **Secrets**: Never commit secrets, passwords, or tokens.
- **Documentation**: Keep `README.md` command-first, concise, and scannable. Do not add machine-specific tool paths directly to committed files.
