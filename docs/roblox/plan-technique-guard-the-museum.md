# Plan technique — MVP « Guard the Museum »

*Agent : architecte-luau-roblox — 8 octobre 2026. Ce document est un plan, sans code complet. Il s'appuie sur la fiche « Concept n°1 » de `rapport-jeux-roblox.md`.*

---

## 0. En bref

- **Langage : Luau natif en mode `--!strict`**, pas roblox-ts (voir §1.2).
- **Outillage hors Studio :** Rokit, Rojo 7.7, Wally, luau-lsp, selene, StyLua et Lune pour les tests et les scripts. Le **serveur MCP officiel de Roblox Studio** permet aux agents de lancer des tests de jeu sur le PC de Hugo.
- **2 places dans une même expérience :** un *Lobby* (équipes, boutique, codex) et une *Nuit* (un serveur réservé par équipe, via TeleportService).
- **Le serveur décide de tout.** Le client ne fait qu'afficher et envoyer des intentions. Toute la logique de jeu vit dans des modules « purs », testables avec Lune sans Studio.
- **Anomalies décrites par des données :** `règle × objet × salle`. Les objets du décor sont repérés par des tags CollectionService. Ajouter une anomalie revient à ajouter une ligne de données et éventuellement un tag dans la carte, sans nouveau code.
- **Délai révisé : 9 à 10 semaines à plein temps** pour le périmètre complet, ou **6 semaines avec un périmètre réduit** (§6).

---

## 1. Stack et justification

### 1.1 État de l'outillage en octobre 2026 (vérifié)

| Outil | État | Usage ici |
|---|---|---|
| **Rojo** | 7.7.0 (docs.rs, SourceForge) | Synchronise les fichiers du disque vers Studio. Le code vit dans git. |
| **Script Sync (Studio, natif)** | Disponible. Ne synchronise que les scripts et les dossiers. Roblox renvoie lui-même vers Rojo quand le disque doit faire foi. | Non retenu. Rojo est plus complet. |
| **Studio MCP (officiel)** | `execute_luau`, `start_stop_play`, `get_console_output`, `insert_asset`, capture d'écran, simulation d'entrées. Connexion rapide « Claude Code » proposée dans Studio. | Les agents lancent des tests de jeu et lisent la sortie **sur le PC Windows de Hugo**. |
| **Open Cloud – Luau Execution** | Exécute du Luau sans interface dans une version de place. Tâches de 5 min maximum, 5 créations par minute, pas de physique, les scripts de la place ne démarrent pas, accès possible aux vrais DataStores. | Tests d'intégration en CI, en option (phase 4), sur une **expérience de test séparée**. |
| **roblox-ts** | Compilateur 3.0.0, publié le 12/09/2024 (aucune version depuis 2 ans). `@rbxts/types` est toujours mis à jour (30/09/2026). | Écarté (§1.2). |
| **Lune** | Exécute du Luau hors de Roblox et sait lire et écrire des fichiers `.rbxl`/`.rbxm` (bibliothèque `roblox`). | Tests unitaires, validation du contenu et de la carte, scripts. |
| **Wally** | Reste le gestionnaire de paquets de référence (pesde existe en alternative). | Paquets : React-lua, Promise si besoin. |
| **ProfileStore** (loleris) | Successeur de ProfileService : verrouillage de session et sauvegarde automatique. | Couche de sauvegarde. Le module est copié dans le dépôt (vendored). |
| **CaptureService** | `StartVideoCaptureAsync` (30 s maximum, voix coupées), `PromptShareCapture`, `PromptSaveCapturesToGallery`. | Clip partageable de fin de nuit (§4, tâche P3.3). |
| **SocialService** | `PromptGameInvite`, `CanSendGameInviteAsync`, `GetPartyAsync`. | Invitations et parties d'amis. |
| **AnalyticsService** | `LogOnboardingFunnelStepEvent`, `LogFunnelStepEvent`, événements personnalisés et économiques. Envoyés **uniquement par le serveur, en jeu publié**. 10 funnels au maximum dans le tableau de bord. | Funnel du premier run et rétention. |

### 1.2 Luau natif ou roblox-ts : choix de **Luau natif strict**

