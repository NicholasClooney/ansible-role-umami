# Ansible Role: Umami

Deploys [Umami](https://umami.is) using Docker Compose with hardened defaults. The role lays down a production-focused `docker-compose.yml`, manages secrets via a `.env` file, and ensures the stack is running.

## Requirements

- Target host already has Docker Engine and Docker Compose v2.
- Python dependencies for the `community.docker` collection are available on the controller.
- Install the required Ansible collection:

  ```bash
  ansible-galaxy collection install -r requirements.yml
  ```

- (optional) Set up dev/test tooling:

  ```bash
  pip install -r requirements.txt
  ```

## Role Variables

All variables live in `defaults/main.yml`. Key options:

| Variable | Default | Description |
| --- | --- | --- |
| `umami_base_dir` | `/usr/local/umami` | Directory that stores compose files, env vars and secrets |
| `umami_timezone` | `Europe/London` | Time zone injected into the containers |
| `umami_bind_address` | `127.0.0.1` | Host interface exposed to the reverse proxy |
| `umami_listen_port` | `3000` | Host port forwarded to Umami |
| `umami_db_name` | `umami` | PostgreSQL database name |
| `umami_db_user` | `umami` | Database user |
| `umami_db_password` | `""` | If empty, the role generates and persists a 32-char password |
| `umami_app_secret` | `""` | If empty, the role generates and persists a 64-char hex secret |
| `umami_compose_project_name` | `umami` | Docker compose project name |
| `umami_compose_pull` | `true` | Whether to pull the latest images on each run |
| `umami_compose_recreate` | `auto` | Compose recreate behaviour (`auto`, `always`, `never`) |

## Secrets

If you omit `umami_db_password` or `umami_app_secret`, the role generates cryptographically strong values and stores them inside `{{ umami_base_dir }}/.db_password` and `{{ umami_base_dir }}/.app_secret` to remain idempotent across runs.

## Example Playbook

```yaml
- hosts: umami_servers
  become: true
  roles:
    - role: ansible-role-umami
      vars:
        umami_timezone: Europe/London
        umami_bind_address: 127.0.0.1
        umami_listen_port: 3000
        # Optional overrides
        # umami_db_password: "your-strong-password"
        # umami_app_secret: "your-64-char-hex"
```

## Reverse Proxy

The role binds Umami to the loopback address by default. Publish HTTPS with a reverse proxy (Nginx, Caddy, Traefik) on the same host, forwarding to `http://127.0.0.1:3000`.

## Testing With Molecule

A sample Molecule scenario is included. To run it (requires Docker locally):

```bash
pip install molecule[docker]
molecule test
```

## License

MIT

## Author Information

Nicholas Clooney
