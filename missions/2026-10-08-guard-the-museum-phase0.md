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
| P0.5 | Prototype boucle | dev-luau-prototype | nouvelle | terminé (non testé en Studio) |

## Journal
- Contrainte découverte : le proxy du conteneur bloque les releases GitHub (403) ; crates.io/npm OK.

- P0.1+P0.2 rendus (branche locale agent/p0-fondations, 2 commits). Contrôle du Chef : stylua OK, 16 tests Lune OK, rojo build OK. luau-lsp et selene std roblox n'ont pas tourné (proxy) → à voir en CI.
- P0.5 lancé sur agent/p0-prototype (depuis agent/p0-fondations), avec carte de secours générée pour tester sans la greybox.

- P0.5 rendu (55 tests). main créé (README, validé par Hugo via carte). PR enizah35/guard-the-museum#1 (agent/phase0-integration → main).
- CI : 4 tours pour passer au vert : StyLua sans feature luau (largeur de ligne), 23 avertissements selene UDim2, 2 erreurs luau-lsp (singleton élargi par générique, comparaison RemoteEvent/nil). Vert le 2026-10-08 17:32.
- P0.3 : Studio de Hugo piloté depuis le cloud via robloxstudio-mcp (remote-devices) ; playtest lancé/arrêté. .rbxl du prototype déposé dans C:\Users\hugoe\source\repos.

## Résultat livré

## Leçons
- Brief des agents Luau : installer stylua avec --features luau ; selene roblox et luau-lsp ne tournent qu'en CI → prévoir 1-3 tours de CI.
- Le MCP Studio de Hugo est joignable directement depuis le thread (outils remote-devices robloxstudio-mcp) : tests de jeu possibles sans Remote Control.

## Compteurs
- Agents lancés : 2
- Relances : 0