| Critère | Luau `--!strict` | roblox-ts |
|---|---|---|
| Courbe pour Hugo (fort en TypeScript) | Environ 1 semaine. La syntaxe des types ressemble à TypeScript. | Immédiate pour le langage. Il reste à apprendre l'API Roblox, qui est la vraie difficulté dans les deux cas. |
| **Agents IA** | Bien plus de corpus d'entraînement. La documentation, les exemples, le DevForum, l'Assistant Studio et le MCP sont tous en Luau. | Corpus réduit. Les agents mélangent les idiomes TypeScript et Roblox (`undefined` contre `nil`, indexation à partir de 0 ou de 1, macros). |
| Débogage | Les erreurs de Studio et du MCP pointent directement vers le fichier source. | Les erreurs pointent vers le Luau compilé, et il faut remonter vers le TypeScript. |
| Pérennité | Langage officiel, avec un nouveau solveur de types. | Compilateur sans version depuis septembre 2024. Projet communautaire. |
| Assets du Creator Store | Lisibles et modifiables directement. | Il faut écrire des déclarations de types. |
| Étape de build | Aucune (Rojo synchronise directement). | `rbxtsc` en mode watch, puis Rojo : un maillon de plus. |

**Verdict.** Luau natif. Le confort de TypeScript ne compense pas trois choses : un compilateur qui ne publie plus de version, des agents moins fiables sur ce langage, et une boucle de débogage plus longue. Les réflexes de Hugo se transposent directement en Luau : typage strict, modules, injection de dépendances, tests. Une règle selene, appliquée en CI, impose `--!strict` en tête de chaque fichier.

### 1.3 Boîte à outils retenue

- **Rokit** (`rokit.toml`) fixe les versions de rojo, wally, selene, stylua, lune et luau-lsp.
- **VS Code** avec luau-lsp. Les définitions de l'API Roblox et le sourcemap Rojo (`rojo sourcemap --watch`) servent à l'autocomplétion des chemins d'instances.
- **UI : React-lua** (paquets Wally `jsdotlua/react` et `react-roblox`). C'est le modèle mental déjà connu de Hugo et des agents. Il couvre le HUD, la tablette caméras, le codex, la boutique et le récapitulatif.
- **Réseau :** un module `Net` maison, petit et typé, construit à partir d'un **fichier de contrat unique** (§3). Pas de framework (ni Knit ni autre) : moins de magie, donc moins d'erreurs des agents.
- **CI GitHub Actions :** `stylua --check`, `selene`, `luau-lsp analyze` (strict), tests Lune et validateurs de contenu. Elle tourne sur Linux et ne demande aucun secret jusqu'à la phase 4.
- **Où travaille chaque agent :**
  - **Les agents dans le cloud** (sans Studio) s'occupent de la logique pure, des données, des tests et de l'UI vérifiée par les types.
  - **Les agents sur le PC de Hugo** (Studio et MCP) s'occupent de l'intégration et des tests de jeu.
  - Hugo garde **le level design dans Studio**.

---

## 2. Arborescence du dépôt

