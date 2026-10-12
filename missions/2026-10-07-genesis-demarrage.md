# Mission : Genesis, démarrage du projet

- **Date :** 2026-10-07
- **Demande de Hugo :** « j'aimerais commencer à mettre en place mon nouveau projet : genesis » + spec PDF (Genesis_Architecture_Specification.pdf, 21 pages).
- **Projet / dépôt :** Genesis, nouveau (dépôt enizah35/genesis à créer par Hugo)

## Cadrage
- Objectif en une phrase : transformer la spec Genesis en plan d'exécution validé (stack, structure, tâches de la phase 0 / sprint 1) puis lancer le dev.
- Livrables : plan technique `/mnt/project-files/genesis/plan-technique.md` (copie dans `docs/genesis/`), puis une PR par phase sur enizah35/genesis.
- Critères de réussite : plan fidèle à la spec (§22 structure, §23 roadmap, §25 sprint 1), tâches parallélisables avec propriétaire de chaque fichier partagé, contrat (schéma SQL + interfaces Python) figé avant la vague de dev.
- Défauts choisis (questions non posées) :
  - Stack de la spec conservée : Python 3.12 + FastAPI + PostgreSQL (Docker Compose en local, pas Supabase au départ).
  - Premier fournisseur LLM : Anthropic (Claude), derrière une interface multi-fournisseurs ; 2 tiers (economy/standard) pour la phase 0.
  - Capital 100 EUR simulé (V1 de la spec), aucune dépense réelle hors coûts API.
  - Tests LLM avec un fournisseur simulé (fake) pour que la CI ne coûte rien.

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| 1 | Plan technique phase 0 + sprint 1 | architecte | existante | terminé |

## Journal
- Spec lue par le Chef (texte extrait dans /mnt/project-files/genesis/spec-genesis.txt).
- Dépôt genesis inexistant : demandé à Hugo ; créé par Hugo (enizah35/Genesis, vide), connecté au projet.
- Architecte lancé sur la spec. Plan rendu (418 lignes) : stack confirmée (uv, psycopg 3 sans ORM, migrations SQL brut), ledger en ajout seul avec hold/settle et triggers de conservation, contrat Python figé, 5 tâches phase 0 + 5 tâches phase 1, propriété des fichiers attribuée.
- Contrôle du Chef : fidèle à §22/§23/§25 ; arbitrage accepté (lignes skill du §25.4 reportées au sprint 2). Effectif 10 dev + 2 intégrateurs au-dessus de la cible 1-5 : justifié par 2 phases (chacune = 5-6 agents) ; P0-1 et P0-2 peuvent être confiés au même agent (séquentiels). Prix des modèles marqués à revérifier avant le mode réel.

## Résultat livré
- docs/genesis/plan-technique.md (copie de /mnt/project-files/genesis/plan-technique.md), en attente de validation de Hugo (4 questions).

## Leçons

## Compteurs
- Agents lancés : 1
- Relances : 0
