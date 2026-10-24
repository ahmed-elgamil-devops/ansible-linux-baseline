# Ansible Linux Baseline

![Ansible Lint](https://github.com/ahmed-elgamil-devops/ansible-linux-baseline/actions/workflows/lint.yml/badge.svg)

Ansible playbook that turns a fresh **Ubuntu** server into a secure, ready-to-use web server in one command.

## What it does

| Role | What it configures |
|------|--------------------|
| `common` | apt update, base packages (curl, git, htop, fail2ban...), automatic security updates, timezone |
| `users` | admin user in the `sudo` group, SSH public key, sudoers file validated with `visudo` |
| `ssh_hardening` | no root login, no password login, key-only auth, `MaxAuthTries 3`, `AllowUsers` (config validated with `sshd -t` before restart) |
| `firewall` | UFW: deny all incoming, allow only SSH / HTTP / HTTPS |
| `nginx` | Nginx with a hardened default site (`server_tokens off`, security headers) and a landing page |

## Project structure

```
.
├── ansible.cfg
├── site.yml                 # main playbook
├── requirements.yml         # community.general, ansible.posix
├── inventory/hosts.ini      # example inventory
├── group_vars/all.yml       # all settings in one place
├── roles/
│   ├── common/
│   ├── users/
│   ├── ssh_hardening/
│   ├── firewall/
│   └── nginx/
└── .github/workflows/lint.yml   # CI: syntax check + ansible-lint (production profile)
```

## Usage

```bash
# 1. Install Ansible and the required collections
pip install ansible-core
ansible-galaxy collection install -r requirements.yml

# 2. Edit inventory/hosts.ini and group_vars/all.yml (add your SSH public key!)

# 3. Dry run first
ansible-playbook site.yml --check --diff

# 4. Apply
ansible-playbook site.yml

# Run only one part
ansible-playbook site.yml --tags ssh,firewall
```

> ⚠️ Make sure your SSH public key is set in `users_admin_ssh_public_key` before running.
> The playbook disables password login, so without a key you can lock yourself out.

## Design choices

- **Idempotent**: safe to run many times; only changes what is different.
- **Safe changes**: `sshd` and `sudoers` configs are validated before they are applied, so a typo can't break access.
- **Handlers**: services are restarted/reloaded only when their config actually changes.
- **Role variables are prefixed** (`ssh_hardening_port`, `firewall_allowed_tcp_ports`...) and pass `ansible-lint` with the strictest `production` profile.
- **CI on every push**: GitHub Actions runs `--syntax-check` and `ansible-lint`.

## Author

**Ahmed Elgamil**, DevOps & Cloud Engineer
[LinkedIn](https://linkedin.com/in/ahmed-elgamil-devops) · [GitHub](https://github.com/ahmed-elgamil-devops)