```
guard-the-museum/
├── rokit.toml  wally.toml  selene.toml  stylua.toml  .luaurc
├── lobby.project.json          # Rojo → place Lobby
├── night.project.json          # Rojo → place Nuit
├── CLAUDE.md                   # conventions pour les agents (strict, pureté, contrat Net)
├── src/
│   ├── shared/                 # synchronisé dans ReplicatedStorage des 2 places
│   │   ├── Net/                # Contract.luau (source unique) + Net.luau (wrapper typé)
│   │   ├── Types.luau          # types partagés (RoleId, AnomalyKey, NightSnapshot…)
│   │   ├── Content/            # DONNÉES uniquement (aucune logique)
│   │   │   ├── Rules.luau      # règles : Moved, Missing, Rotated, Recolored, Duplicated,
│   │   │   │                   #   LightOff, Facing, PaintingSwap, Intruder, Sound…
│   │   │   ├── Objects.luau    # catégories d'objets + règles compatibles
│   │   │   ├── Wings.luau      # ailes → salles → caméras
│   │   │   ├── Codex.luau      # entrées du codex (règle×catégorie, variantes rares)
│   │   │   ├── Modifiers.luau  # modificateurs de nuit (panne, brouillard, samedi…)
│   │   │   ├── Progression.luau# grades, XP, contrats de 5 nuits
│   │   │   └── Products.luau   # IDs game passes / dev products / cosmétiques
│   │   ├── Domain/             # LOGIQUE PURE, sans API Roblox → testée sous Lune
│   │   │   ├── Rng.luau  AnomalyPicker.luau  NightDirector.luau  Threat.luau
│   │   │   ├── ReportJudge.luau  Scoring.luau  Progression.luau  Pods.luau
│   │   │   └── ProfileSchema.luau  (défaut + migrations versionnées)
│   │   └── UI/                 # composants React-lua partagés
│   ├── server/common/          # services serveur communs aux 2 places
│   │   ├── DataService.luau    # ProfileStore (vendored dans src/vendor/)
│   │   ├── PurchaseService.luau# ProcessReceipt idempotent + game passes
│   │   ├── PolicyGate.luau     # PolicyService
│   │   └── Analytics.luau      # wrapper AnalyticsService (no-op en Studio)
│   ├── lobby/server/  lobby/client/    # pods, party, téléport, boutique, codex
│   ├── night/server/           # NightService, AnomalyService, ThreatService,
│   │                           # RoleService, ReportService, EntityService,
│   │                           # ReplayRecorder, ReviveService
│   ├── night/client/           # contrôleurs : CameraTablet, Flashlight, RepairTool,
│   │                           # Catalog (conservateur), HUD, Tutorial, Recap
│   └── vendor/                 # ProfileStore (copié, version notée)
├── tests/                      # specs Lune : Domain/*, Content, Net contract
├── tools/                      # scripts Lune
│   ├── test.luau               # mini-runner + mocks (DataStore, Players, clock)
│   ├── validate-content.luau   # cohérence Rules×Objects×Wings, IDs uniques, ≥40 anomalies
│   └── validate-map.luau       # lit maps/*.rbxm : tags/attributs vs Content
├── maps/                       # exports .rbxm des ailes (git LFS), source = Studio
└── .github/workflows/ci.yml
```

**Conventions.**
- **Rojo ne gère que le code.** Les décors restent dans la place publiée, et Hugo les exporte en `.rbxm` à chaque modification pour que `validate-map` les vérifie.
- **Les objets qui peuvent porter une anomalie** ont le tag `Anomalable` et les attributs `ObjectId`, `Category` et `RoomId`. Les caméras ont le tag `SecurityCam` et l'attribut `RoomId`.
- **La Nuit se lance seule dans Studio.** Sans `TeleportData`, une configuration simulée est chargée (solo, aile 1). C'est indispensable, car les téléportations ne marchent pas dans Studio.
- **Travail parallèle.** Un worktree par agent. **Un seul `rojo serve` à la fois** vers Studio : l'intégration et les tests de jeu se font donc l'un après l'autre.

---

## 3. Contrat réseau

**Principes.**
- Toutes les remotes sont déclarées dans `shared/Net/Contract.luau` : nom, sens, type du contenu, limite de fréquence.
- Le serveur **valide chaque entrée** : type, valeurs énumérées connues, rôle, distance, cooldown, phase de la nuit.
- L'état continu (horloge, menace, phase) passe par des **attributs** sur un objet `NightState`, répliqués automatiquement. Les événements ponctuels passent par des RemoteEvents.

