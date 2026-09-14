# Gouvernance & sécurité des repos — POPolitics

Ce dossier centralise les règles de protection appliquées à **tous les repos** de l'org
`Popolitics`. Elles font respecter automatiquement les conventions déjà écrites dans
[`documentation/CONTRIBUTING.md`](https://github.com/Popolitics/documentation/blob/main/CONTRIBUTING.md)
(nommage de branches, Conventional Commits, process de PR) et la
[Definition of Done](https://github.com/Popolitics/documentation/blob/main/02-technologie/05-definition-of-done.md).

---

## 1. Les fichiers de ce dossier

| Fichier | Rôle |
|---|---|
| `ruleset-protect-main.json` | Protège `main` : PR obligatoire, 2 reviews, CODEOWNERS, CI verte, historique linéaire, pas de force-push ni suppression, message de commit conforme. |
| `ruleset-branch-naming.json` | Force le nommage des branches (`feat/PROJ-123-...`, etc.) sur toutes les branches sauf `main`/`develop`. |

Ce sont des **rulesets GitHub exportés en JSON** : au lieu de cocher ~15 cases à la
main dans l'UI, tu les **importes** et GitHub crée toutes les règles d'un coup. Le
fichier est versionné ici → tes règles de gouvernance sont documentées, review-ables
et réappliquables à l'identique sur chaque repo.

---

## 2. Prérequis (important — repos privés)

Sur des repos **privés**, les rulesets ne sont **pas** disponibles sur le plan
**GitHub Free** : il faut GitHub Pro, Team ou Enterprise Cloud. Deux options :

1. **GitHub Team via GitHub Education** — gratuit pour une organisation éligible
   (notamment un enseignant vérifié ; un club peut aussi passer par son enseignant).
   Vérifier l'éligibilité avant de compter dessus.
2. À défaut : s'appuyer uniquement sur les hooks locaux + GitHub Actions (§5), qui
   restent gratuits même en privé mais sont contournables en local.

---

## 3. Importer un ruleset

Par repo (ou une seule fois au niveau org si plan Team) :

1. `Settings` → `Rules` → `Rulesets` → `New ruleset` → **`Import a ruleset`**.
2. Sélectionner le fichier JSON.
3. Vérifier / ajuster (ex. nom du status check `ci`), puis **Create**.

> **Niveau org (recommandé avec Team)** : `github.com/organizations/Popolitics/settings/rules`
> → cible **All repositories** → importe une fois, appliqué partout. Sinon, importe
> repo par repo.

### Ajuster avant import
- `required_status_checks` → les contexts doivent correspondre exactement aux checks
  publiés par GitHub Actions. Ce ruleset attend `ci / branch-name` et
  `ci / commit-messages`, produits par `pr-checks.yml`. Ajoute les checks de la CI
  applicative de chaque dépôt (tests, lint, build) avant l'import.
- `required_approving_review_count` = `2` (aligné sur CONTRIBUTING). Si l'équipe est
  petite, `1` peut être plus réaliste au démarrage.
- `allowed_merge_methods` = `["squash"]` → pense à désactiver merge-commit/rebase dans
  `Settings → General → Pull Requests` pour rester cohérent.

---

## 4. Ordre d'installation par repo (les repos sont encore vides)

On ne peut pas brancher l'outillage sur un repo vide. Séquence par repo :

1. **Scaffold** le projet selon la stack (voir §5).
2. **Hooks locaux** (feedback avant le push).
3. **Workflow CI** (`.github/workflows/ci.yml`) et ses checks requis (tests, lint,
   build).
4. **Importer les rulesets** (§3).
5. Ajouter **`CODEOWNERS`** + **`.github/PULL_REQUEST_TEMPLATE.md`**.

---

## 5. Outillage local par stack

Deux familles dans le projet :

### Node / TypeScript — `frontend` (Next.js)
```bash
npx create-next-app@latest .           # scaffold
npm i -D husky @commitlint/cli @commitlint/config-conventional lint-staged
npx husky init
echo "module.exports = { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
echo 'npx --no -- commitlint --edit "$1"' > .husky/commit-msg
echo 'npx lint-staged' > .husky/pre-commit
```

### Python — `api`, `auth-service`, `data-service`, `ia-service` (Django), `data` (Kestra)
Utiliser le framework **`pre-commit`** (équivalent Python de husky) :
```bash
pip install pre-commit
```
`.pre-commit-config.yaml` :
```yaml
repos:
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.6.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff        # lint
      - id: ruff-format # format
```
```bash
pre-commit install --hook-type commit-msg --hook-type pre-commit
```

---

## 6. Sécurité applicative (gratuit, à activer partout)

Dans `Settings → Code security` de chaque repo (ou org-wide) :

- **Secret scanning + Push protection** — bloque un push contenant une clé/token
  (critique pour `auth-service`).
- **Dependabot** (alerts + security updates) — CVE sur les dépendances.
- **CodeQL / Code scanning** — analyse statique de sécurité.

---

## 7. Récapitulatif des conventions forcées

| Convention (CONTRIBUTING) | Forcée par |
|---|---|
| Branche `feat/PROJ-123-...` (et autres catégories Conventional Commits) | `ruleset-branch-naming.json` |
| Conventional Commits | `ruleset-protect-main.json` (commit_message_pattern) + hook local |
| PR obligatoire, 2 reviews | `ruleset-protect-main.json` (pull_request) |
| Review par le responsable | `CODEOWNERS` + `require_code_owner_review` |
| Contrôles de gouvernance verts avant merge | `required_status_checks` + `pr-checks.yml` |
| Tests/lint/build verts avant merge | Ajouter leurs checks au `required_status_checks` de chaque dépôt |
| Pas de push direct / force-push sur `main` | `ruleset-protect-main.json` |
