# Hooks git — enforcement local (Plan B, gratuit)

Deux hooks qui bloquent **sur le poste du dev**, avant que quoi que ce soit
n'atteigne GitHub :

- **`pre-push`** — refuse de pousser une branche mal nommée (`feat/PROJ-123-...`)
- **`commit-msg`** — refuse un commit hors Conventional Commits

## Méthode recommandée : `core.hooksPath` (universelle, versionnée)

Fonctionne pour **tous les repos** (Node comme Python), et les hooks sont
versionnés dans le repo au lieu de vivre dans `.git/hooks/` (non partagé).

Dans **chaque** repo, une fois :

```bash
# 1. copier les hooks depuis infra/governance/hooks vers un dossier versionné
mkdir -p .githooks
cp ../infra/governance/hooks/pre-push   .githooks/pre-push
cp ../infra/governance/hooks/commit-msg .githooks/commit-msg
chmod +x .githooks/*

# 2. committer .githooks/ dans le repo
git add .githooks && git commit -m "chore: add git hooks"

# 3. chaque dev active les hooks une fois après le clone
git config core.hooksPath .githooks
```

> L'étape 3 (`git config`) est **locale** : chaque contributeur doit la lancer
> une fois après avoir cloné. Ajoute-la au README du repo, section "Setup".
> Pour l'automatiser, voir ci-dessous.

## Automatiser l'étape 3 (optionnel, selon la stack)

### Node — `frontend` (Next.js)
Avec **husky**, l'activation se fait toute seule au `npm install` :
```bash
npm i -D husky
npx husky init                       # crée .husky/ et le script prepare
# place les checks dans .husky/pre-push et .husky/commit-msg
```

### Python — backends Django, `data` (Kestra)
Avec le framework **pre-commit** :
```bash
pip install pre-commit
pre-commit install --hook-type commit-msg --hook-type pre-push
```
(voir `.pre-commit-config.yaml` d'exemple dans le README du dossier `governance/`)

## Windows
 
Ces hooks sont des scripts **bash**. Git for Windows fournit bash, donc ils
tournent nativement au commit/push. Aucune config supplémentaire.

## Limites (à connaître)

- **Local** → chaque dev doit l'activer une fois (`git config core.hooksPath`).
- **Contournable** → `git commit/push --no-verify` passe outre.
- Le **filet de sécurité** derrière : le workflow `pr-checks.yml` (dossier
  `governance/workflows/`) revérifie tout sur la PR → une PR non conforme
  devient rouge.