| Remote | Sens | Contenu | Validation et effet côté serveur |
|---|---|---|---|
| `Lobby.JoinPod` | C→S | `podId` | Pod non plein, non verrouillé (ou réservé aux amis) |
| `Lobby.LeavePod` | C→S | — | — |
| `Lobby.StartPod` | C→S | `{wingId, difficulty}` | Uniquement le chef du pod, aile débloquée pour le chef → `ReserveServer` + `TeleportAsync` groupé avec `TeleportData` |
| `Lobby.PodState` | S→C | `PodSnapshot` | Diffusé à chaque changement |
| `Night.SelectRole` | C→S | `RoleId` | Phase = briefing. Rôle libre, ou rôles fusionnés en solo et en duo |
| `Night.Ready` | C→S | — | La nuit démarre quand tout le monde est prêt, ou après 20 s |
| `Night.ViewCamera` | C→S | `cameraId` | Rôle caméras (ou fusionné), caméra active (non sabotée). Le serveur retient la caméra regardée |
| `Night.Report` | C→S | `{roomId, objectId, ruleId}` | Cooldown de 3 s. Le joueur doit être dans la salle **ou** regarder une caméra qui la couvre. `ReportJudge` décide : juste → menace en baisse et points ; faux → menace en hausse |
| `Night.ReportResult` | S→C | `{ok, anomalyKey?, threatDelta}` | Envoyé à toute l'équipe (retour visuel et sonore) |
| `Night.Repair` | C→S | `{deviceId, phase: "start" \| "cancel"}` | Rôle technicien, distance inférieure à 8 studs. Le serveur chronomètre la réparation |
| `Night.Ping` | C→S | `Vector3` | Limité à 1 par seconde. Marqueur pour l'équipe |
| `Night.Event` | S→C | `{kind, data}` | Panne, coupure caméra, scare, chasse en cours |
| `Night.RequestRevive` | C→S | — | Joueur à terre, revive pas encore utilisé cette nuit → `PromptProductPurchase` |
| `Night.End` | S→C | `NightSummary` | Résultat, statistiques, chronologie du récap, XP gagnée |
| `Meta.Equip` | C→S | `cosmeticId` | Cosmétique possédé (profil ou game pass) |
| `Meta.ProfileDelta` | S→C | sous-ensemble du profil | Codex, XP, grade, contrats |
| `Meta.TutorialStep` | C→S | `stepIndex` | Valeur croissante, une fois par étape → funnel d'onboarding |

**Ce que le client ne décide jamais :** quelle anomalie est active, la menace, les récompenses, la possession d'un objet, le succès d'une réparation.

Les anomalies sont **appliquées par le serveur** sur les instances du Workspace (déplacer, masquer, changer la couleur, échanger une texture), ce qui les réplique d'elles-mêmes. Les salles et caméras de la Nuit sont des modèles `Persistent` ou `Atomic` en StreamingEnabled, ou bien le streaming est désactivé si la carte reste petite. À trancher en phase 0 (§7).

---

## 4. Données sauvegardées (ProfileStore)

**Profil** (clé `Player_<UserId>`, `schemaVersion` avec migrations dans `Domain/ProfileSchema`) :

```
schemaVersion: number
xp: number, grade: number
contracts: { [wingId]: { nightsDone: 0..5, completedAt?: number } }
unlockedWings: { [wingId]: true }
codex: { [anomalyKey]: { seen: number, firstAt: number, variants: { [variantId]: true } } }
cosmetics: { owned: { [id]: true }, equipped: { uniform?: id, flashlight?: id, emote?: {id} } }
stats: { nights: number, wins: number, reportsOk: number, reportsBad: number, revives: number }
tutorialDone: boolean
settings: { sensitivity, scareIntensity: "normal"|"reduced" }
purchaseLog: { [purchaseId]: number }   -- idempotence ProcessReceipt (purgé > 200 entrées)
```

**Non sauvegardé :** l'état d'une nuit en cours (aucune reprise après déconnexion au MVP) et le revive « utilisé » (propre à la session du serveur de Nuit).

**Game passes :** jamais copiés dans le profil. Le serveur appelle `UserOwnsGamePassAsync` (avec un cache) à chaque connexion.

**Pas de MemoryStore au MVP.**
- Les pods sont propres à chaque serveur de lobby. La « file publique » consiste à rejoindre un pod ouvert du lobby où l'on se trouve.
- Le matchmaking entre serveurs (MemoryStore SortedMap) n'est ajouté que si les mesures montrent des pods qui restent vides (après le MVP).

---

## 5. Phases et tâches

**Légende :**
- **∥** : peut tourner en parallèle (un worktree par agent).
- **Studio** : nécessite le PC de Hugo et le MCP.
- **H** : Hugo lui-même.

### Phase 0 — Fondations et prototype (semaine 1) : *« une nuit est-elle amusante à 2 ? »*

