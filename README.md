# workflow-ia

Un « Chef » IA dans Claude Code : tu lui donnes une mission, il crée des agents spécialisés,
les lance, contrôle leur travail et améliore sa bibliothèque d'agents au fil des missions.

## Utilisation

1. Ouvre Claude Code à la racine de ce dossier (le Chef lit `CLAUDE.md` automatiquement).
2. Lance une mission :
   ```
   /mission Ajouter un écran "Mes favoris" dans CROK (~/dev/CROK)
   ```
3. Le Chef cadre, découpe, recrute ou crée les agents, les lance, puis te livre le résultat
   et un journal dans `missions/`.

Pour travailler sur un dépôt existant, donne son chemin dans la mission.
Pour un projet from scratch, décris l'idée : le Chef commence par l'agent `architecte`.

## Comment les agents sont créés

- `agents/README.md` est le catalogue : déroulé type d'un projet et liste des agents.
- `agents/` contient les fiches (rôle, méthode, interdits, pièges connus, format de sortie).
- Le Chef réutilise une fiche existante, en fait une variante, ou en crée une nouvelle
  à partir de `templates/agent-template.md`.
- Une fiche qui a fait ses preuves sur 2 missions peut être promue dans `.claude/agents/`
  (c'est le cas d'`architecte` et d'`integrateur`).

## Agents en cascade

Dans Claude Code, un agent lancé par le Chef ne peut pas lancer lui-même d'autres agents.
Il demande donc des sous-agents via un bloc `BESOIN_SOUS_AGENTS`, et c'est le Chef qui les
crée et les lance (profondeur max 3). La vraie récursion arrivera à l'étape 3 (Agent SDK).

## Garde-fous

Toute action irréversible ou à impact extérieur (argent réel, trading, déploiement prod,
suppression, push sur main) attend ta validation.

## Feuille de route

1. Le Chef dans Claude Code ✓
2. Test sur de vraies missions ← ici (CROK v2, Genesis, Guard the Museum, Cap Libre, Platine, CircularRouter)
3. Passage à l'API (Claude Agent SDK) : boucle autonome, vraie récursion, suivi du budget
4. Trading : simulation de l'expérience des 100 $, puis réel avec plafond et arrêt d'urgence
