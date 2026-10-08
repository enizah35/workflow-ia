# Mission : Genesis phase 0 (Foundation)

- **Date :** 2026-10-08
- **Demande de Hugo :** « ok » au plan technique (choix par défaut des 4 questions retenus).
- **Projet / dépôt :** enizah35/genesis

## Cadrage
- Objectif : livrer la phase 0 du plan (`docs/genesis/plan-technique.md` §4) en une seule PR.
- Livrables : squelette + CI, schéma SQL, service Finance, passerelle LLM, event log + API lecture.
- Critères : ceux de P0-1..P0-5 ; CI verte ; une seule PR `agent/phase0-integration` → `main`.
- Défauts : clé Anthropic + plafonds 1 €/run, 5 € total (pas utilisés en phase 0, CI sans clé) ; Tavily ; code EN / docs FR ; lignes skill §25.4 au sprint 2.
- Environnement conteneur : pas de Docker ; Postgres 16 local, une base par agent (genesis_p0a, genesis_p03, genesis_p04, genesis_p05, genesis_int).
- Dépôt vide : création de `main` (README seul) soumise à l'accord de Hugo (carte de décision).
- Branches de tâche gardées en local (non poussées) pour qu'il n'y ait qu'une PR visible.

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| 1 | P0-1 + P0-2 squelette, CI, schéma, migrations | dev-fondations-python | nouvelle (prompt) | en cours |
| 2 | P0-3 Finance | dev-finance-ledger | à créer | à faire |
| 3 | P0-4 Passerelle LLM | dev-llm-gateway | à créer | à faire |
| 4 | P0-5 Events + API | dev-fastapi-events | à créer | à faire |
| 5 | Intégration | integrateur | existante | à faire |

## Journal
- P0-1 et P0-2 confiés au même agent (séquentiels).

## Résultat livré

## Leçons

## Compteurs
- Agents lancés : 1
- Relances : 0