| # | Tâche | Livrable | Terminé quand… | ∥ |
|---|---|---|---|---|
| P0.1 | Squelette du dépôt | rokit, wally, les 2 projets Rojo, selene, StyLua, `.luaurc` strict, CI, `CLAUDE.md` | La CI est verte sur un dépôt vide, et `rojo serve` ouvre les 2 places | — (premier) |
| P0.2 | Lanceur de tests Lune et mocks | `tools/test.luau`, simulations de DataStore, Players et horloge | 3 tests d'exemple passent en local et en CI | ∥ |
| P0.3 | Connexion Studio MCP | Claude Code relié au MCP de Studio sur le PC Windows | Un agent démarre un test de jeu et lit la console | H, Studio |
| P0.4 | Aile 1 en *greybox* | 6 à 8 salles, 4 à 6 caméras, environ 25 objets tagués (Creator Store) | `validate-map` passe | H, Studio, ∥ |
| P0.5 | Prototype jetable de la boucle | 2 règles (Moved, Missing), signalement, menace, fin de nuit | Test de jeu à 2 : Hugo juge le fun et note ce qu'il apprend | Studio |

**Point de décision de fin de semaine 1 :** si ce n'est pas amusant, on revoit la boucle avant d'aller plus loin.

### Phase 1 — Noyau de la Nuit (semaines 2 et 3)

| # | Tâche | Livrable | Terminé quand… | ∥ |
|---|---|---|---|---|
| P1.1 | Schéma de contenu et validateur | `Content/*` typés, `validate-content` | Échoue sur un ID en double ou une combinaison impossible. Annonce ≥ 40 anomalies | ∥ |
| P1.2 | `Domain` des anomalies | `Rng` (graine), `AnomalyPicker` (pondérations, pas de répétition, variantes rares 2 %), `NightDirector` (une anomalie toutes les 25 à 35 s, montée en difficulté, réglage pour 1 à 4 joueurs) | Tests Lune : déterministe avec une graine donnée, couvre tout le contenu | ∥ |
| P1.3 | `Domain` de la menace et du jugement | `Threat` (anomalie manquée, faux signalement, déclin), `ReportJudge`, `Scoring` | Table de cas testée, y compris les égalités et les signalements simultanés | ∥ |
| P1.4 | Net et contrat | `Contract.luau`, `Net.luau`, limites de fréquence, validateurs de types | Test Lune : chaque remote a un validateur, et un contenu invalide est rejeté | ∥ |
| P1.5 | Services serveur de la Nuit | NightService (phases), AnomalyService (applicateurs par règle), ThreatService, ReportService | Nuit complète de 8 min dans Studio en solo, sans erreur dans la console | Studio |
| P1.6 | Rôles et outils | Tablette caméras, lampe et signalement en salle, réparation des appareils sabotés, catalogue du conservateur (plan de référence et validation qui double les points) ; fusion des rôles en solo et en duo | Test de jeu MCP à 1 et à 2 joueurs (Studio « 2 players ») | Studio |
| P1.7 | Entité et fin de nuit | À menace 100 : l'entité « chasse » en se téléportant de salle en salle quand personne ne la regarde (pas de poursuite avec pathfinding au MVP), joueurs à terre, défaite | Une défaite ou une victoire déclenche `Night.End` | Studio |
| P1.8 | Sauvegarde | DataService (ProfileStore), schéma et migrations, accès en mode simulé | Tests de migration v1→v2. En Studio, avec l'accès API désactivé, le jeu tourne quand même sur un profil en mémoire | ∥ |
| P1.9 | Contenu : 40 anomalies | Données pour environ 10 règles × catégories × salles de l'aile 1 | `validate-content` et `validate-map` passent ; 40 entrées de codex | ∥ (agent de contenu) |

### Phase 2 — Lobby et méta (semaines 4 et 5)

