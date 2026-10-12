# Catalogue des agents

Le Chef consulte ce tableau à l'étape « Recruter ». Une fiche promue vit dans `.claude/agents/`
(agent natif de Claude Code) ; les autres vivent ici. Mettre ce tableau à jour à chaque fiche
créée, promue ou supprimée.

## Déroulé type d'un nouveau projet

| Étape | Agent | Livrable |
|-------|-------|----------|
| 1. Vision | `product-vision` (en parallèle de l'étape 2) | vision.md + questions au fondateur |
| 2. Marché et légal | `analyste-marche-legal` (ou `chercheur` pour une question précise) | recherche-marche-et-legal.md |
| 3. Plan | `architecte` (promu) | plan-technique.md : stack, contrat figé, phases, propriété des fichiers |
| 4. Phase 0 | `dev-fondations-depot` puis `dev-feature` en parallèle | une branche par tâche |
| 5. Intégration | `integrateur` (promu) | `agent/phaseN-integration` → une seule PR |
| 6. Vérification | `testeur-qa` (si parcours réel ou visuel) ; `relecteur` (si enjeu fort) | verdict + guide de test pour Hugo |

## Généralistes

| Agent | Où | Rôle | Missions réussies | Modèle |
|-------|----|------|-------------------|--------|
| architecte | `.claude/agents/` | Plan technique, contrat figé, phases | 5 | opus |
| integrateur | `.claude/agents/` | Fusion des branches parallèles, une PR par phase | 3 | sonnet |
| dev-feature | `agents/` | Une tâche du plan, tests compris, dans son worktree | 4 | sonnet |
| dev-fondations-depot | `agents/` | Squelette, outillage, CI gratuite, CLAUDE.md du dépôt | nouvelle (tiré de 4 phases 0) | sonnet |
| product-vision | `agents/` | Vision produit, MVP, questions au fondateur | 2 | opus |
| chercheur | `agents/` | Question précise, sources datées | 2 | sonnet |
| analyste-marche-legal | `agents/` | Concurrents, angle libre, CGU, licences, RGPD | nouvelle (tiré de 3 études) | sonnet |
| testeur-qa | `agents/` | Parcours réels, avant/après, captures, guide de test | nouvelle | sonnet |
| relecteur | `agents/` | Revue critique avant livraison | jamais lancé | sonnet |

## Spécialistes

| Agent | Variante de | Domaine | Missions | Modèle |
|-------|-------------|---------|----------|--------|
| analyste-marche-roblox | analyste-marche-legal | Roblox : économie, genres, tendances | 1 (2 instances) | sonnet |
| verificateur-conformite-roblox | analyste-marche-legal | Roblox : maturité, âge, monétisation | 1 | sonnet |
| evaluateur-concepts-jeu | — | Générer et noter des concepts de jeu | 1 | opus |
| architecte-luau-roblox | architecte | Plan technique d'un jeu Roblox | 1 | opus |
| dev-luau-fondations | dev-fondations-depot | Dépôt Roblox outillé (Rojo, Lune, CI) | 1 | sonnet |
| dev-luau-prototype | dev-feature | Prototype jouable d'une boucle Roblox | 1 | sonnet |
| game-designer-systemes | — | Système de jeu chiffré (sortie, spawns, progression) | nouvelle (tiré de Cap Libre) | opus |
| dev-godot-gdscript | dev-feature | Mécanique Godot sur copie du dossier de Hugo | nouvelle (tiré de Cap Libre) | sonnet |
| chercheur-references-web | chercheur | Références de sites haut de gamme et techniques d'animation | 1 | sonnet |
| directeur-artistique-web | — | DA + motion design + section prototype | 1 | opus |

Modèle : valeur à passer au paramètre `model` de l'outil Agent (voir « Économie » dans CLAUDE.md).
Opus seulement quand la qualité de conception décide de tout le reste.

## Promotion

Une fiche réussie sur 2 missions ou plus peut passer dans `.claude/agents/` (format natif :
`name`, `description` dans l'en-tête). Candidats suivants : `dev-feature` (4), `product-vision` (2),
`chercheur` (2).
