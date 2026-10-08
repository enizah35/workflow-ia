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
| 1 | P0-1 + P0-2 squelette, CI, schéma, migrations | dev-fondations-python | nouvelle (prompt) | terminé (35 tests) |
| 2 | P0-3 Finance | dev-finance-ledger | nouvelle (prompt) | terminé (+40 tests) |
| 3 | P0-4 Passerelle LLM | dev-llm-gateway | nouvelle (prompt) | terminé (+53 tests) |
| 4 | P0-5 Events + API | dev-fastapi-events | nouvelle (prompt) | terminé (+22 tests) |
| 5 | Intégration | integrateur | existante | terminé (153 tests) |

## Journal
- P0-1 et P0-2 confiés au même agent (séquentiels) : 1 correction SQL (IF NOT EXISTS sur schema_migrations).
- Brief commun en fichier (contrat + pièges GN002 au COMMIT, vues NULL) partagé par les 3 agents parallèles : aucun conflit au merge, propriété des fichiers respectée (vérifiée par diff).
- P0-4 a vérifié IDs/prix modèles via le skill claude-api : conformes.
- Intégrateur : 3 merges sans conflit, coutures finance→events, router↔finance (SettlementExceedsHold réglé au montant du hold + incident), CLI api, test de câblage réel. Vérifié par le Chef : 153 passed, ruff propre.
- Points ouverts pour la phase 1 : type d'event « incident » dédié, écart non comptabilisé en cas d'incident, spend_refused émis 2 fois, atomicité ledger/events non câblée par défaut.

## Résultat livré
- Branche enizah35/genesis `agent/phase0-integration` poussée ; PR en attente de la création de `main` (accord de Hugo).

## Leçons
- Un brief commun en fichier + liste des pièges remontés par l'agent fondations = vague parallèle sans conflit.
- Garder les branches de tâche en local : une seule branche visible sur GitHub.
- Dépôt vide : demander la création de `main` dès le cadrage, pas à la fin.

## Compteurs
- Agents lancés : 5
- Relances : 0
