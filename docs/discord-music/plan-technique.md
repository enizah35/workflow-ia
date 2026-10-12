# Platine : plan technique (spec, stack, phases)

*Rédigé le 2026-10-12 par l'agent Architecte pour Hugo. Sources : `reponses-fondateur.md`, `vision.md`, `recherche-marche-et-legal.md`.*
*Ce document dit COMMENT construire. Il ne contient pas de code de production : les extraits TypeScript de la section « Contrats » sont des signatures à figer, pas des implémentations.*
*« Platine » est un nom provisoire : il n'apparaît que dans `config/defaults.ts`, le `package.json` et les fichiers de traduction.*

---

## 0. Décisions prises par défaut (faciles à changer)

Les questions ouvertes de la vision (§9) reçoivent leur défaut proposé. Chaque défaut vit à **un seul endroit** : la table `guild_settings` (valeur par serveur) initialisée depuis `config/defaults.ts`. Changer un défaut = modifier une ligne, sans toucher au code métier. Le module `policy/` est le seul à lire ces réglages pour décider d'un droit.

| Question | Défaut appliqué | Réglage |
|---|---|---|
| Qui contrôle la lecture ? | Tout le monde | `control_policy = 'everyone'` (valeur future : `'dj_role'`, V2) |
| Autoplay | Activé, désactivable par commande ou bouton | `autoplay_default = true` |
| Départ du vocal | Après 5 min seul, file conservée | `idle_timeout_s = 300`, `keep_queue_on_leave = true` |
| Playlists | Tout le monde ajoute ; créateur ou admin retire et supprime | `playlist_edit_policy = 'creator_or_admin'` |
| Historique | 12 mois, identifiant Discord + morceau uniquement, commande d'effacement | `history_retention_days = 365` |
| Code | Dépôt GitHub privé | (hors code) |
| Nom | Platine | `BOT_NAME` dans `config/defaults.ts` |
| Langue | FR par défaut, EN disponible | `locale = 'fr'` |

