# Mission : trouver 1 ou 2 jeux Roblox à fort potentiel de rentabilité

- **Date :** 2026-10-08
- **Demande de Hugo :** « j'aimerais que le workflow d'ia cherche un ou deux jeux roblox à créer avec un gros potentiel de rentabilité »
- **Projet / dépôt :** nouveau (recherche, pas de code)

## Cadrage
- Objectif en une phrase : recommander 1 ou 2 concepts de jeux Roblox à créer, justifiés par le marché actuel.
- Livrables : rapport `/mnt/project-files/roblox/rapport-jeux-roblox.md` ; ce journal.
- Critères de réussite : chiffres sourcés et datés ; concepts faisables par une petite équipe aidée par l'IA ; plan MVP + KPI de go/no-go.
- Défauts choisis (questions non posées) : équipe = Hugo seul + agents IA, petit budget pub ; publics FR + international ; pas de monétisation prédatrice.

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| 1 | Économie créateurs Roblox (revenus, DevEx, découvrabilité, coûts) | analyste-marche-roblox | nouvelle | terminé |
| 2 | Tendances genres et top jeux 2025-2026, niches sous-servies | analyste-marche-roblox | nouvelle (2e instance) | terminé |
| 3 | Générer et noter les concepts, retenir 2 | evaluateur-concepts-jeu | nouvelle | terminé |

## Journal
- 2026-10-08 : agents 1 et 2 lancés en parallèle.
- Agent 1 rendu : bon niveau de sourcing (8-K Q2 2026, Creator Hub). Points clés : algo cible la rétention J28 depuis juin 2026, bookings Q3 2026 en baisse prévue, accès <16 ans conditionné, méga-succès = prototypes rapides portés par studios.

- Agent 2 rendu : top 20 daté, hits viraux qui s'effondrent en 2-3 mois, genres durables (survival coop, pêche, horreur asymétrique, anime), niches (adultes 18+, Kids/Select, coop 4 joueurs, 2D). Synthèses dans /mnt/project-files/roblox/sources/.
- Agent 3 (évaluateur) lancé avec les deux synthèses.

- Agent 3 rendu : 9 concepts notés /35, 2 retenus. Contrôle du Chef : calculs de revenus cohérents avec l'étude 1 (0,14 $/DAU/jour de bookings plateforme) ; le Chef inverse l'ordre proposé (pari principal d'abord) car la demande du concept n°2 n'est pas prouvée.
- BESOIN_SOUS_AGENTS reçu (architecte-luau-roblox, verificateur-conformite-roblox) : reporté, en attente du go de Hugo (règle budget).

- 13:11 : go de Hugo (sans choix d'ordre) → défaut option A (Guard the Museum d'abord). Agents 4 (architecte-luau-roblox) et 5 (verificateur-conformite-roblox) lancés en parallèle, fiches nouvelles.

## Résultat livré
- Rapport : /mnt/project-files/roblox/rapport-jeux-roblox.md (copie dans docs/roblox/).
- Recommandation : « Guard the Museum » (coop 4 anomalies, pari principal) ; « Catch a Critter » (capture cozy Kids, projet d'apprentissage).

## Leçons
- Fiches : analyste-marche-roblox a bien marché en 2 instances parallèles sur des questions disjointes ; evaluateur-concepts-jeu utile, garder l'étape de vérification des concurrents (elle a fait baisser 2 notes).
- Le Chef a condensé chaque rapport dans un fichier sources/ avant l'évaluateur : contexte minimal et traçable, à refaire.
- Les sites de CCU divergent : toujours demander 2 sources et la date.

## Compteurs
- Agents lancés : 5
- Relances : 0
