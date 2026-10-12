# Mission : mettre à jour et compléter les agents à partir des projets faits

- **Date :** 2026-10-12
- **Demande de Hugo :** « j'aimerais qu'à partir des projets que j'ai déjà faits, tu mettes à jour / ajoutes des agents »
- **Projet / dépôt :** enizah35/workflow-ia

## Cadrage
- Objectif en une phrase : capitaliser les leçons de CROK v2, Genesis, Roblox, Cap Libre, Platine et CircularRouter dans les fiches, le catalogue et le CLAUDE.md du Chef.
- Livrables : une PR sur workflow-ia ; copie à jour dans /mnt/project-files/workflow-ia/.
- Défauts choisis :
  - Les 4 branches de mission jamais fusionnées (Roblox, CircularRouter, Genesis, Platine) sont réunies dans cette PR, pour que leurs fiches et journaux ne se perdent pas.
  - `architecte` et `integrateur` promus dans `.claude/agents/` (règle des 2 missions).

## Découpage
| # | Tâche | Agent | Fiche | Branche | Statut |
|---|-------|-------|-------|---------|--------|
| 1 | Relire journaux, mémoire, fils et livrables | Chef | — | — | fait |
| 2 | Améliorer les fiches existantes | Chef | — | claude/project-thread-yhvvqx | fait |
| 3 | Créer les fiches manquantes | Chef | — | idem | fait |
| 4 | CLAUDE.md, modèles, catalogue | Chef | — | idem | fait |

## Journal
- Tâche de rédaction courte et liée : faite par le Chef sans agent.
- Nouvelles fiches : analyste-marche-legal, dev-fondations-depot, testeur-qa, game-designer-systemes, dev-godot-gdscript.
- Journal rétroactif ajouté pour Cap Libre (aucun n'existait).

## Résultat livré
- PR à merger : celle de la branche claude/project-thread-yhvvqx.

## Leçons
- Des fiches créées en mission sont restées sur des branches jamais fusionnées, et d'autres n'existaient que dans les prompts (Genesis, Platine) : règle ajoutée (« une fiche écrite seulement dans un prompt est perdue »).
- Le travail fait par le Chef seul (Cap Libre) n'avait pas de journal : en ouvrir un même sans agent.

## Compteurs
- Agents lancés : 0
- Relances : 0