Autres défauts techniques :
- **Un seul serveur Discord** au MVP, mais tout le modèle est déjà indexé par `guild_id` (multi-serveur et dashboard possibles sans refonte).
- **Deezer = catalogue principal** (API publique sans clé, ISRC, genres, artistes liés, radio d'artiste). **Spotify = secondaire** : lecture des liens et playlists Spotify collés par les utilisateurs. Raison : depuis le 27/11/2024, Spotify a fermé aux nouvelles apps les endpoints `recommendations`, `related-artists` et `audio-features` [fait connu, à revérifier en phase 0] ; l'autoplay ne peut donc pas reposer sur Spotify.
- **Lecture : YouTube uniquement**, via Lavalink et le plugin youtube-source, derrière le port `AudioSource`. Aucune donnée persistée en dehors du cache `playback_matches` ne référence YouTube ; ce cache peut être vidé sans perte.

---

## 1. Spécification fonctionnelle résumée (MVP)

| # | Fonction | Commandes / interactions | Comportement clé |
|---|---|---|---|
| F1 | Lecture de base | `/play <recherche ou lien>`, `/pause`, `/resume`, `/skip`, `/stop`, `/queue`, `/remove <n>`, `/shuffle`, `/clear`, `/loop off\|track\|queue`, `/seek` | Le bot rejoint le salon vocal de l'auteur. `/stop` vide la file et quitte |
| F2 | Panneau « en cours » | Un message unique par serveur, édité à chaque titre (pas à chaque seconde) | Boutons : pause/reprise, passer, boucle, autoplay, j'aime, ajouter à une playlist (menu), « mauvais titre ? » |
| F3 | Fiabilité | (automatique) | Reprise après redémarrage du bot, de Lavalink ou coupure voix : même morceau, position à ±5 s près. Morceau injouable : candidat suivant, puis morceau suivant avec message clair. Jamais de silence muet |
| F4 | Recherche avec choix | `/search <texte>` ; `/play` choisit seul si le score de confiance est haut, sinon propose 3 à 5 résultats (menu) | Bouton « mauvais titre ? » : propose les autres candidats catalogue et audio, mémorise la correction pour le serveur |
| F5 | Playlists de serveur | `/playlist create\|add\|addcurrent\|play [shuffle]\|show\|remove\|delete\|list` | Stockent des `track_id` (métadonnées), jamais d'URL audio |
| F6 | Autoplay ambiance | `/autoplay on\|off`, `/radio [morceau]` | File vide : enchaîne via artistes liés et radio Deezer, filtré par l'historique récent (pas de répétition sur les 50 derniers titres) |
| F7 | Historique | `/history clear` (admin) | Enregistre qui a lancé quoi, fin (terminé / passé / échec), j'aime. Purge automatique au-delà de 12 mois |
| F8 | FR + EN | `/settings language fr\|en` ; noms et descriptions des commandes localisés côté Discord | Textes dans des fichiers de traduction, test de parité des clés |
| F9 | Comportement propre | (automatique) | Permissions minimales : View Channel, Send Messages, Embed Links, Connect, Speak. Aucun intent privilégié. Départ après 5 min seul |
| F10 | Observabilité | `/status` (admin), `GET /healthz` | Logs JSON, table `incidents`, healthcheck Docker, alerte optionnelle |

Non-fonctionnel : démarrage d'un morceau en moins de 3 s après `/play` (hors première résolution YouTube lente) ; RAM totale < 2,5 Go ; aucun secret dans le dépôt ; tests sans réseau.

---

## 2. Stack

| Brique | Choix (version minimale) | Justification en une ligne |
|---|---|---|
| Langage | **TypeScript 5.6+**, mode `strict` | Hugo connaît déjà TS ; un seul langage pour le bot et le futur dashboard web |
| Runtime | **Node.js ≥ 22.12** (recommandé : 24 LTS) | Version exigée par l'écosystème discord.js à l'ère DAVE ; LTS longue |
| Gestionnaire | **pnpm 9+** (via corepack) | Rapide, lockfile strict, workspaces prêts pour `apps/dashboard` en V2 |
| Discord | **discord.js 14.x** (dernière 14.x au démarrage) | Écosystème le plus mature ; on n'utilise **pas** `@discordjs/voice` : la voix est déléguée à Lavalink |
| Moteur audio | **Lavalink 4.2.2** (min. 4.2.0), image Docker officielle | Seule pile mature qui gère DAVE côté serveur audio, la reprise de session et les filtres ; isole la voix du bot |
| Source YouTube | **plugin youtube-source ≥ 1.18.2** | Maintenu par lavalink-devs ; la source YouTube native de Lavalink est dépréciée |
| Client Lavalink | **Shoukaku 4.x**, version compatible Lavalink 4.2 (envoi du `channelId` dans l'état voix, requis pour DAVE) — à vérifier tâche 0.2 ; repli : `lavalink-client` 2.x | Client fin : on garde notre propre file et nos propres règles de reprise, testables |
| Catalogue | Clients HTTP maison : **API Deezer** (publique), **Spotify Web API** (client credentials) | Le catalogue reste dans notre code (testable avec des faux), au lieu de LavaSrc dont on n'a pas besoin |
| Base de données | **PostgreSQL 16** (conteneur) + **Drizzle ORM 0.3x** + drizzle-kit (migrations) | Hugo connaît Postgres (Supabase) ; le dashboard V2 s'y branchera ; migrations versionnées et types générés |
| Tests | **Vitest 2+**, **PGlite** (Postgres en WASM, sans Docker) pour les dépôts, `undici` MockAgent pour le HTTP | Tests rapides, sans Discord, YouTube, Deezer ni Docker |
| Qualité | **Biome** (lint + format), `tsc --noEmit` | Un seul outil, configuration minimale |
| Logs | **pino** (JSON) ; rotation via le driver Docker `json-file` | Standard, peu coûteux |
| Healthcheck | `node:http` minimal (`/healthz`, `/readyz`) | Pas besoin de framework web au MVP |
| Validation config | **zod** pour `.env` et les entrées | Échec immédiat et lisible si un secret manque |
| i18n | Dictionnaires JSON typés + petite fonction `t()` maison | Deux langues, pas besoin d'i18next |
| Conteneurs | **Docker Compose v2** : `bot`, `lavalink`, `postgres` | Un seul fichier pour le PC de Hugo (Docker Desktop + WSL2) et le VPS |
| CI | **GitHub Actions** : lint, typecheck, tests, build de l'image | Dès la phase 0 |
| Hébergement | VPS **4 Go RAM**, 2 vCPU, Debian 12/13 (Netcup VPS 500 ~5 €/mois ou équivalent) | La JVM de Lavalink rend 2 Go trop juste |

**Alternatives écartées :**
- *Python + discord.py 2.7 + wavelink 3.5* : viable, mais wavelink a un rythme de maintenance plus lent, et le futur dashboard pousserait vers un second langage.
- *NodeLink* (100 % Node, DAVE natif) : séduisant pour la RAM, mais moins éprouvé que Lavalink sur YouTube. Gardé comme plan B : il parle l'API Lavalink v4, donc un changement ne touche que le conteneur.
- *Songbird (Rust)* : léger mais courbe d'apprentissage trop forte pour un solo pressé.
- *`@discordjs/voice` + yt-dlp* : on gérerait soi-même DAVE, Opus, FFmpeg et la reprise ; trop de surface pour un gain faible.

**DAVE :** c'est Lavalink (≥ 4.2.0) qui ouvre la connexion voix et chiffre les frames. Le bot ne fait que relayer `VOICE_STATE_UPDATE` et `VOICE_SERVER_UPDATE` (avec `channelId`) via Shoukaku. Le critère de la tâche 0.2 vérifie cela en vrai sur un serveur de test.

---

## 3. Structure du dépôt

```
platine/
├─ .github/workflows/ci.yml          # lint, typecheck, test, docker build
├─ .gitattributes                    # eol=lf (Hugo est sous Windows)
├─ docker-compose.yml                # bot + lavalink + postgres (prod/VPS)
├─ docker-compose.dev.yml            # surcharges dev (ports, volumes, logs lisibles)
├─ .env.example                      # toutes les variables, sans valeur secrète
├─ lavalink/
│  ├─ application.yml                # plugins, sources, resuming, mot de passe via env
│  └─ plugins/                       # (téléchargés au démarrage, ignorés par git)
├─ Dockerfile                        # bot : multi-stage, node:24-slim, utilisateur non-root
├─ package.json / pnpm-lock.yaml / tsconfig.json / biome.json / vitest.config.ts
├─ drizzle.config.ts
├─ docs/
│  ├─ setup-windows.md               # Docker Desktop, WSL2, Node, pnpm, app Discord
│  ├─ deploy-vps.md                  # installation VPS, mise à jour, sauvegardes
│  ├─ runbook.md                     # « le bot ne joue plus » : quoi regarder, dans quel ordre
│  └─ adr/                           # décisions d'architecture (1 page chacune)
├─ src/
│  ├─ main.ts                        # composition root : instancie adaptateurs et services
│  ├─ config/
│  │  ├─ env.ts                      # validation zod du .env
│  │  └─ defaults.ts                 # défauts produit (§0), nom du bot
│  ├─ domain/                        # PUR : aucun import discord.js, shoukaku, drizzle, fetch
│  │  ├─ contracts/                  # ★ FIGÉ (§5) : types, ports, événements
│  │  ├─ queue/                      # file, boucle, mélange (logique pure)
│  │  ├─ player/                     # PlayerService : machine à états d'un lecteur de serveur
│  │  ├─ resolution/                 # recherche catalogue → candidats audio → score → repli
│  │  ├─ autoplay/                   # choix des morceaux suivants
│  │  ├─ playlists/                  # PlaylistService
│  │  ├─ history/                    # HistoryService + purge
│  │  ├─ recovery/                   # RecoveryService : sauvegarde périodique et reprise
│  │  └─ policy/                     # droits (control_policy, playlist_edit_policy)
│  ├─ adapters/
│  │  ├─ catalog/deezer/             # implémente CatalogProvider
│  │  ├─ catalog/spotify/            # implémente CatalogProvider (liens + playlists)
│  │  ├─ audio/youtube-lavalink/     # implémente AudioSource
│  │  ├─ engine/lavalink-shoukaku/   # implémente PlayerEngine
│  │  ├─ db/                         # schema.ts, migrations/, repositories/ (implémente les *Repository)
│  │  └─ fakes/                      # FakeCatalog, FakeAudioSource, FakePlayerEngine, InMemory repos, FakeClock
│  ├─ discord/
│  │  ├─ client.ts                   # connexion gateway, intents (Guilds, GuildVoiceStates)
│  │  ├─ commands/                   # un fichier par commande ; handlers testables (entrée = DTO)
│  │  ├─ components/                 # boutons et menus du panneau
│  │  ├─ panel/                      # rendu du panneau « en cours » (pur) + édition limitée en débit
│  │  ├─ registry.ts                 # ★ partagé : liste des commandes
│  │  └─ deploy-commands.ts          # enregistrement des slash commands (dev : guilde ; prod : guilde)
│  ├─ i18n/
│  │  ├─ t.ts
│  │  └─ locales/{fr,en}/<module>.json   # un fichier par module → pas de conflit entre agents
│  └─ ops/
│     ├─ health.ts                   # /healthz, /readyz
│     ├─ logger.ts
│     └─ incidents.ts                # écrit dans la table incidents
└─ tests/
   ├─ unit/                          # domaine avec fakes
   ├─ db/                            # dépôts sur PGlite
   ├─ adapters/                      # HTTP Deezer/Spotify sur fixtures enregistrées
   ├─ contract/                      # même suite exécutée contre fake ET adaptateur réel (réel = manuel, hors CI)
   └─ scenarios/                     # parcours complets : coupure, redémarrage, échec YouTube…
```

Règle d'architecture vérifiée en CI : `src/domain/**` n'importe rien de `src/adapters`, `src/discord`, ni des paquets `discord.js`, `shoukaku`, `drizzle-orm`, `undici` (règle Biome `noRestrictedImports` ou petit script).

---

## 4. Modèle de données (PostgreSQL)

Principes : tout est rangé **par serveur (`guild_id`) et par morceau (`track_id`)**, ce qui suffit au dashboard V2. Les identifiants Discord sont des `text` (snowflakes). Les horodatages sont en `timestamptz`. Aucune URL audio hors de `playback_matches`.

| Table | Colonnes principales | Rôle / remarques |
|---|---|---|
| `guild_settings` | `guild_id` PK, `locale`, `control_policy`, `dj_role_id` null, `autoplay_default`, `idle_timeout_s`, `keep_queue_on_leave`, `playlist_edit_policy`, `history_retention_days`, `updated_at` | Défauts §0. Lu par `policy/` et l'i18n |
| `tracks` | `id` uuid PK, `isrc` null unique, `title`, `artists` text[], `primary_artist`, `album` null, `duration_ms`, `artwork_url` null, `genres` text[], `explicit` bool, `created_at` | **Identité catalogue** d'un morceau. Dédoublonnage par ISRC, sinon par (artiste normalisé, titre normalisé, durée ±3 s) |
| `track_external_ids` | (`provider`, `external_id`) PK, `track_id` FK, `url` | `provider` ∈ `deezer`, `spotify`, `musicbrainz`, `lrclib` (V2). Liens vers le catalogue, jamais vers l'audio |
| `artist_external_ids` | (`provider`, `external_id`) PK, `name` | Sert l'autoplay (artistes liés Deezer) |
| `playback_matches` | `id`, `track_id` FK, `guild_id` null, `source` (`youtube`…), `source_ref`, `score`, `status` (`ok`, `bad`, `unplayable`), `reported_by` null, `checked_at` | **Seul endroit** avec une référence de source audio. Cache de résolution ; `guild_id` non nul = correction « mauvais titre ? » propre au serveur. Vidable sans perte |
| `playlists` | `id` uuid PK, `guild_id`, `name`, `created_by`, `created_at`, `updated_at`; unique (`guild_id`, lower(`name`)) | Playlists de serveur |
| `playlist_items` | `id`, `playlist_id` FK cascade, `track_id` FK, `position` int, `added_by`, `added_at` | Ordre explicite (réorganisation V2) |
| `queue_items` | `id` uuid PK, `guild_id`, `track_id` FK, `position` int, `requested_by`, `origin` (`user`, `playlist`, `autoplay`, `radio`), `added_at` | **File persistante** : survit aux redémarrages |
| `player_state` | `guild_id` PK, `voice_channel_id`, `text_channel_id`, `panel_message_id`, `current_queue_item_id` null, `current_track_id` null, `position_ms`, `position_at`, `paused`, `loop_mode`, `autoplay`, `status` (`idle`, `playing`, `paused`, `left_idle`, `recovering`), `updated_at` | Instantané pour la reprise (écrit à chaque événement + toutes les 5 s en lecture) |
| `play_history` | `id` bigserial, `guild_id`, `track_id`, `requested_by` null (null pour autoplay), `origin`, `started_at`, `ended_at`, `played_ms`, `end_reason` (`finished`, `skipped`, `failed`, `stopped`, `replaced`), `wrong_match` bool | Nourrit l'autoplay, les métriques (North Star, taux de skip) et les recos V2. Index (`guild_id`, `started_at` desc) |
| `track_likes` | (`guild_id`, `user_id`, `track_id`) PK, `created_at` | « J'aime » |
| `incidents` | `id`, `guild_id` null, `kind` (`voice_disconnect`, `lavalink_down`, `track_failed`, `resume_ok`, `resume_failed`, `stuck`…), `details` jsonb, `recovered` bool, `at` | Fiabilité mesurable (garde-fou « sessions sans incident ») |
| `voice_sessions` | `id`, `guild_id`, `started_at`, `ended_at`, `max_humans`, `listen_ms_2plus` | Calcul de la North Star (heures d'écoute avec ≥ 2 humains) |

**Préparé pour la V2, non créé au MVP :** `users` (profil OAuth Discord), `web_sessions`, `lyrics_cache` (`track_id`, `synced` text, `plain` text, `source`, `fetched_at`). Le dashboard lira les tables ci-dessus et recevra les événements du domaine (§5.3) via un pont WebSocket.

**Rétention :** une tâche planifiée quotidienne (dans le bot) supprime `play_history` et `incidents` plus vieux que `history_retention_days`. `/history clear` supprime l'historique du serveur (et les `track_likes` si l'option est passée).

**Sauvegardes :** `pg_dump` quotidien sur le VPS, 7 jours conservés (tâche 3.3).

---

## 5. Contrats (à figer en phase 0, à remettre tels quels à tous les agents de la première vague)

Fichiers : `src/domain/contracts/{track.ts, ports.ts, events.ts, repositories.ts, settings.ts, errors.ts}`. Toute modification après la phase 0 passe par le Chef et une ADR. Les signatures ci-dessous sont normatives ; les noms de champs le sont aussi.

### 5.1 Types de base

```ts
// track.ts
export type GuildId = string; export type UserId = string; export type TrackId = string;

export interface CatalogTrack {           // identité catalogue, jamais d'URL audio
  title: string;
  artists: string[];
  album?: string;
  durationMs: number;
  isrc?: string;
  artworkUrl?: string;
  genres: string[];
  explicit: boolean;
  externalIds: { provider: 'deezer' | 'spotify' | 'musicbrainz'; id: string; url?: string }[];
}
export interface Track extends CatalogTrack { id: TrackId }       // persisté

export type QueueOrigin = 'user' | 'playlist' | 'autoplay' | 'radio';
export interface QueueItem { id: string; track: Track; requestedBy: UserId | null; origin: QueueOrigin; addedAt: Date }
export type LoopMode = 'off' | 'track' | 'queue';
export type EndReason = 'finished' | 'skipped' | 'failed' | 'stopped' | 'replaced';
```

### 5.2 Ports

```ts
// ports.ts
export interface CatalogProvider {                       // Deezer, Spotify ; fakes en test
  readonly id: 'deezer' | 'spotify';
  search(query: string, opts: { limit: number; signal?: AbortSignal }): Promise<CatalogTrack[]>;
  resolveUrl(url: string): Promise<CatalogTrack[] | null>;   // lien de titre/album/playlist ; null = pas pour moi
  related(seed: CatalogTrack, opts: { limit: number }): Promise<CatalogTrack[]>; // autoplay ; [] si non supporté
}

export interface PlayableCandidate {                     // produit par une AudioSource, consommé par PlayerEngine
  source: string;            // 'youtube', plus tard 'jamendo', 'upload'…
  ref: string;               // opaque pour le domaine (identifiant de la source)
  engineHandle: string;      // opaque : piste encodée Lavalink
  title: string; author: string; durationMs: number;
  score: number;             // 0..1, calculé par la source (durée, titre, artiste, chaîne officielle/« Topic »)
}

export interface AudioSource {                           // ★ le port à remplacer par une source légale
  readonly id: string;
  findCandidates(track: CatalogTrack, opts: { limit: number; signal?: AbortSignal }): Promise<PlayableCandidate[]>;
  fromRef(ref: string): Promise<PlayableCandidate | null>; // recharge depuis playback_matches
}

export interface PlayerEngine {                          // Lavalink via Shoukaku ; fake en test
  join(guildId: GuildId, voiceChannelId: string): Promise<void>;
  leave(guildId: GuildId): Promise<void>;
  play(guildId: GuildId, c: PlayableCandidate, opts?: { startMs?: number; paused?: boolean }): Promise<void>;
  stop(guildId: GuildId): Promise<void>;
  pause(guildId: GuildId, paused: boolean): Promise<void>;
  seek(guildId: GuildId, positionMs: number): Promise<void>;
  position(guildId: GuildId): number | null;             // ms, dernière valeur connue
  isReady(): boolean;                                    // nœud Lavalink connecté
  on(listener: (e: EngineEvent) => void): () => void;
}
export type EngineEvent =
  | { type: 'trackStart'; guildId: GuildId }
  | { type: 'trackEnd'; guildId: GuildId; reason: 'finished' | 'loadFailed' | 'stopped' | 'replaced' | 'cleanup' }
  | { type: 'trackException'; guildId: GuildId; message: string; severity: 'common' | 'suspicious' | 'fault' }
  | { type: 'trackStuck'; guildId: GuildId; thresholdMs: number }
  | { type: 'voiceClosed'; guildId: GuildId; code: number; byRemote: boolean }
  | { type: 'nodeDown' } | { type: 'nodeUp'; resumed: boolean };

export interface Clock { now(): Date; setTimeout(fn: () => void, ms: number): () => void } // FakeClock en test
```

### 5.3 Événements du domaine

Bus en mémoire typé (`EventBus.emit / on`). Consommateurs au MVP : panneau, historique, recovery, incidents, métriques. En V2 : pont WebSocket vers le dashboard, paroles.

```ts
// events.ts
export type DomainEvent =
  | { type: 'TrackStarted';    guildId: GuildId; item: QueueItem; candidate: Pick<PlayableCandidate,'source'|'ref'>; at: Date }
  | { type: 'TrackEnded';      guildId: GuildId; item: QueueItem; reason: EndReason; playedMs: number; at: Date }
  | { type: 'TrackFailed';     guildId: GuildId; item: QueueItem; attempt: number; willRetry: boolean; error: string; at: Date }
  | { type: 'QueueChanged';    guildId: GuildId; size: number; at: Date }
  | { type: 'PlayerStateChanged'; guildId: GuildId; status: 'idle'|'playing'|'paused'|'left_idle'|'recovering'; paused: boolean; loop: LoopMode; autoplay: boolean; at: Date }
  | { type: 'PositionTick';    guildId: GuildId; positionMs: number; at: Date }   // toutes les 5 s ; jamais vers Discord
  | { type: 'VoiceJoined' | 'VoiceLeft'; guildId: GuildId; channelId: string; reason?: 'idle'|'stop'|'kicked'|'error'; at: Date }
  | { type: 'RecoveryAttempted'; guildId: GuildId; trigger: 'bot_restart'|'node_down'|'voice_closed'; ok: boolean; at: Date }
  | { type: 'TrackLiked';      guildId: GuildId; userId: UserId; trackId: TrackId; at: Date }
  | { type: 'WrongMatchReported'; guildId: GuildId; userId: UserId; trackId: TrackId; ref: string; at: Date }
  | { type: 'PlaylistChanged'; guildId: GuildId; playlistId: string; at: Date };
```

### 5.4 Dépôts et réglages

```ts
// repositories.ts — implémentés en Drizzle (adapters/db) ET en mémoire (adapters/fakes)
export interface TrackRepository   { upsert(t: CatalogTrack): Promise<Track>; get(id: TrackId): Promise<Track | null> }
export interface QueueRepository   { load(g: GuildId): Promise<QueueItem[]>; replace(g: GuildId, items: QueueItem[]): Promise<void> }
export interface PlayerStateRepository { load(g: GuildId): Promise<PlayerSnapshot | null>; save(s: PlayerSnapshot): Promise<void>; listActive(): Promise<PlayerSnapshot[]> }
export interface MatchRepository   { best(trackId: TrackId, g: GuildId): Promise<StoredMatch | null>; record(m: StoredMatch): Promise<void>; markBad(trackId: TrackId, ref: string, g: GuildId, by: UserId): Promise<void> }
export interface PlaylistRepository { /* create, rename, delete, addItem, removeItem, list, items */ }
export interface HistoryRepository { start(...): Promise<number>; end(...): Promise<void>; recent(g: GuildId, n: number): Promise<TrackId[]>; purgeOlderThan(d: Date): Promise<number>; clear(g: GuildId): Promise<void> }
export interface SettingsRepository { get(g: GuildId): Promise<GuildSettings>; update(g: GuildId, p: Partial<GuildSettings>): Promise<GuildSettings> }
// PlayerSnapshot reprend les colonnes de player_state ; StoredMatch celles de playback_matches.

// errors.ts : erreurs typées, traduites par l'i18n
export type DomainErrorCode = 'NOT_IN_VOICE' | 'NOT_FOUND' | 'UNPLAYABLE' | 'FORBIDDEN' | 'QUEUE_EMPTY' | 'ENGINE_UNAVAILABLE' | 'PLAYLIST_EXISTS' | 'PLAYLIST_NOT_FOUND';
```

### 5.5 Pipeline de résolution (comportement contractuel)

1. `/play` avec lien → `CatalogProvider.resolveUrl` (Deezer, puis Spotify). Texte libre → `search` Deezer (5 résultats).
2. Score de confiance catalogue : si le 1er résultat devance nettement le 2e (et correspond aux mots de la requête), on le prend ; sinon menu de 3 à 5 choix.
3. Pour le morceau choisi : `MatchRepository.best` (correction du serveur, puis global, statut `ok`) ; sinon `AudioSource.findCandidates` (recherche par ISRC, puis « artiste – titre » ; score = durée ±5 s, titre, artiste, chaîne « - Topic »/officielle, pénalités « live », « remix », « cover », « sped up » si absents de la requête).
4. Lecture du meilleur candidat. Sur `trackException`, `loadFailed` ou `trackStuck` : candidat suivant (max 3 tentatives, 10 s au total), match marqué `unplayable`, incident enregistré. Après épuisement : message « injouable, passé au suivant », `TrackEnded(reason='failed')`, morceau suivant.
5. Jamais de silence muet : tout échec produit un message dans le salon du panneau.

### 5.6 Reprise (comportement contractuel)

- **Sauvegarde** : `player_state` écrit à chaque événement de lecteur et toutes les 5 s pendant la lecture ; `queue_items` à chaque `QueueChanged`.
- **Redémarrage du bot** : au `ready`, `listActive()` ; pour chaque serveur `playing`/`paused` (et `recovering`) dont le salon vocal contient encore au moins un humain : rejoindre, rejouer le morceau courant à `position_ms + (now - position_at)` si `playing` (borne : durée − 5 s), sinon à `position_ms` en pause.
- **Lavalink tombe** : Shoukaku reconnecte ; si la session Lavalink est reprise (`resuming` activé, délai 60 s), rien à faire ; sinon même procédure que le redémarrage.
- **Fermeture voix** (codes 4006, 4009, 4014 hors expulsion…) : rejoindre et reprendre, 3 essais avec attente exponentielle (1 s, 4 s, 15 s), puis abandon propre avec message et incident.
- **Expulsion volontaire** (déplacement ou déconnexion par un membre) : on ne revient pas.

---

## 6. Phases et tâches

Convention : chaque tâche = une branche et un worktree ; les tâches marquées ∥ d'une même vague partent en parallèle ; après chaque vague, un agent **Intégrateur** fusionne, répare les jonctions, fait passer la CI, et ouvre **une seule PR par phase** (merge sur `main` validé par Hugo).

### Fichiers partagés : un seul propriétaire par phase

| Fichier / dossier | Propriétaire | Les autres agents… |
|---|---|---|
| `src/domain/contracts/**` | Chef (figé en 0.3) | lisent seulement ; un besoin de changement remonte dans `BESOIN_SOUS_AGENTS` / leur rapport |
| `package.json`, `pnpm-lock.yaml` | Intégrateur de la phase | listent les dépendances voulues dans leur rapport ; ne touchent pas au lockfile |
| `src/adapters/db/schema.ts` + `migrations/` | Agent « Base de données » de la phase (ou l'Intégrateur) | décrivent la colonne voulue ; une seule migration par phase |
| `src/discord/registry.ts`, `src/main.ts` | Intégrateur | exportent leur commande/service depuis leur propre fichier |
| `src/i18n/locales/{fr,en}/<module>.json` | L'agent du module | un fichier par module : pas de conflit |
| `docker-compose*.yml`, `lavalink/application.yml`, `Dockerfile`, `.github/workflows/*` | Agent « DevOps » | ne modifient pas |
| `src/adapters/fakes/**` | Créé en 0.3 ; ensuite chaque agent peut **ajouter** un fichier, jamais modifier ceux des autres | |

### Phase 0 : socle (≈ 1 semaine) — 4 tâches

Vague A : 0.1 seule. Vague B : 0.2 ∥ 0.3. Vague C : 0.4. PR « Phase 0 ».

| # | Tâche | Livrable | Critère « terminé » | Profil d'agent |
|---|---|---|---|---|
| 0.1 | Échafaudage du dépôt | Arborescence §3, TS strict, pnpm, Biome, Vitest, `.gitattributes`, `.env.example`, `ci.yml` (lint, typecheck, test, build image), règle d'import du domaine | `pnpm i && pnpm lint && pnpm typecheck && pnpm test` passent en local et en CI ; un import interdit dans `domain/` fait échouer la CI | Dev TS outillage / CI GitHub Actions |
| 0.2 ∥ | Conteneurs + spike voix DAVE | `Dockerfile`, `docker-compose(.dev).yml` (bot, Lavalink 4.2.2 + youtube-source 1.18.2, Postgres 16, healthchecks, `restart: unless-stopped`, limites mémoire, rotation de logs), `application.yml` (resuming 60 s), `docs/setup-windows.md`, ADR « client Lavalink » (Shoukaku vs lavalink-client, avec la version qui envoie `channelId`) | Sur le PC de Hugo : `docker compose up` démarre les 3 services « healthy » ; le bot jetable du spike rejoint un salon et joue `ytsearch:` un morceau avec DAVE actif (vérifier : pas d'erreur 4017/« E2EE required ») | DevOps Docker + Lavalink (spike) |
| 0.3 ∥ | Contrats + fakes + schéma initial | `src/domain/contracts/*` (§5 à la lettre), `EventBus`, `adapters/fakes/*` (FakeCatalog, FakeAudioSource programmable : échecs, lenteurs ; FakePlayerEngine émettant des `EngineEvent` ; dépôts en mémoire ; FakeClock), `schema.ts` + 1re migration (§4), dépôts Drizzle de base, tests PGlite | Les fakes passent la suite `tests/contract/` ; migration appliquée sur PGlite et sur le Postgres du compose ; couverture des dépôts ≥ 80 % | Dev TS domaine / modélisation de données |
| 0.4 | Bot minimal de bout en bout | `discord/client.ts`, `deploy-commands.ts`, adaptateurs `engine/lavalink-shoukaku` et `audio/youtube-lavalink` (version simple : 1er candidat), commande `/play <texte>` + `/stop`, logger pino, `/healthz` (gateway, nœud Lavalink, DB) | Sur le serveur de test : `/play daft punk one more time` joue en moins de 5 s ; `curl /healthz` = 200, passe à 503 si on arrête Lavalink ; CI verte | Dev discord.js + Lavalink |

### Phase 1 : MVP-A « lecture fiable » (≈ 2 semaines) — 7 tâches

Vague A (∥) : 1.1, 1.2, 1.3, 1.6. Vague B (∥, dépend de 1.1) : 1.4, 1.5, 1.7. Intégrateur, PR « Phase 1 ».

| # | Tâche | Livrable | Critère « terminé » | Profil d'agent |
|---|---|---|---|---|
| 1.1 ∥ | PlayerService + file | `domain/queue`, `domain/player` : machine à états, file, boucle, mélange, idle 5 min seul (via Clock), émission des événements §5.3 | Tests unitaires avec FakePlayerEngine + FakeClock : tous les états et transitions ; départ à 5 min seul, file conservée ; aucun import Discord | Dev TS domaine (machine à états) |
| 1.2 ∥ | Catalogue Deezer | `adapters/catalog/deezer` : `search`, `resolveUrl` (titre, album, playlist), `related` (artistes liés + radio d'artiste), limites de débit et délai d'attente | Tests sur fixtures HTTP enregistrées (aucun appel réseau en CI) ; suite `contract/` passe contre le fake et (manuellement) contre l'API réelle | Dev intégration API REST |
| 1.3 ∥ | Résolution audio YouTube + repli | `adapters/audio/youtube-lavalink` complet (ISRC puis texte, scoring §5.5), `domain/resolution` (cache `playback_matches`, 3 tentatives, marquage `unplayable`) | Scénarios : 1er candidat en échec → 2e joué ; 3 échecs → message + morceau suivant ; jeu de 30 paires (catalogue, résultats YouTube enregistrés) : bon candidat en tête ≥ 85 % | Dev TS résolution / scoring |
| 1.4 | Persistance et reprise | `domain/recovery`, branchement dépôts `queue_items` / `player_state`, gestion `nodeDown/nodeUp`, `voiceClosed`, redémarrage | `tests/scenarios/` : redémarrage simulé → même morceau, position ±5 s ; Lavalink coupé 30 s → reprise ; 3 fermetures voix → abandon propre + incident. Test manuel : `docker compose restart bot` en pleine lecture reprend | Dev TS fiabilité / systèmes distribués |
| 1.5 | Commandes de lecture + panneau | `discord/commands/*` (F1), `discord/panel` (rendu pur + édition limitée : max 1 édition / 2 s, uniquement sur événement), boutons F2 (sauf « playlist » et « mauvais titre ? » branchés en phase 2) | Handlers testés avec des DTO d'interaction (sans discord.js réel) ; rendu du panneau testé en snapshot FR et EN ; test manuel des boutons | Dev discord.js UI (interactions, embeds) |
| 1.6 ∥ | i18n + réglages | `i18n/t.ts`, locales FR/EN des modules, localisation des noms/descriptions de commandes, `guild_settings` + `/settings language`, module `policy/` | Test de parité : chaque clé existe en FR et EN, aucun texte en dur dans `discord/` (règle de lint ou test) ; changement de langue visible immédiatement | Dev TS i18n |
| 1.7 | Observabilité | `ops/incidents.ts` (abonné aux événements), `voice_sessions` + calcul heures ≥ 2 humains, `/status` (admin : uptime, nœud, file, incidents 24 h), `/readyz`, `docs/runbook.md` | Chaque type d'incident des scénarios 1.4 crée une ligne ; `/status` affiché ; runbook relu par le Chef | Dev TS ops / observabilité |

### Phase 2 : MVP-B « trouver, garder, enchaîner » (≈ 2 semaines) — 6 tâches

Vague A (∥) : 2.1, 2.2, 2.3, 2.5. Vague B : 2.4 (dépend de 2.5 pour l'historique), 2.6. Intégrateur, PR « Phase 2 ».

| # | Tâche | Livrable | Critère « terminé » | Profil d'agent |
|---|---|---|---|---|
| 2.1 ∥ | Recherche avec choix + « mauvais titre ? » | `/search`, menu 3 à 5 résultats sur `/play` incertain, bouton « mauvais titre ? » (autres candidats catalogue et audio, `markBad`, correction par serveur) | Scénarios : correction mémorisée rejouée au prochain `/play` du même titre dans le serveur ; `play_history.wrong_match` renseigné | Dev discord.js UI + résolution |
| 2.2 ∥ | Catalogue Spotify (liens) | `adapters/catalog/spotify` : titres, albums, playlists publiques (pagination), conversion en `CatalogTrack` puis rapprochement Deezer par ISRC | Fixtures enregistrées ; playlist de 200 titres convertie, taux de rapprochement ISRC mesuré et logué ; aucune donnée audio Spotify utilisée | Dev intégration API REST (OAuth client credentials) |
| 2.3 ∥ | Playlists de serveur | `domain/playlists`, `/playlist …`, bouton « ajouter à une playlist » du panneau, droits via `policy/` | Tests : tout le monde ajoute ; seul créateur/admin retire et supprime (et cela change en modifiant `playlist_edit_policy`) ; jouer en aléatoire ; noms uniques par serveur | Dev TS fonctionnalité CRUD + discord.js |
| 2.4 | Autoplay ambiance + `/radio` | `domain/autoplay` : graines = 3 derniers titres joués, `related` Deezer, filtre des 50 derniers, diversité d'artistes, `origin='autoplay'` ; bouton et commande on/off | Tests avec FakeCatalog : pas de répétition, pas plus de 2 titres du même artiste d'affilée ; autoplay désactivé = arrêt propre. Banc H3 : script produisant 10 suites pour 5 graines, à faire noter par les amis | Dev TS algorithmique / recommandation simple |
| 2.5 ∥ | Historique, j'aime, rétention | `domain/history`, bouton « j'aime », purge quotidienne, `/history clear` | Tests : purge à 365 j (paramètre lu dans `guild_settings`) ; effacement par serveur ; aucune donnée autre que `user_id` + morceau | Dev TS données / RGPD de base |
| 2.6 | Banc d'évaluation recherche (H4) | Script `pnpm eval:search` : rejoue un fichier de 50 requêtes réelles fourni par Hugo, sort le taux « bon titre en 1er » et « dans le top 5 » | Rapport généré ; résultat consigné dans le journal de mission (objectif 85 % / 95 %) | Dev TS outillage / QA |

### Phase 3 : MVP-C « durcir et mettre en service » (≈ 1 semaine) — 5 tâches

Vague A (∥) : 3.1, 3.2, 3.3. Vague B : 3.4, puis 3.5 (Hugo + Chef). PR « Phase 3 ».

| # | Tâche | Livrable | Critère « terminé » | Profil d'agent |
|---|---|---|---|---|
| 3.1 ∥ | Tests de chaos | Script `pnpm chaos` (en local, compose) : tuer le bot, tuer Lavalink, couper le réseau du conteneur Lavalink 20 s, pendant une lecture | 10 cycles sur 10 reprennent au bon morceau (H2) ; résultats dans `docs/runbook.md` | QA fiabilité / DevOps |
| 3.2 ∥ | Robustesse YouTube | Config youtube-source documentée et commentée (clients, serveur de cipher distant, option poToken/OAuth **désactivée par défaut**), alerte quand le taux d'échec YouTube > 20 % sur 1 h, ADR « sortie de YouTube » (sources légales candidates et effort) | Simulation d'échec massif via FakeAudioSource → alerte émise ; ADR relue | Dev Lavalink / youtube-source |
| 3.3 ∥ | Déploiement VPS | `docs/deploy-vps.md` (durcissement SSH, pare-feu, Docker, utilisateur dédié, mises à jour), script `deploy.sh` (pull image GHCR, migrations, redémarrage), `pg_dump` quotidien 7 jours, workflow de publication d'image (tag manuel) | Déploiement à blanc réussi sur une VM locale ou le VPS, **exécuté par Hugo** ; restauration d'une sauvegarde testée | DevOps VPS |
| 3.4 | Relecture sécurité et qualité | Revue : secrets, permissions bot, injection dans les entrées, limites de débit, logs sans données perso inutiles | Aucun point bloquant ouvert ; corrections fusionnées | Relecteur sécurité TS |
| 3.5 | Mise en service + test H1 | Bot en prod sur le serveur d'amis, à côté de l'ancien bot pendant 2 semaines ; tableau de bord SQL simple (requêtes North Star et garde-fous dans `docs/metrics.sql`) | Critères de la vision : ≥ 95 % de sessions sans incident, part d'écoute > 50 % en semaine 2 | (Chef + Hugo, pas d'agent) |

### V2 (après 3 à 4 semaines d'usage sans incident majeur) — 4 lots

| Lot | Contenu | Tâches (≈) | Notes d'architecture |
|---|---|---|---|
| **V2.1 Paroles synchronisées** (1er lot) | Client LRCLIB (`/api/get` par titre/artiste/album/durée, puis `/api/search`), `lyrics_cache`, affichage dans le panneau (fenêtre de 3 lignes, édition au plus toutes les 2 à 3 s, synchronisée sur `PositionTick`), bouton masquer, repli paroles simples puis « indisponibles » | 4 : client LRCLIB + cache ∥ rendu panneau + limiteur ∥ réglage on/off par serveur ; puis intégration | Nouveau port `LyricsProvider` ; zéro impact sur la lecture. Mesurer H6 |
| **V2.2 Dashboard web** | `apps/dashboard` (workspace pnpm) : connexion OAuth Discord, droits par serveur (membre / admin), file en direct, playlists (réorganiser, renommer, importer), historique, réglages ; pont WebSocket des `DomainEvent` ; reverse proxy Caddy (HTTPS auto) | 6 à 8, précédées d'un agent Architecte dédié (choix front : Next.js ou Vite + React, au vu du profil Expo/React de Hugo) | Tables `users`, `web_sessions` ; le bot expose une API interne (pas d'accès direct du front à la base). Valider H7 avant de lancer |
| **V2.3 Recommandations** | « Vous aimerez aussi » du serveur, playlist automatique hebdomadaire, à partir de `play_history`, `track_likes` et `related` | 3 à 4 | Option : Last.fm `track.getSimilar` (clé gratuite) comme second `CatalogProvider.related` |
| **V2.4 Confort** | Vote skip et rôle DJ (`control_policy='dj_role'`), filtres audio Lavalink, import de playlist Deezer/Spotify en playlist de serveur | 3 à 5 | Tout passe par `policy/` et `PlayerEngine` (ajouter `setFilters`) |

**Récapitulatif :** phase 0 = 4 tâches, phase 1 = 7, phase 2 = 6, phase 3 = 5 (dont 1 sans agent), V2 = 4 lots (~16 à 21 tâches). Au plus 4 agents en parallèle par vague, plus 1 intégrateur par phase.

---

## 7. Tests et CI (dès la phase 0)

- **Pyramide :** domaine en tests unitaires avec fakes (majorité) ; dépôts sur PGlite ; adaptateurs HTTP sur fixtures enregistrées ; scénarios bout en bout « simulés » (domaine + fakes + vraie base PGlite + FakeClock).
- **Interdits en CI :** aucun appel à Discord, YouTube, Deezer, Spotify, LRCLIB ; aucune variable secrète requise. Les tests « réels » (`tests/contract/*.live.test.ts`) ne tournent que si `LIVE_TESTS=1` sur le PC de Hugo.
- **Handlers Discord testables :** chaque commande = `handler(ctx: CommandContext)` où `CommandContext` est un DTO (guildId, userId, voiceChannelId, options, `reply()` espionnable). discord.js n'est touché que dans un adaptateur fin.
- **CI GitHub Actions** (`ci.yml`, sur PR et `main`) : `pnpm install --frozen-lockfile`, `biome ci`, `tsc --noEmit`, `vitest run --coverage` (seuil 70 % global, 85 % sur `domain/`), `docker build`. Durée cible < 5 min. Publication d'image GHCR sur tag (phase 3).
- **Protection de `main`** : PR obligatoire, CI verte requise (à activer par Hugo).

---

## 8. Ce que Hugo doit faire lui-même (et quand)

| Quand | Action | Détails / risque |
|---|---|---|
| **Avant la phase 0** | Installer **Docker Desktop** (avec WSL2), **Node 24 LTS** (≥ 22.12), activer **corepack** (pnpm), Git | Sous Windows, cloner le dépôt dans le système de fichiers WSL2 de préférence (performances Docker) |
| **Avant la phase 0** | Créer le **dépôt GitHub privé** `platine` et donner l'accès | Push sur `main` uniquement via PR validée par Hugo |
| **Avant la tâche 0.2** | Créer **deux applications Discord** (Developer Portal) : `Platine Dev` et `Platine` ; copier les **tokens** dans `.env` (jamais dans le dépôt) ; désactiver « Public Bot » ; aucun intent privilégié | Inviter le bot dev sur un **serveur Discord de test** créé par Hugo, avec les permissions View Channel, Send Messages, Embed Links, Connect, Speak (scopes `bot` + `applications.commands`) |
| **Avant la tâche 2.2** | Créer une app **Spotify for Developers** (client ID + secret, flux client credentials) | Les règles du mode développement Spotify évoluent (restrictions d'accès en 2024-2026) : vérifier à la création qu'un compte gratuit suffit et que les endpoints `tracks`/`playlists` restent accessibles. Si bloqué : liens Spotify non pris en charge, Deezer seul (aucune clé requise) |
| **Avant la tâche 2.6** | Fournir **50 recherches réelles** du groupe (liste texte) | Banc H4 |
| **Seulement si YouTube bloque** (phase 3 ou plus tard) | Créer un **compte Google dédié et jetable** pour l'OAuth/poToken de youtube-source | **Risque de bannissement du compte** : ne jamais utiliser le compte personnel. Violation des CGU YouTube assumée pour un usage privé |
| **Avant la tâche 3.3** | Louer le **VPS** (4 Go RAM, ~5 €/mois, Debian), ajouter sa clé SSH, me donner l'IP | Les IP de datacenter sont plus souvent bloquées par YouTube ; plan B : héberger sur un PC ou un Raspberry Pi à la maison (même compose) |
| **Tâche 3.3** | **Lancer le déploiement en production** (ou valider qu'un agent le fasse) | Action à impact extérieur : validation obligatoire |
| **Optionnel, phase 3** | Compte gratuit **healthchecks.io** ou webhook Discord privé pour les alertes | Sans cela, alertes visibles seulement dans les logs et `/status` |
| **Phase 3.5** | Faire tester par les amis, noter les enchaînements autoplay (H3), signaler les « mauvais titres » | |
| **V2.2** | Nom de domaine (optionnel) et URL de redirection OAuth Discord | Seulement si le dashboard est lancé |

---

## 9. Risques

| # | Risque | Probabilité / impact | Parade |
|---|---|---|---|
| R1 | **YouTube casse ou bloque l'IP du VPS** (poToken, cipher, 403, « sign in to confirm ») | Élevée / élevé | youtube-source à jour (veille mensuelle), serveur de cipher distant, repli multi-candidats, alerte sur taux d'échec (3.2), compte Google jetable en dernier recours, plan B hébergement maison. Port `AudioSource` pour changer de source |
| R2 | **Compatibilité DAVE** de la chaîne Lavalink ↔ client (Shoukaku doit transmettre `channelId`) | Moyenne / bloquant | Spike en tâche 0.2 avant tout le reste ; repli `lavalink-client` ; plan C NodeLink (même API) |
| R3 | **Restrictions API Spotify** (endpoints fermés, mode dev limité) | Moyenne / moyen | Deezer catalogue principal ; Spotify limité aux liens, fonction dégradable |
| R4 | **Juridique / CGU** (YouTube, métadonnées Spotify) | Faible en privé / élevé en public | Bot non public, non listé, non monétisé ; aucune ouverture publique avant le remplacement de la source audio (ADR 3.2) |
| R5 | **Mémoire du VPS** (JVM Lavalink + Postgres + Node) | Moyenne / moyen | 4 Go RAM ; `-Xmx768m` pour Lavalink ; limites mémoire Docker ; NodeLink si besoin |
| R6 | **Limites de débit Discord** (édition du panneau, puis paroles en V2) | Moyenne / faible | Édition sur événement uniquement, limiteur 1 édition / 2 s, jamais de position en temps réel au MVP |
| R7 | **Mauvais appariement** (remix, live, titres FR rares) | Moyenne / moyen | ISRC d'abord, scoring durée/titre/chaîne, « mauvais titre ? » mémorisé, banc H4 |
| R8 | **Reprise imparfaite** (position décalée, double lecture après restart) | Moyenne / élevé (promesse n°1) | Contrats §5.6, tests de scénarios avec FakeClock, chaos 3.1 |
| R9 | **Dérive de périmètre** (dashboard trop tôt) | Moyenne / moyen | Dashboard en V2 conditionné à H7 ; modèle de données déjà prêt |
| R10 | **Environnement Windows** (fins de ligne, chemins, perfs Docker) | Moyenne / faible | `.gitattributes`, dépôt dans WSL2, scripts Node plutôt que bash pour le dev |
