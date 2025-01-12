# Ansible Role: Tor Web Bridge Setup

This Ansible role is designed to simplify the deployment of a Tor Web Bridge. It configures Nginx, Docker, and other necessary components to set up a functional bridge. Additionally, the role supports dynamic DNS record management using external services, such as Cloudflare.

## Why This Role Exists

This role was created in response to a call from the Tor Foundation to enhance the accessibility and scalability of Tor Web Bridges. By providing an easy-to-use automation tool, this role lowers the barrier to entry for setting up and maintaining bridges, supporting the Tor network's mission of enabling free and open communication worldwide.

## Features

- Automated setup of a Tor Web Bridge.
- Configuration of Nginx as a reverse proxy.
- Deployment of Docker containers for the Tor Web Bridge and Watchtower.
- Dynamic management of DNS records via external services (e.g., Cloudflare).
- Support for IPv6 (based on insights from [this Tor Project forum thread](https://forum.torproject.org/t/webtunnel-docker-unable-to-find-ipv6-address-for-orport/16099/2), with thanks to atari!).
- ORPort is not exposed for added security. Note that this may cause the bridge to appear offline on the Tor Metrics website, but it does not affect the bridge's functionality. For more details, see the following links:
  - [Tor Project Issue #129](https://gitlab.torproject.org/tpo/anti-censorship/team/-/issues/129)
  - [Tor Community Issue #329](https://gitlab.torproject.org/tpo/web/community/-/issues/329).
- Automatic generation of SSL certificates using Let's Encrypt and certbot.
- Support for Ubuntu and Debian-based systems.

## Role Variables

The following variables can be configured in your inventory or playbook:

### General Settings

| Variable Name                    | Default Value       | Description                                                                                                                |
| -------------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `tor_web_bridge_admin_email`     | `admin@example.com` | Contact email address for the bridge administrator, used by certbot (Let's Encrypt) and for other administrative purposes. |
| `tor_web_bridge_base_domain`     | `example.com`       | Base domain for the Tor Web Bridge.                                                                                        |
| `tor_web_bridge_bridge_nickname` | `ExampleBridge`     | Unique nickname for the bridge.                                                                                            |

### Nginx Configuration

| Variable Name                           | Default Value | Description                                           |
| --------------------------------------- | ------------- | ----------------------------------------------------- |
| `tor_web_bridge_nginx_package_name`     | `nginx-light` | Name of the Nginx package to install.                 |
| `tor_web_bridge_nginx_update_nginxconf` | `true`        | Whether to overwrite the default Nginx configuration. |

### DNS Management

| Variable Name                              | Default Value | Description                                                              |
| ------------------------------------------ | ------------- | ------------------------------------------------------------------------ |
| `tor_web_bridge_manage_dns`                | `true`        | Whether to manage DNS records.                                           |
| `tor_web_bridge_manage_dns_service`        | `cloudflare`  | DNS service to use for record management (e.g., `cloudflare`).           |
| `tor_web_bridge_dns_propagation_wait_time` | `1`           | Time in minutes to wait for DNS propagation before issuing certificates. |

### Facts and State Management

| Variable Name                      | Default Value           | Description                                                                                                                    |
| ---------------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `tor_web_bridge_internal_hostname` | *Generated dynamically* | Internal hostname for the bridge. If not provided, it will be generated automatically and saved on each host in custom facts.  |
| `tor_web_bridge_web_path`          | *Generated dynamically* | Web path for accessing the bridge. If not provided, it will be generated automatically and saved on each host in custom facts. |

The final bridge URL will look like:

```
https://<tor_web_bridge_internal_hostname>.<tor_web_bridge_base_domain>/<tor_web_bridge_web_path>
```

The `tor_web_bridge_base_domain` must be set by the user. The `tor_web_bridge_internal_hostname` and `tor_web_bridge_web_path` can be set optionally. If they are not provided, the role will generate them dynamically and store the values in custom facts on the target host.

## Supported Platforms

- Ubuntu: `focal`, `jammy`
- Debian: `bullseye`, `bookworm`

## Dependencies

This role requires the following Ansible collections:

```yaml
collections:
  - name: community.docker
    version: ">=2.10.1"
  - name: community.general
    version: ">=5.8.0"
```

Install dependencies using:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Example Playbook

```yaml
---
- name: Install Tor Web Bridge
  hosts: webbridges
  remote_user: root
  vars:
    tor_web_bridge_admin_email: "admin@example.com"
    tor_web_bridge_base_domain: "example.com"
    tor_web_bridge_manage_dns_service: "cloudflare"
  tasks:
    - name: Include the Tor Web Bridge role
      ansible.builtin.include_role:
        name: tor_web_bridge
```

## Configuring External DNS Services

This role supports dynamic DNS record management via services like Cloudflare. To use Cloudflare:

1. Set `tor_web_bridge_manage_dns: true` in your variables.
2. Specify `tor_web_bridge_manage_dns_service: cloudflare`.
3. Ensure you have a valid Cloudflare API token, stored securely (e.g., encrypted with `ansible-vault`).

Example `group_vars/all.yml`:

```yaml
tor_web_bridge_manage_dns: true
tor_web_bridge_manage_dns_service: cloudflare
cloudflare_api_token: "{{ lookup('env', 'CLOUDFLARE_API_TOKEN') }}"
```

## License

This project is licensed under the MIT License. See the LICENSE file for details.

## Contributing

Contributions are welcome! Please submit issues and pull requests on the [GitHub repository](https://github.com/iprok/tor_web_bridges).

## Author Information

This role was created by Ivan Prokudin.

