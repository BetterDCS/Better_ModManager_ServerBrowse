# Server Repositories Browser Guide

This guide explains how to add your server repository to the Better Mods Manager browser and how BMM verifies repository integrity.

---

## repos.json Structure

The `repos.json` file contains a list of server repositories that users can browse and connect to directly from BMM.

**Hosted at:** `https://raw.githubusercontent.com/BetterDCS/Better_ModManager_ServerBrowse/main/repos.json`

> **Important:** Only entries with a valid `hash` field are displayed in the browser. Entries without a hash are silently filtered out.

---

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name of the repository |
| `url` | string | Direct URL to the repository's `repo.json` file |
| `id` | string | Unique identifier (kebab-case, no spaces) |
| `category` | string | `"official"` or `"partner"` |
| `mods_count` | number | Total number of mods in the repository |
| `size` | number | Total size in bytes |
| `version` | string | Version string (e.g. `"1.0"`) |
| `last_update` | string | Last update date — `YYYY-MM-DD` format |
| `hash` | string | **Verification hash set by the BMM team.** Entries without this field are not shown in the browser. |

---

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `signature` | string | Ed25519 signature of the `repo.json` content at the time the BMM team validated it. Used for **content-integrity checks** — if the repo owner changes their content after validation, BMM detects the mismatch and marks it **Unverified**. |
| `whitelist_enabled` | boolean | `true` = locked server (Whitelist badge), `false` = open server (Open badge), omit = no badge. |
| `changelog_link` | string | URL to a changelog markdown file |
| `tags` | array | Search tags (e.g. `["DCS", "Stable"]`, max 3 shown) |
| `region` | string | Server region — see table below |
| `ping` | number \| null | Average ping in milliseconds (`null` if unknown) |
| `description` | string | Brief description of the repository |
| `website_link` | string | URL to the repository's website |
| `discord_link` | string | Discord invite URL |

---

### Region Values

| Value | Region |
|-------|--------|
| `EU` | Europe |
| `US` | North America |
| `SA` | South America |
| `Asia` | Asia |
| `Africa` | Africa |
| `Oceania` | Oceania |

---

### Category Values

| Value | Badge | Description |
|-------|-------|-------------|
| `"official"` | Green `Official` | Maintained by the BMM team |
| `"partner"` | Blue `Partner` | Trusted community members |

---

## Verification System

BMM performs **two independent checks** whenever a user fetches a repository:

### 1. Self-Signature (Ed25519)
BMM calls `verify_repo_signature` to check whether the `repo.json` is correctly signed by its declared `author_id`. This validates that the file was not corrupted in transit.

- ✅ **VERIFIED** — self-signature is valid
- ❌ **UNVERIFIED** — no signature, invalid key, or broken JSON

### 2. Content-Integrity (signature vs. repos.json)
If `repos.json` includes a `signature` field for this entry, BMM compares the **live repo's signature** against the one recorded at validation time.

- ✅ **VERIFIED** — live signature matches the recorded one (content unchanged)
- ❌ **Signature no longer matches** — the repo owner changed their content after the BMM team validated it. Users see a dedicated warning in Repository Details.

> **For the BMM team:** when validating a new repo, record its current `repo.signature` value into the `repos.json` entry. Any subsequent change by the repo owner will be automatically detected.

---

## Example repos.json

```json
[
  {
    "name": "Exemple Official Repo",
    "url": "https://your-server.com/repo.json",
    "id": "dcs-root-mod-folder",
    "category": "official",
    "mods_count": 10,
    "size": 5520129295,
    "version": "1.0",
    "last_update": "2026-05-15",
    "hash": "abc123def456...",
    "signature": "a1b2c3d4e5f6...",
    "whitelist_enabled": false,
    "changelog_link": "",
    "tags": ["DCS"],
    "region": "EU",
    "ping": null,
    "description": "Official DCS World mod repository with root folder structure",
    "website_link": "https://freeproject089.github.io/BMM_Web/",
    "discord_link": "https://discord.gg/CTaaEF9R75"
  },
  {
    "name": "Community Mods",
    "url": "https://community-server.com/repo.json",
    "id": "community-mods-001",
    "category": "partner",
    "mods_count": 85,
    "size": 3221225472,
    "version": "2.1.0",
    "last_update": "2026-05-10",
    "hash": "xyz789...",
    "signature": "f6e5d4c3b2a1...",
    "whitelist_enabled": true,
    "changelog_link": "https://community-server.com/changelog.md",
    "tags": ["Community", "Experimental"],
    "region": "US",
    "ping": 120,
    "description": "Community-maintained repository with experimental mods",
    "website_link": "https://community-server.com",
    "discord_link": "https://discord.gg/community-invite"
  }
]
```

---

## How to Submit Your Repository

1. **Generate your repository** via **Server Repo → Generate Repo** in BMM
2. **Host your `repo.json`** on a web server or GitHub Pages
3. **Sign your repository** using the BMM host tools — this populates `author_id` and `signature` in `repo.json`
4. **Fill in the entry** using the fields above — omit `hash` and `signature` (set by the BMM team on validation)
5. **Submit a pull request** to [BetterDCS/Better_ModManager_ServerBrowse](https://github.com/BetterDCS/Better_ModManager_ServerBrowse)
6. The BMM team **validates** the entry, sets the `hash` and records the `signature`, then merges the PR

---

## Size Calculation

The total size in bytes comes from the `total_size_bytes` field of the `Info.json` file generated by BMM when exporting a repository.

---

## Changelog Format

```markdown
# Changelog

## v1.0.0 (2026-05-15)
- Initial release
- Added 10 mods
- Total size: 5.14 GB
```

---

## Hosting Options

### GitHub Pages
1. Create a repository with your repo files
2. Enable GitHub Pages in repository settings
3. Use the GitHub Pages URL as your `repo.json` URL

### Web Server
1. Upload repository files to your server
2. Configure CORS headers if needed
3. Use your server URL for `repo.json`

### Built-in BMM Server
1. Use **Start Local Host** in BMM
2. Configure port and network settings
3. Share your local IP with network users

---

## Testing Your Repository

Before submitting, test locally:

1. Open BMM
2. Go to **Server Repo → Synchronize from a Repository**
3. Enter your `repo.json` URL
4. Click **FETCH**
5. Check that the **Verified** badge appears and mods/profiles display correctly

---

## Support

For questions or issues, visit the [BMM Discord](https://discord.com/invite/CTaaEF9R75) or the [GitHub repository](https://github.com/FreeProject089/BetterModsManager).