| # | Tâche | Livrable | Terminé quand… | ∥ |
|---|---|---|---|---|
| P2.1 | Lobby et pods | `Domain/Pods` (testé), pods de 1 à 4 places, option « amis seulement », invitations (`PromptGameInvite`), téléportation groupée vers un serveur réservé | Testé en jeu publié (expérience de test) avec 2 comptes | Studio, H |
| P2.2 | Progression | Contrats de 5 nuits → aile débloquée, XP et grades, codex alimenté par `Night.End` | Tests Lune de progression ; codex visible dans le lobby | ∥ |
| P2.3 | UI méta (React-lua) | Écran codex, grades, contrats, sélection d'aile | Captures d'écran MCP validées par Hugo | ∥ |
| P2.4 | Aile 2 | Greybox, puis habillage de l'aile Égypte | `validate-map` passe ; +10 anomalies propres à l'aile | H, Studio |
| P2.5 | Tutoriel de 60 s | Nuit guidée en solo : 3 anomalies scriptées, une démonstration par rôle | Terminé en moins de 75 s par un nouveau compte ; les étapes sont journalisées | Studio |
| P2.6 | Analytics | Funnel d'onboarding (lobby → tutoriel → 1re nuit → 1er signalement juste → 1re nuit finie → 2e nuit), événements `night_end` (résultat, durée, taille d'équipe, rôle) | Événements visibles dans le tableau de bord après publication sur l'expérience de test | ∥ |

### Phase 3 — Monétisation, récap, finitions (semaines 6 et 7)

| # | Tâche | Livrable | Terminé quand… | ∥ |
|---|---|---|---|---|
| P3.1 | Achats | `PurchaseService` : ProcessReceipt idempotent (revive à 25 R$), game passes cosmétiques, emotes, `PolicyGate` | Tests Lune : un reçu rejoué n'est pas crédité deux fois. Achat de test en Studio | ∥ |
| P3.2 | Serveurs privés | Activés dans les paramètres. Dans un serveur privé, les pods ne sont ouverts qu'aux joueurs présents | Vérifié sur l'expérience de test | H |
| P3.3 | Récap de fin de nuit | `ReplayRecorder` enregistre une chronologie (anomalies, signalements, menace, positions à 2 Hz). L'écran récap affiche la chronologie et les « moments clés ». **Clip partageable :** le client lance `StartVideoCaptureAsync` au début d'une chasse ou d'un scare (30 s maximum), puis `PromptShareCapture`. | Récap affiché en solo et à 4 ; clip enregistré et partagé sur PC et sur mobile | Studio |
| P3.4 | Mobile et manette | Commandes tactiles et HUD adapté | Test sur le téléphone de Hugo | Studio |
| P3.5 | Performance et streaming | Budget de parts, `Persistent` sur les salles, mesure de la mémoire du serveur | 4 joueurs à 60 FPS sur PC moyen, mobile au-dessus de 30 FPS | Studio |
| P3.6 | Habillage | Lumières, sons, scares réglables (option « réduit »), assets de l'artiste | Hugo valide l'ambiance | H |

### Phase 4 — Bêta (semaines 8 et 9) — *actions extérieures soumises à la validation de Hugo*

| # | Tâche | Livrable | Terminé quand… |
|---|---|---|---|
| P4.1 | QA à 4 joueurs | Liste de tests, correction des bugs bloquants | Aucun bug bloquant sur 10 nuits complètes |
| P4.2 | Conformité | Questionnaire de maturité (peur légère), descriptions et icône | Validé par Hugo avant envoi |
| P4.3 | CI d'intégration (option) | Open Cloud Luau Execution sur l'**expérience de test** (clé API fournie par Hugo) | Une suite de fumée qui passe en CI |
| P4.4 | Bêta, puis publication | Bêta fermée (amis, Discord), puis ouverture | **Décision de Hugo** |

**Agents prévus :** environ 5 fiches réutilisables (`dev-luau-domain`, `dev-luau-services-serveur`, `dev-ui-react-lua`, `contenu-anomalies`, `testeur-studio-mcp`) et un `integrateur` après chaque vague parallèle.

---

## 6. Délai révisé

| Hypothèse | Délai réaliste |
|---|---|
| Périmètre complet de la fiche concept, Hugo à **plein temps** (environ 35 h par semaine) | **9 à 10 semaines** jusqu'à la bêta publique |
| Hugo à mi-temps (environ 15 à 20 h par semaine) | 16 à 20 semaines |
| **6 semaines tenables** avec ce périmètre réduit | 1 aile + 1 aile verrouillée « bientôt » ; 30 anomalies ; 3 rôles (le conservateur fusionné avec le rôle caméras) ; récap limité aux statistiques et à la chronologie, sans clip ; game passes cosmétiques et revive seulement ; PC d'abord, mobile au minimum |

