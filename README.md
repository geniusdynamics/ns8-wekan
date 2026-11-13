# ns8-wekan

Experience efficient task management with WeKan - the Open-Source, customizable, and privacy-focused kanban.

Module id: `wekan`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/wekan:latest 1
```

The command returns the instance id, for example `wekan1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/wekan1 --data - <<'EOF'
{
  "host": "wekan.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'wekan.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/mongo` | `9.0.8` |
| `ghcr.io/wekan/wekan` | `v13.09` |

## Network and access

- Reserves 1 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/database.env`
- `state/wekan.pg_dump`
- `volumes/wekan-app`

## Services

- `mongo-app.service`
- `wekan-app.service`
- `wekan.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/wekan1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/wekan:latest","instances":["wekan1"],"force":true}'
remove-module --no-preserve wekan1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- tests/wekan.robot generated from template: set real configure payload and URL check
- metadata: categories empty (use standard categories)
- mongo-app.service: uses %S/state, kickstart uses %E/state
- wekan-app.service: uses %S/state, kickstart uses %E/state
- wekan.service: uses %S/state, kickstart uses %E/state
- discover-smarthost still present: remove if the app does not use SMTP
- mongo-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- wekan-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- wekan.service: %S/state changed to %E/state (review the bind mounts that remain)
- imageroot/etc/state-include.conf: smarthost lines removed, check the unit still starts
- imageroot/actions/restore-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/actions/configure-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/actions/configure-module/10configure_environment_vars: smarthost lines removed, check the unit still starts
- imageroot/systemd/user/wekan-app.service: smarthost lines removed, check the unit still starts
- MONGO_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- POSTGRES_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- WEKAN_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://wekan.github.io/

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
