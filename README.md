# POPolitics — Infrastructure

Point d'entrée du projet. Ce repo regroupe tous les services via git submodules.

---

## Repos

| Repo | Rôle |
|---|---|
| [api](https://github.com/Popolitics/api) | Business logic, agrégation des services internes |
| [auth-service](https://github.com/Popolitics/auth-service) | JWT, gestion des utilisateurs et permissions |
| [data-service](https://github.com/Popolitics/data-service) | Exposition des datamarts AN, Sénat, Parlement européen |
| [ia-service](https://github.com/Popolitics/ia-service) | Résumés automatiques, classification et analyse politiques |
| [frontend](https://github.com/Popolitics/frontend) | Interface web Next.js (SSR/BFF) |
| [data](https://github.com/Popolitics/data) | Pipelines ETL, ingestion et transformation via Kestra |
| [documentation](https://github.com/Popolitics/documentation) | Documentation projet |

---

## Cloner le projet

```bash
git clone --recurse-submodules https://github.com/Popolitics/infra.git
```

---

## Mettre à jour les submodules

```bash
git submodule update --remote --merge
```

---

## Architecture

Voir [documentation/02-technologie/18-architecture.md](https://github.com/Popolitics/documentation/blob/main/02-technologie/18-architecture.md)
