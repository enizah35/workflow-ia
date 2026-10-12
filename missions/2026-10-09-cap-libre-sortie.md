# Mission : Cap Libre, revue et sortie façon ARC Raiders (journal rétroactif)

- **Date :** 2026-10-09 → 2026-10-12 (journal écrit le 2026-10-12)
- **Demande de Hugo :** « regarde ce que je pourrais ajouter / changer / supprimer au jeu (C:\Bordel\Projet\sea-of-thieves-remake) pour faire un mélange fun et streamable entre Sea of Thieves et ARC Raiders », puis « une sortie façon ARC Raiders, des spawns aléatoires, sans chargement entre l'île et le jeu ».
- **Projet / dépôt :** jeu Godot 4.7.2 dans un dossier du PC de Hugo (pas de dépôt GitHub dans le projet)

## Cadrage
- Livrables : propositions priorisées, conception sortie + spawns, puis implémentations successives.
- Défauts : rien n'est commité chez Hugo ; sauvegarde avant chaque écrasement.

## Découpage
| # | Tâche | Agent | Fiche | Branche | Statut |
|---|-------|-------|-------|---------|--------|
| 1 | Revue, 19 propositions | Chef | — | — | fait |
| 2 | Conception sortie + spawns | Chef | — | — | fait |
| 3 | Directeur de spawns, renforts, bruit | Chef | — | dossier de Hugo | fait |
| 4 | Sortie (mur de brume, marée 30 min, passes, fusée) | Chef | — | dossier de Hugo | fait |
| 5 | Coffre balise, relève | Chef | — | dossier de Hugo | fait |
| 6 | Corrections touche R et vue du mât | Chef | — | /mnt/project-files/cap-libre/corrections/ | en attente du PC de Hugo |

## Journal
- Tout a été fait par le Chef seul (tâches séquentielles dans un même jeu) : 0 agent.
- Méthode qui marche : archive du dossier sur le PC → copie dans le conteneur → Godot 4.7.2 Linux
  téléchargé dans le conteneur → tests avant/après → serveur réel 25 s → copie retour avec
  sauvegarde dans `.artifacts/sauvegarde-avant-<sujet>/`.
- 66 tests comparés avant/après sans écart ; 3 échecs préexistants (depth, outing, treasure) ;
  18 tests graphiques impossibles en headless.
- Rendus hors écran du mur de brume via xvfb-run + gl_compatibility.

## Résultat livré
- Livrables : /mnt/project-files/cap-libre/ (propositions, conception, diffs, captures) ; code dans le dossier de Hugo, non commité.
- Décisions attendues de Hugo : commiter ses 421 fichiers en attente, tester à 4, chat vocal de proximité (restant).

## Leçons
- Fiches créées le 2026-10-12 à partir de ce travail : `game-designer-systemes`, `dev-godot-gdscript`.
- Règle « Dossier local de Hugo » ajoutée à CLAUDE.md.
- Fins de ligne mélangées dans certains fichiers : restaurer ligne par ligne.
- Le PC de Hugo peut être hors ligne : préparer les fichiers dans /mnt/project-files avec un diff.

## Compteurs
- Agents lancés : 0
- Relances : 0
