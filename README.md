# es-python for Ansible

**Install es-python, the EyalSec Python runtime, on every host in your inventory. One API key covers the fleet, and each host reports to your EyalSec dashboard as its own machine.**

[![Website](https://img.shields.io/badge/website-eyalsec.com-e8b04b)](https://eyalsec.com)
[![Docs](https://img.shields.io/badge/docs-install-2a2c27)](https://eyalsec.com/docs/install)

You already describe your servers in Ansible. This collection adds one role to
that description: it installs es-python for the user your Python service runs
as, the same way the install command on the EyalSec Machines page does, on as
many hosts as your inventory holds. Run it on every play; a host that already
has es-python is left alone.

es-python runs your Python app unchanged and follows untrusted input to the SQL
query, shell command or file path that uses it. You choose, per machine,
whether it reports each flow or also stops it.

**For:** teams with an EyalSec account that includes es-python, on Linux hosts
(x86_64 or aarch64), Python 3.9 to 3.14, Ansible 2.15 or newer.

## Quick start

1. On your EyalSec dashboard, open **Settings > API access** and create a
   full-access key ([API keys](https://eyalsec.com/docs/api-keys)). Installing
   is a write, so a read-only key is refused.
2. Install the collection:

   ```bash
   ansible-galaxy collection install git+https://github.com/EyalSec/ansible-collection-es-python.git,v1.0.0
   ```

3. Put the key in Ansible Vault:

   ```bash
   ansible-vault encrypt_string --name es_python_api_key 'es2_...' >> group_vars/python_apps/vault.yml
   ```

4. Add the role to a play:

   ```yaml
   - name: Install es-python
     hosts: python_apps
     gather_facts: false
     vars:
       es_python_user: app             # the user your service runs as
       es_python_authorized: true      # see "Authorization" below
       es_python_python_version: "3.12"
     roles:
       - role: eyalsec.es_python.es_python
   ```

Then run your service with `/home/app/.local/bin/es-python` where it ran
`python`, and open the Events page. See
[Running es-python](https://eyalsec.com/docs/running-es-python).

## Variables

| Variable | Default | Meaning |
|---|---|---|
| `es_python_api_key` | `""` | A full-access API key. One key serves every host. Keep it in Vault. |
| `es_python_install_secret` | `""` | Instead of a key: a one-time install secret for one machine, from the Machines page. It works once and expires 10 minutes after it was issued. |
| `es_python_authorized` | `false` | Required with an API key. See [Authorization](#authorization). |
| `es_python_user` | `""` | The user es-python is installed for. Empty means the user Ansible connects as. |
| `es_python_machine` | `{{ inventory_hostname }}` | The machine name each host reports under. Must be unique per host. |
| `es_python_python_version` | `""` | `"3.9"` to `"3.14"`, quoted. Empty matches the host's own `python3`. |
| `es_python_url` | `https://eyalsec.com` | Your EyalSec address. |
| `es_python_force` | `false` | Install again on hosts that already have es-python, for example to update. |
| `es_python_install_dependencies` | `true` | Install `curl`, `tar`, `gzip` and CA certificates when they are missing. Needs `become` to root. |

## How it works

For each host the role:

1. checks whether es-python is already installed for the user, and stops there
   if it is (the host reports `ok`, not `changed`);
2. finds the machine named `es_python_machine` on your account, or adds it with
   its build left to detection, exactly as **Add machine** does by default;
3. asks for a fresh one-time install secret for that machine;
4. runs the EyalSec installer as the user, reporting the host's processor, C
   library, distribution and Python version so the server picks the right
   build.

es-python installs into the user's home (`~/opt/`, `~/.local/bin/es-python`,
`~/.eyalsec/`) and changes nothing system-wide. See
[What it installs](https://eyalsec.com/docs/install#what-it-installs).

## Keeping the key secret

The key reaches the installer on standard input. It is never on a command
line, in an environment variable Ansible prints, or in a log: the task that
carries it is `no_log`, and the installer's own output never contains it, even
at `-vvv`. On the host, the key lives only in the memory of the install step
and in a 0600 file inside a private temporary directory, removed when the step
ends.

## Authorization

`es_python_authorized: true` is your confirmation that you own these hosts or
are authorized to monitor them, that you will not use es-python to surveil
third parties, and that you are not located in or a national of an embargoed
destination and not on a denied-party list. It is the same statement as the
tick box on the Machines page, and each install records it in your
[activity log](https://eyalsec.com/docs/activity-log). es-python may capture
values from your program's memory, including secrets and personal data, when
untrusted data reaches a monitored operation, so only run it on systems you are
authorized to monitor.

## Things to know

- **Each host is one machine** and counts toward your
  [machine limit](https://eyalsec.com/docs/plans#machine-limit). When a host is
  retired, delete its machine on the Machines page so it stops counting.
- **Machine names must be unique.** Two hosts with the same
  `es_python_machine` share one machine, and the second install stops the
  first host from reporting.
- **Become, and whose home.** The role installs for `es_python_user` even when
  Ansible reaches that user through `sudo`; it does not follow `SUDO_USER` into
  the connecting account's home the way a hand-run install would.
- **Updating.** Set `es_python_force: true` for one run to reinstall the
  newest build on every host; the machine keeps its name, settings and events.
- **Sources.** A new machine watches network input and file reads from its
  first run. Change that per machine in
  [Configure](https://eyalsec.com/docs/configure).

## Getting es-python

es-python needs an EyalSec account. To get one,
[book a live demo](https://eyalsec.com/contact); pricing is at
[eyalsec.com/pricing](https://eyalsec.com/pricing).

Questions about this role: open an issue. Anything about your account:
support@eyalsec.com.

## License

MIT. See [LICENSE](LICENSE).
