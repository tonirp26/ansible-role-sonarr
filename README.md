# Ansible Role: Sonarr

This role installs the Sonarr application on Debian and also installs the recommended configuration for TV series or anime, following the TRaSH guide for Sonarr.

After running the role, the `sonarr` service will be available on the server. The application's web interface can be accessed at `http://{your-container-ip}:8989`.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
sonarr_repository_path: "/opt/Sonarr"
```

Sets the repository destination path; if modified, make sure that the system user `sonarr` still has the rights to make modifications.

```yaml
sonarr_version: "4.0.20.3014"
```

Set the desired version of the app; this will be used to get the desired Sonarr prebuilt release.

```yaml
sonarr_systemd_template: sonarr.service.j2
```

The template to use when generating the systemd service.

```yaml
sonarr_auth_username: "admin"
sonarr_auth_password: "password"
```

Sonarr credentials to create on the first start. Both values must be set using Ansible Vault or another secret manager; the ones provided are for light or temporary use. A production Sonarr instance must have at least a strong private password.

When set, the role writes them to `/etc/default/sonarr` with root-only permissions. Sonarr creates the user only if its database does not already contain one, so changing these values later does not update an existing user. The username is lowercased by Sonarr.

Keep the password in an Ansible Vault-encrypted variable rather than plain text:

```yaml
sonarr_auth_username: admin
sonarr_auth_password: "{{ vault_sonarr_password }}"
```

This is an example of how to set the password variable using Ansible Vault.

```yaml
  sonarr_anime_setup: false
```

> [!WARNING]
> This role uses the separation recommended in the TRaSH guides, so it is intended to be used only for TV series or only for anime. If both are needed, use two Sonarr containers: one for TV series and another for anime.

This boolean variable indicates whether the configuration is for a TV Sonarr or an anime Sonarr. Using the TV series configuration first and then the anime one in the same container can lead to the creation of two profiles, but it does not use the recommended TRaSH configuration for having both in the same Sonarr instance.

```yaml
  sonarr_quality_sizes_tv:
    .....

  sonarr_quality_sizes_anime:
    ....
```

By default, both variables use the quality file sizes recommended in the TRaSH guide for each profile, but they can be changed to the desired ones.

```yaml
  sonarr_naming_scheme:
    ...
```

The naming scheme variable uses the [standard|daily|anime] episode, season and series formats recommended in the TRaSH guide by default. This does not need an anime variation because it is the same. It also includes options such as the desired multiEpisodeStyle.

`colonReplacementFormat` and `multiEpisodeStyle` use a number to set the desired option; the number corresponds to its position in the drop-down list in the UI.

```yaml
sonarr_tv_custom_formats:
  - av1.json
  ...

sonarr_anime_custom_formats:
  - anime-bd-tier-01.json
  ...
```

List of TRaSH custom formats to install. Each entry is a JSON file stored in `files/custom_formats/` and copied as-is from the [TRaSH Guides collection](https://trash-guides.info/Sonarr/sonarr-collection-of-custom-formats/). The role converts the TRaSH definition into the format expected by the Sonarr API and only adds formats that do not exist yet.

To add another format, save its TRaSH JSON file in `files/custom_formats/` and add the filename to `sonarr_tv_custom_formats` or `sonarr_anime_custom_formats`.

```yaml
sonarr_media_management:
  ....
```

This variable allows modifying some settings in the media management section. In this role only one is strictly necessary: `downloadPropersAndRepacks`, because `enableMediaInfo` is set to true by default and is only used to set a desired state and for documentation.

```yaml
sonarr_default_quality_profiles:
 - Any
 ...
```
> [!WARNING]
> All quality profiles whose name appears in this list will be deleted.

Every fresh Sonarr installation comes with a set of default quality profiles; these profiles are not necessary, so it is preferable to delete them. This process has an inconvenience: every execution of this role deletes all custom quality profiles whose name appears in this list, so none of the quality profiles can use a name from this variable.

> [!NOTE]
> To set a quality profile with a name that appears in the list, it is possible to manually delete the quality profile, modify this variable so it does not include the name.

```yaml
sonarr_quality_profiles_tv:
  - web-1080p.json

sonarr_quality_profiles_anime:
  - [anime]remux-1080p.json
```

List of quality profiles to install. Each entry is a JSON file stored in `files/quality_profiles/` that describes the desired profile using quality, group, and custom-format names. The role resolves the corresponding IDs from Sonarr's `/api/v3/qualityprofile/schema` endpoint, so the profile files do not need installation-specific IDs.

The `items` list enables qualities or quality groups and can restrict the qualities inside a group:

```json
"items": [
  {
    "name": "WEB 1080p",
    "allowed": true,
    "items": ["WEBRip-1080p", "WEBDL-1080p"]
  },
  {
    "name": "Bluray-2160p Remux",
    "allowed": false
  }
]
```

The order of the `items` list is the quality preference order: the first entry appears at the top of the Sonarr quality list and is the most preferred. Qualities inside the same group are treated as equal. Qualities and groups not listed are appended below as disabled.

The role updates an existing profile when its settings differ from the file, and creates it when it is missing.

The `formatItems` list sets scores for specific custom formats. All other installed custom formats are added with a score of `0`, as required by Sonarr:

```json
"formatItems": [
  { "name": "AMZN", "score": 75 }
]
```

It can be used to have more than one profile, such as one for TV series 1080p and another for 4K. To add another profile, save its definition in `files/quality_profiles/` and add the filename to `sonarr_quality_profiles_tv` or `sonarr_quality_profiles_anime`.

```yaml
  sonarr_language_only: 1
```

This variable allows setting the desired media language using reverse scoring. Each language is represented by a number inside Sonarr; you can see a list with some languages and their corresponding numbers in the list below.

## Sonarr language IDs

| ID | Language  | ID | Language  | ID | Language    | ID | Language       | ID | Language         |
|----|-----------|----|-----------|----|-------------|----|----------------|----|------------------|
| 0  | Unknown   | 1  | English   | 2  | French      | 3  | Spanish        | 4  | German           |
| 5  | Italian   | 6  | Danish    | 7  | Dutch       | 8  | Japanese       | 9  | Icelandic        |
| 10 | Chinese   | 11 | Russian   | 12 | Polish      | 13 | Vietnamese     | 14 | Swedish          |
| 15 | Norwegian | 16 | Finnish   | 17 | Turkish     | 18 | Portuguese     | 19 | Flemish          |
| 20 | Greek     | 21 | Korean    | 22 | Hungarian   | 23 | Hebrew         | 24 | Lithuanian       |
| 25 | Czech     | 26 | Arabic    | 27 | Hindi       | 28 | Bulgarian      | 29 | Malayalam        |
| 30 | Ukrainian | 31 | Slovak    | 32 | Thai        | 33 | Portuguese(BR) | 34 | Spanish (Latino) |
| 35 | Romanian  | 36 | Latvian   | 37 | Persian     | 38 | Catalan        | 39 | Croatian         |
| 40 | Serbian   | 41 | Bosnian   | 42 | Estonian    | 43 | Tamil          | 44 | Indonesian       |
| 45 | Macedonian| 46 | Slovenian | 47 | Azerbaijani | 48 | Uzbek          | 49 | Malay            |
| 50 | Urdu      | 51 | Romansh   | 52 | Georgian    | 53 | Irish          | -2 | Original         |

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
sonarr_version: "4.0.20.3012"
```

## License

## Author Information

This role was created in 2026 by [tonirp](https://github.com/tonirp26)
