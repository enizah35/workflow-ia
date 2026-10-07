# Mission : Refonder CROK sur de bonnes bases

- **Date :** 2026-10-07
- **Demande de Hugo :** « j'aimerais recommencer le projet CROK sur des bonnes bases avec mon workflow »
- **Projet / dépôt :** enizah35/CROK (dépôt vide au démarrage)

## Cadrage
- Objectif en une phrase : poser les fondations du MVP CROK (Expo + Supabase + TypeScript) pour que les agents de dev puissent ensuite livrer les fonctionnalités une par une.
- Livrables : plan d'architecture validé par Hugo, puis squelette du dépôt.
- Critères de réussite : plan conforme aux skills crok-* (schéma, RF-xx, sécurité, tests), découpé en tâches assignables à des agents.
- Défauts choisis : le dépôt étant vide, on part de zéro ; les skills crok-* sont la spec de référence.

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| 1 | Plan d'architecture du MVP | architecte | existante | terminé |
| 2 | Validation du plan par Hugo | — | — | en attente |

## Journal
- Lancement de l'agent architecte avec les 6 skills crok-* en contexte.
- Plan rendu : 21 tâches en 4 phases (socle, parcours roi, favoris/recherche, social/RGPD/notifications), 9 questions ouvertes avec défaut. Copie : /mnt/project-files/crok/plan-architecture-mvp.md
- Contrôle du Chef : conforme aux skills ; incohérences des skills relevées (disliked_ingredients int[] vs uuid, crok-conventions inexistant).

## Résultat livré

## Leçons
- La fiche architecte limite à 3-8 tâches : trop peu pour un MVP complet si une tâche = une PR. Autoriser des phases de 3-8 tâches.

## Compteurs
- Agents lancés : 1
- Relances : 0
