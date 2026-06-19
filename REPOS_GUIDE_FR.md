# Guide du Navigateur de Dépôts Serveurs

Ce guide explique comment ajouter votre dépôt serveur au navigateur Better Mods Manager et comment BMM vérifie l'intégrité des dépôts.

---

## Structure de repos.json

Le fichier `repos.json` contient la liste des dépôts serveurs que les utilisateurs peuvent parcourir et auxquels ils peuvent se connecter directement depuis BMM.

**Hébergé à :** `https://raw.githubusercontent.com/BetterDCS/Better_ModManager_ServerBrowse/main/repos.json`

> **Important :** Seules les entrées avec un champ `hash` valide sont affichées dans le navigateur. Les entrées sans hash sont silencieusement filtrées.

---

### Champs Obligatoires

| Champ | Type | Description |
|-------|------|-------------|
| `name` | string | Nom d'affichage du dépôt |
| `url` | string | URL directe vers le fichier `repo.json` du dépôt |
| `id` | string | Identifiant unique (kebab-case, sans espaces) |
| `category` | string | `"official"` ou `"partner"` |
| `mods_count` | number | Nombre total de mods dans le dépôt |
| `size` | number | Taille totale en octets |
| `version` | string | Version du dépôt (ex: `"1.0"`) |
| `last_update` | string | Date de dernière mise à jour — format `YYYY-MM-DD` |
| `hash` | string | **Hash de vérification défini par l'équipe BMM.** Les entrées sans ce champ ne sont pas affichées dans le navigateur. |

---

### Champs Optionnels

| Champ | Type | Description |
|-------|------|-------------|
| `signature` | string | Signature Ed25519 du contenu du `repo.json` au moment de la validation par l'équipe BMM. Utilisée pour les **vérifications d'intégrité** — si le propriétaire du dépôt modifie son contenu après validation, BMM détecte la différence et affiche **Non Vérifié**. |
| `whitelist_enabled` | boolean | `true` = serveur fermé (badge Whitelist), `false` = serveur ouvert (badge Open), absent = aucun badge. |
| `changelog_link` | string | URL vers un fichier markdown de changelog |
| `tags` | array | Tags de recherche (ex: `["DCS", "Stable"]`, max 3 affichés) |
| `region` | string | Région du serveur — voir tableau ci-dessous |
| `ping` | number \| null | Ping moyen en millisecondes (`null` si inconnu) |
| `description` | string | Courte description du dépôt |
| `website_link` | string | URL vers le site web du dépôt |
| `discord_link` | string | URL d'invitation Discord |

---

### Valeurs de Région

| Valeur | Région |
|--------|--------|
| `EU` | Europe |
| `US` | Amérique du Nord |
| `SA` | Amérique du Sud |
| `Asia` | Asie |
| `Africa` | Afrique |
| `Oceania` | Océanie |

---

### Valeurs de Catégorie

| Valeur | Badge | Description |
|--------|-------|-------------|
| `"official"` | Vert `Official` | Maintenu par l'équipe BMM |
| `"partner"` | Bleu `Partner` | Membres de confiance de la communauté |

---

## Système de Vérification

BMM effectue **deux vérifications indépendantes** à chaque fois qu'un utilisateur récupère un dépôt :

### 1. Auto-Signature (Ed25519)
BMM appelle `verify_repo_signature` pour vérifier que le `repo.json` est correctement signé par son `author_id` déclaré. Cela valide que le fichier n'a pas été corrompu en transit.

- ✅ **VÉRIFIÉ** — auto-signature valide
- ❌ **NON VÉRIFIÉ** — pas de signature, clé invalide, ou JSON corrompu

### 2. Intégrité du Contenu (signature vs. repos.json)
Si `repos.json` inclut un champ `signature` pour cette entrée, BMM compare la **signature live du dépôt** avec celle enregistrée au moment de la validation.

- ✅ **VÉRIFIÉ** — la signature live correspond à celle enregistrée (contenu inchangé)
- ❌ **La signature ne correspond plus** — le propriétaire du dépôt a modifié son contenu après la validation par l'équipe BMM. Les utilisateurs voient un avertissement dédié dans les Détails du Dépôt.

> **Pour l'équipe BMM :** lors de la validation d'un nouveau dépôt, enregistrez la valeur de `repo.signature` dans l'entrée `repos.json`. Toute modification ultérieure par le propriétaire sera automatiquement détectée.

---

## Exemple de repos.json

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
    "description": "Dépôt maintenu par la communauté avec des mods expérimentaux",
    "website_link": "https://community-server.com",
    "discord_link": "https://discord.gg/community-invite"
  }
]
```

---

## Comment Soumettre Votre Dépôt

1. **Générez votre dépôt** via **Server Repo → Generate Repo** dans BMM
2. **Hébergez votre `repo.json`** sur un serveur web ou GitHub Pages
3. **Signez votre dépôt** avec les outils d'hébergement BMM — cela renseigne `author_id` et `signature` dans `repo.json`
4. **Remplissez l'entrée** avec les champs ci-dessus — laissez vides `hash` et `signature` (définis par l'équipe BMM lors de la validation)
5. **Soumettez une pull request** sur [BetterDCS/Better_ModManager_ServerBrowse](https://github.com/BetterDCS/Better_ModManager_ServerBrowse)
6. L'équipe BMM **valide** l'entrée, définit le `hash`, enregistre la `signature`, puis fusionne la PR

---

## Calcul de la Taille

La taille totale en octets provient du champ `total_size_bytes` du fichier `Info.json` généré par BMM lors de l'export d'un dépôt.

---

## Format du Changelog

```markdown
# Changelog

## v1.0.0 (2026-05-15)
- Version initiale
- Ajout de 10 mods
- Taille totale : 5.14 GB
```

---

## Options d'Hébergement

### GitHub Pages
1. Créez un dépôt avec vos fichiers de dépôt
2. Activez GitHub Pages dans les paramètres du dépôt
3. Utilisez l'URL GitHub Pages pour votre `repo.json`

### Serveur Web
1. Uploadez vos fichiers sur votre serveur
2. Configurez les en-têtes CORS si nécessaire
3. Utilisez l'URL de votre serveur pour `repo.json`

### Serveur BMM Intégré
1. Utilisez **Start Local Host** dans BMM
2. Configurez le port et les paramètres réseau
3. Partagez votre IP locale avec les utilisateurs de votre réseau

---

## Test de Votre Dépôt

Avant de soumettre, testez localement :

1. Ouvrez BMM
2. Allez dans **Server Repo → Synchronize from a Repository**
3. Entrez l'URL de votre `repo.json`
4. Cliquez sur **FETCH**
5. Vérifiez que le badge **Vérifié** apparaît et que les mods/profils s'affichent correctement

---

## Support

Pour les questions ou problèmes, visitez le [Discord BMM](https://discord.com/invite/CTaaEF9R75) ou le [dépôt GitHub](https://github.com/FreeProject089/BetterModsManager).
