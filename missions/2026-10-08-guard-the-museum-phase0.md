# Mission : Guard the Museum, phase 0 (fondations et prototype)

- **Date :** 2026-10-08
- **Demande de Hugo :** « je veux un périmètre assez moyen (plus grand que ce que tu recommande), je suis disponible à plein temps pour l'instant, et le repo est créé »
- **Projet / dépôt :** enizah35/guard-the-museum (créé vide, public)

## Cadrage
- Objectif : dépôt outillé + prototype jouable à 2 pour le point de décision « est-ce amusant ? ».
- Livrables : squelette (P0.1), lanceur de tests Lune (P0.2), guide Hugo Studio MCP + greybox (P0.3/P0.4), prototype (P0.5). Une seule PR de phase.
- Périmètre retenu : /mnt/project-files/roblox/perimetre-guard-the-museum.md (2 ailes, 45 anomalies, 4 rôles, ~8 semaines).
- Défauts : main créé avec un README seulement si Hugo valide (carte posée).

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| P0.1+P0.2 | Squelette, CI, tests Lune, CLAUDE.md | dev-luau-fondations | nouvelle | terminé |
| P0.3+P0.4 | Studio MCP + greybox aile 1 | Hugo (guide du Chef) | — | guide écrit |
| P0.5 | Prototype boucle | dev-luau-prototype | nouvelle | en cours |

## Journal
- Contrainte découverte : le proxy du conteneur bloque les releases GitHub (403) ; crates.io/npm OK.

- P0.1+P0.2 rendus (branche locale agent/p0-fondations, 2 commits). Contrôle du Chef : stylua OK, 16 tests Lune OK, rojo build OK. luau-lsp et selene std roblox n'ont pas tourné (proxy) → à voir en CI.
- P0.5 lancé sur agent/p0-prototype (depuis agent/p0-fondations), avec carte de secours générée pour tester sans la greybox.

## Résultat livré

## Leçons

## Compteurs
- Agents lancés : 2
- Relances : 0
