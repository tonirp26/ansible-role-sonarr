# Ansible Role: Sonarr

This role installs the Sonarr application in Debian.

After running the role, the `sonarr` service will be available on the server. The application's web interface can be accessed at `http://{your-container-ip}:8989`.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
app_repository_path: "/opt/Sonarr"
```

Set repository destination path, if modified make sure that system user `sonarr` can made changes.

```yaml
app_version: "4.0.20.3014"
```

Set desired version of the app, this will be used to get desired Sonarr prebuild release.

```yaml
systemd_template: sonarr.service.j2
```

The template to use when generating systemd service.

## Dependencies

None.

## Example Playbook

```yaml
- hosts: server
  vars_files:
    - vars/main.yml
  roles:
    -  sonarr
```

*Inside `vars/main.yml`*:

```yaml
app_version: "4.0.20.3012"
```

## License

## Author Information

This role was created in 2026 by [tonirp](https://github.com/tonirp26)