**Pourquoi le planning initial de 6 semaines ne tient pas.** La fiche concept suppose que Luau s'apprend en une semaine, mais d'autres postes sont absents ou sous-estimés :
- le **level design dans Studio**, que les agents ne peuvent pas faire seuls ;
- les tests multijoueurs : les téléportations n'existent qu'en jeu publié ;
- le mobile ;
- l'UI, qui représente environ 25 % du travail.

Les agents accélèrent nettement la logique et les tests, mais pas l'ambiance, le réglage du fun ni la QA à 4.

---

## 7. Risques

| Risque | Impact | Parade |
|---|---|---|
| Les agents produisent du Luau qui passe le linter mais ne fonctionne pas dans le moteur | Élevé | Logique pure séparée des services. Test de jeu MCP obligatoire avant chaque fusion d'un service. `CLAUDE.md` du dépôt avec les pièges connus (`task.wait`, yields, réplication) |
| Le fun ne tient pas sur 8 à 10 minutes | Élevé | Point de décision en semaine 1 (P0.5). Modificateurs de nuit. Rythme réglable par les données |
| StreamingEnabled contre les caméras qui regardent des salles lointaines | Moyen | Salles `Persistent` ou streaming désactivé (carte petite), à trancher en P0.4 |
| Téléportations et serveurs réservés impossibles à tester dans Studio | Moyen | Mode simulé de la Nuit. Expérience de test publiée dès la semaine 4 |
| Équipes de 4 difficiles à remplir | Moyen | Solo et duo équilibrés par `NightDirector`, rôles fusionnés, pods ouverts à tous |
| Perte ou duplication de données (achats) | Élevé | ProfileStore avec verrouillage de session, `purchaseLog` idempotent, tests de rejeu |
| Classement de maturité (scares) | Moyen | Option « scares réduits », pas de gore. Questionnaire rempli tôt |
| Open Cloud ou tests qui touchent les vrais DataStores | Moyen | Expérience de test distincte, clés à portée limitée, aucun secret en CI avant la phase 4 |
| Un seul Studio pour plusieurs agents | Faible à moyen | Agents en parallèle sur `Domain`, `Content` et l'UI. Intégration dans Studio l'une après l'autre |
| CaptureService limité à 30 s et voix coupées pendant l'enregistrement | Faible | Clip court centré sur le scare. Le récap ne dépend pas de la capture |

---

## 8. Questions pour Hugo

1. **Temps disponible :** plein temps ou mi-temps ? Le délai passe de 9-10 à 16-20 semaines.
2. **Périmètre :** viser la bêta en 6 semaines avec le périmètre réduit (§6), ou le périmètre complet en 9 à 10 semaines ?
3. **Luau natif :** d'accord pour abandonner roblox-ts ? Sinon, il faut accepter un compilateur sans nouvelle version depuis 2024.
4. **Mobile au MVP :** indispensable dès la bêta, ou PC et console d'abord ?
5. **Art :** l'artiste (300 à 800 €) intervient-il dès la semaine 4 (aile 2), ou seulement pour les miniatures et l'icône ?
6. **Expérience de test :** accord pour créer une seconde expérience privée, réservée aux tests (téléportations, analytics, Open Cloud) ?
7. **Hébergement du dépôt :** dépôt GitHub privé avec CI ? L'ouverture d'accès est à faire par Hugo.

---

*Sources vérifiées le 8 octobre 2026 :*
- *Rojo 7.7.0 (docs.rs) ; Roblox Docs (Script Sync, Studio MCP, Luau Execution, CaptureService, SocialService, Funnel events) ;*
- *npm (roblox-ts 3.0.0 du 12/09/2024, @rbxts/types du 30/09/2026) ;*
- *DevForum « Luau vs. TypeScript (roblox-ts) for Production Roblox Games » (juillet 2026) ;*
- *DevForum ProfileStore.*
