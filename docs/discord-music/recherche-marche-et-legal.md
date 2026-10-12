# Bot musique Discord : recherche marché, légal et technique

*Rapport du 2026-10-12, rédigé par l'agent Chercheur pour Hugo. Avant la spec.*
*Convention : **[V]** = fait vérifié dans une source primaire ou recoupé par au moins deux sources. **[S]** = une seule source secondaire (blog, comparatif). **[D]** = déduction de ma part.*

---

## Résumé

1. Le marché est dominé par **Jockie Music**, environ 10 M de serveurs selon top.gg. Restent actifs aussi FredBoat, Lara, Chip, Green-bot et Vexera. Groovy et Rythm (2021) puis Hydra (2023) ont arrêté la musique après des mises en demeure de Google/YouTube. **[V]**
2. **Rythm est revenu en juin 2024** sous forme de Discord Activity, avec un catalogue sous licence, une offre gratuite et un Premium à 4,99 $/mois. Le contournement légal passe donc par des **licences**, pas par la technique. **[V]**
3. Les CGU de YouTube interdisent explicitement de télécharger ou diffuser le contenu, l'accès automatisé et le « streaming de musique depuis le Service » (Restrictions 1, 3 et 9). Sur le plan technique, YouTube casse régulièrement les extracteurs (PO tokens, ciphers, blocage d'IP, OAuth à risque). **[V]**
4. Les Developer Policy de Spotify, Deezer et SoundCloud interdisent le rebroadcast, le cache et l'agrégation. Spotify, Apple Music et Tidal ne servent qu'aux **métadonnées**, et LavaSrc les « mirrore » vers YouTube par défaut. **[V]**
5. **DAVE** (chiffrement E2EE de la voix Discord) est **obligatoire depuis le 1er/2 mars 2026** pour tous les clients, bots compris. Les briques compatibles sont Lavalink ≥ 4.2.0, @discordjs/voice avec @snazzah/davey (Node ≥ 22.12), discord.py ≥ 2.7, Songbird ≥ 0.6.0 et NodeLink. **[V]**
6. **Vérification Discord obligatoire au-delà de 100 serveurs**, avec contrôle d'identité via Stripe. **[V]**
7. Les plaintes récurrentes portent sur YouTube qui casse, les fonctions passées en premium (24/7, file, qualité audio), le lag aux heures de pointe, l'absence de dashboard et la qualité audio plafonnée. **[S]**
8. Sources à faible risque : les fichiers uploadés par les utilisateurs (avec réserve), Jamendo en usage non commercial, Audius, les flux de radios web officiels et la musique libre ou CC. À risque élevé : YouTube et tout « rip » Spotify/Deezer. **[V]/[D]**
9. **Bot privé** sur un seul serveur d'amis : risque d'action très faible en pratique, mais pas nul juridiquement. **Bot public** listé : YouTube est exclu, sauf à accepter une fermeture probable. **[D]**
10. Différenciations crédibles : fiabilité (multi-sources, reprise automatique), tout gratuit sans pub, blind test, paroles synchronisées (LRCLIB), dashboard web, playlists de serveur, votes et radios. **[D]**

---

## 1. Concurrents

| Bot | État (oct. 2026) | Sources audio annoncées | Fonctions phares | Modèle éco. | Fiabilité de l'info |
|---|---|---|---|---|---|
| **Jockie Music** | Actif. 10 024 963 serveurs, note 4,63/5 sur 304 avis (top.gg) | Spotify, Tidal, Deezer, Apple Music, liens HTTP, pièces jointes Discord, Vimeo, Bandcamp, Mixcloud, plus de 37 000 radios. **YouTube absent** de la fiche top.gg | Jusqu'à 4 bots gratuits en parallèle (23 de plus en premium), plus de 150 commandes, collections partagées par code, « guess the song », dashboard de stats | Gratuit + Patreon. Prix annoncé de 6,99 $/mois [S, une seule source] | [V] top.gg + site officiel |
| **FredBoat** | Actif selon son site, qui annonce « 5 millions de serveurs » | YouTube, SoundCloud, Twitch, Bandcamp, liens directs (site officiel). Pas de lecture Spotify, seulement les liens | Simple, open source, auto-hébergeable (Java), « Music Room » avec jeux | Gratuit selon le site. Un comparatif cite une option payante à 3 $/serveur/mois [S] | [V] site / [S] prix |
| **Lara** | Actif. ~357 000 serveurs (discordbotlist) | Spotify, SoundCloud, Deezer, Facebook, URL, Twitch | 24/7, playlists perso, boutons et menus, filtres (bass boost, nightcore), autoplay, stages | Freemium. Premium à 3,99 $/mois [S]. Certaines commandes exigent un vote | [V] discordbotlist / [S] prix |
| **Chip** | Actif [S] | YouTube, Spotify (recherche), SoundCloud, Twitch [S] | Commandes simples, 320 kbps en premium, file de 500 titres en premium | Freemium. Premium à 4,99 ou 5,99 $/mois selon la source. 24/7 premium | [S] (sources divergentes) |
| **Green-bot** | Actif. Revendique 1,5 M de serveurs (site) | Spotify, YouTube, « plus » | 24/7, « haute qualité », gratuit | Non indiqué | [S] (site marketing) |
| **Vexera** | Statut incertain | YouTube, SoundCloud, Spotify (recherche) [S] | Bot multi-usage (modération, accueil, niveaux) | Premium à 4,99 $/mois [S] | [S] faible |
| **LoFi Girl bot** | Actif | Flux officiels LoFi Girl uniquement | 24/7, stations thématiques, aucune requête libre | Gratuit | [S] |
| **Rythm (v2)** | Actif depuis juin 2024. **Discord Activity** (iframe), plus des apps en développement | Catalogue **sous licence** (~50 M de titres en 2024), revenus partagés avec labels et éditeurs | Écoute de groupe jusqu'à 20 personnes, UI complète, visualiseur, radios curatées | Gratuit + Premium à **4,99 $/mois**. Un premium peut partager une playlist à 20 personnes | [V] MusicTech + Wikipedia |
| **Groovy** | **Fermé le 30/08/2021** après une mise en demeure de YouTube | YouTube | (historique) | Premium | [V] |
| **Rythm (v1)** | **Fermé le 15/09/2021** après une mise en demeure de Google (CGU YouTube, usage commercial) | YouTube | ~560 M d'utilisateurs revendiqués | Premium | [V] Wikipedia + presse |
| **Hydra** | **Musique supprimée**. Annonce de fin de la lecture YouTube le 13/09/2021, mise en demeure YouTube le 31/01/2023, retrait total de la musique le 07/02/2023. Le bot reste actif comme bot utilitaire (niveaux, rôles, dashboard) | Aucune | (historique) | Premium (fonctions non musicales) | [V] site hydra.bot (plus de musique) + 2 articles concordants sur les dates. Un 3e article date la fin au 24/09/2021 → voir Incertitudes |

**Histoire Groovy/Rythm, en bref [V] :** en août et septembre 2021, Google/YouTube envoie des mises en demeure à Groovy puis à Rythm. Le reproche : lecture de YouTube hors du lecteur officiel et monétisation (premium) sans licence. Les deux bots ferment. Rythm lève des fonds, négocie des licences avec les labels et revient en 2024 sous forme d'**Activity Discord** sous licence. Leçon : la seule voie « grand public durable » observée passe par les licences, avec un vrai coût (partage de revenus avec les ayants droit).

**Tendance [D] :** les gros bots qui ont survécu (Jockie) ont **retiré YouTube** de leur offre affichée. Ils misent sur les métadonnées Spotify, Apple et Deezer, résolues vers d'autres sources : Deezer, SoundCloud, Bandcamp, radios, fichiers. Les bots qui affichent encore YouTube sont surtout des bots moyens ou petits.

---

## 2. Plaintes récurrentes = opportunités

*Limite : Reddit est inaccessible depuis mon environnement (bloqué) et le texte des avis top.gg n'était pas récupérable. Les plaintes ci-dessous viennent de comparatifs 2026 (PeakBot, VibeBot, CommunityOne, souvent écrits par des concurrents, donc biaisés) et des tickets GitHub de bots open source. À recouper par Hugo sur r/discordapp et r/Discord_Bots avant la spec.*

| Plainte | Indices | Opportunité |
|---|---|---|
| **YouTube ne marche plus / « Sign in to confirm you're not a bot » / 403** | Tickets jagrosh/MusicBot : #1588, #1829 « Lavalink no longer supports YouTube! », #1833 « yt-dlp for YouTube ». Le README youtube-source qualifie le client ANDROID de « frequently dysfunctional » [V] | Ne pas dépendre de YouTube. Chaîne de repli multi-sources et message clair quand une piste n'est pas jouable |
| **Coupures, bot qui se déconnecte et rejoint, silence** | jagrosh #1847 « Bot keeps Disconnecting and Rejoin », #1841 « not giving sounds ». Tickets de chiffrement #1851 et #1843, liés à la migration des modes de chiffrement et à DAVE [V] | Reconnexion automatique avec reprise à la position exacte, file persistante, supervision |
| **Fonctions passées en premium** | 24/7, file de plus de 50 titres, 256/320 kbps, filtres : premium chez Chip, Lara, Beatra et (selon les sources) FredBoat [S] | Version gratuite généreuse pour les petits serveurs d'amis. Le premium porte sur des extras, pas sur l'essentiel |
| **Lag aux heures de pointe, instance publique saturée** | Jockie et FredBoat cités pour leur lenteur en pointe [S] | Petit périmètre et hébergement dédié, ou auto-hébergement facile (Docker) |
| **Pas de dashboard, interface datée, commandes seulement** | Jockie et FredBoat « no dashboard » [S] | Panneau « now playing » avec boutons, plus un mini dashboard web (file, historique, playlists) |
| **Gestion de playlists limitée, file non persistante** | « limited playlist management » (Jockie), la persistance de file est un argument premium [S] | Playlists de serveur collaboratives et file sauvegardée |
| **Mauvais appariement des titres** (Spotify vers une autre source) | Lara : « matching errors with obscure tracks » [S]. LavaSrc mirrore par ISRC puis par requête texte [V] | Appariement par ISRC + durée + score, avec bouton « mauvais titre ? » |
| **Votes obligatoires sur top.gg et pubs** | Lara exige des votes pour certaines commandes [S] | Aucun vote forcé, aucune pub |
| **Permissions excessives** | Lara demande Administrator (permissions=8) [V] | Permissions minimales (Connect, Speak, Send Messages) |

---

## 3. Légal et ToS

### 3.1 Principes

- **Discord Developer Policy** (en vigueur au 8 juillet 2024) : aucune clause spécifique à la musique ni au droit d'auteur. Elle impose de respecter « all applicable laws » et les conditions des API tierces [V]. Le risque ne vient donc pas de Discord mais des **ayants droit** et des plateformes sources, Google en tête [D].
- **Vérification :** au-delà de **100 serveurs**, l'app doit être vérifiée (onglet App Verification, identité du propriétaire via Stripe, revérifications possibles) [V]. Le seuil de 75 serveurs n'apparaît pas dans la page officielle actuelle. Il ouvrait autrefois la demande de vérification [D, ancien fonctionnement, non revérifié].
- **DAVE** a aussi une conséquence technique pour un bot qui lit ou enregistre l'audio, mais ce n'est pas notre cas (voir §4).

### 3.2 Source par source

| Source | Ce que disent les conditions (primaire) | Risque bot **public** | Risque bot **privé** (1 serveur d'amis) |
|---|---|---|---|
| **YouTube / YouTube Music** | CGU : interdit d'« access, reproduce, download, distribute, transmit, broadcast… » sans autorisation (R1), d'accéder « using any automated means » (R3), d'écouter autrement que pour un « personal, non-commercial use », avec mention explicite de l'interdiction de « streaming music from the Service » (R9). Techniquement : PO tokens exigés sur la plupart des clients, sinon 403 ou **blocage du compte ou de l'IP** (guide yt-dlp). OAuth via youtube-source : « could get your account terminated », d'où l'usage de comptes jetables. Précédents : Groovy, Rythm, Hydra [V] | **Élevé** (mise en demeure probable dès que le bot devient visible + casse technique permanente) | **Moyen** : violation des CGU quand même, mais faible probabilité d'action. Risque réel = compte Google banni et maintenance fréquente [D] |
| **Spotify** | Developer Policy (en vigueur au 15/05/2025) : streaming réservé aux abonnés via les SDK officiels, pas de webcasting non interactif (III.4), pas de diffusion publique (III.10), pas de « bot, script or automated process » de manipulation (II.2), pas d'imitation de l'expérience Spotify (III.11), métadonnées et pochettes **avec lien retour** et uniquement « in connection with the underlying content » (II.4, II.5) [V] | Audio : **élevé** (impossible légalement). Métadonnées pour recherche ou affichage : **moyen** (zone grise : II.5 lie les métadonnées au contenu Spotify, alors qu'un bot les utilise pour jouer autre chose) [D] | Audio : élevé. Métadonnées : faible |
| **Deezer** | Guidelines API : extraits de 30 s pour tous, titres complets seulement pour les utilisateurs Premium via SDK/plugins. Les URL de titres complets ne doivent jamais être exposées. Stockage local interdit. Blocage possible à tout moment [V] | **Élevé** pour l'audio complet (la lecture directe de LavaSrc Deezer exige une clé/ARL d'utilisateur, ce qui revient à contourner le service) [D]. Faible pour les extraits de 30 s (adaptés à un blind test) | Moyen |
| **Apple Music** | Lecture complète via MusicKit, sur l'appareil de l'utilisateur abonné. Rien n'autorise un rebroadcast dans un salon vocal [D, doc Apple non lue en entier]. LavaSrc ne fait que mirrorer Apple Music [V] | Métadonnées seulement : faible à moyen. Audio : élevé | idem |
| **SoundCloud API** | Conditions API (30/03/2024) : créditer l'uploader et SoundCloud avec backlinks, pas de téléchargement, cache ni offline, **pas de pub autour du contenu**, pas d'achats in-app donnant accès à du contenu SoundCloud, **interdiction d'agréger et streamer le contenu de plusieurs utilisateurs en service à la demande** sauf licence. Les uploaders peuvent faire respecter les conditions. Accès API sur demande [V] | **Moyen** : jouer via l'API avec attribution est plus défendable que YouTube, mais un bot « joue n'importe quel titre » ressemble à l'agrégation interdite, et le premium est risqué [D] | Faible à moyen |
| **Bandcamp** | CGU : service pour un « personal, non-commercial use ». Reproduction ou stockage à d'autres fins interdits sans autorisation écrite. Pas d'API publique de streaming [V] | **Moyen** (scraping d'une page web) | Faible (surtout pour des achats perso) |
| **Jamendo** | API libre pour un usage **non commercial**. Commercial (pubs, affiliation, premium compris) : contacter licensing@jamendo.com. Attribution obligatoire (artiste + Jamendo + lien), pas de cache offline, respect de la licence CC de chaque titre [V] | **Faible** sans monétisation, **moyen** si premium (licence à négocier) | Faible |
| **Audius** | API REST publique « stream tracks ». Conditions, quotas et attribution non vérifiés dans la doc (page incomplète) [V partiel] | Faible à moyen (à vérifier) | Faible |
| **Free Music Archive / musique CC** | Non vérifié dans cette passe. Licences CC par titre, NC fréquent [D] | Faible si la licence de chaque titre est respectée | Faible |
| **Radios web** (flux officiels) | Relayer le flux public d'une radio = rediffusion. Côté radio, la licence SACEM webradio exclut la sélection à la demande. Relayer le flux tel quel reste une zone grise (les radios tolèrent souvent, sans autorisation formelle) [D]. Jockie le fait à grande échelle (37 000 stations) [V] | **Faible à moyen** | Faible |
| **Fichiers uploadés par les utilisateurs** (pièces jointes Discord, liens directs) | La responsabilité porte d'abord sur l'utilisateur. Le bot reste un outil de diffusion. Prévoir des CGU et une procédure de retrait [D] | **Faible à moyen** | **Faible** |
| **Licence propre type Rythm v2** | Accords labels et éditeurs, partage des revenus [V] | Faible juridiquement, mais **hors de portée d'un dev solo** (levée de fonds, négociations sur des années) [D] | n/a |
| **« Écoute ensemble » natif** (Spotify Listen Along, Apple Music, Activities) | Listen Along : Premium requis côté hôte, **pas disponible en vocal** (support Spotify) [V]. Une Activity Discord est une app web en iframe, chaque client exécute sa propre page [V]. On peut donc bâtir une Activity où **chaque participant lit l'audio de son côté avec son propre compte** (via les SDK officiels) [D, à valider par une lecture des ToS de chaque SDK] | Faible (pas de rebroadcast) | Faible |

### 3.3 Privé vs public [D, à faire valider par un juriste si projet commercial]

- **Droit français :** l'exception de « représentation privée et gratuite effectuée exclusivement dans un cercle de famille » (art. L122-5 1° CPI) est interprétée de façon stricte par les tribunaux (famille et amis très proches). Un serveur Discord de quelques amis s'en approche, mais rien ne garantit qu'un juge l'admettrait. Cela ne lève **pas** non plus la violation contractuelle des CGU YouTube et Spotify, qui reste un sujet séparé (risque de bannissement de compte).
- **Bot privé** (non listé, invitation désactivée, 1 serveur, aucune monétisation) : probabilité d'action très faible, coût maximal estimé = compte Google ou IP bloqués et bot cassé. Les licences de l'auteur et les CGU s'appliquent en théorie.
- **Bot public** (listé sur top.gg, vérifié au-delà de 100 serveurs, premium) : profil exact de Groovy, Rythm et Hydra. **Monétiser un bot YouTube = risque élevé.** Recommandation : en public, n'utiliser que des sources à risque faible ou moyen et ne monétiser que des fonctions qui ne touchent pas au contenu (dashboard, stats, cosmétique), en relisant la clause anti-pub de SoundCloud.
- **Webradio SACEM** (licence valable France/Luxembourg/Monaco, 80 € HT/an pour une petite radio) : ne couvre **pas** la sélection à la demande, donc **pas** un bot où les utilisateurs choisissent les morceaux. Elle ne couvre pas non plus les droits voisins des producteurs [V]. Ce n'est donc pas une solution pour un bot à requêtes.

---

## 4. Briques techniques

### 4.1 DAVE (point critique) [V]
- DAVE est le protocole E2EE audio et vidéo de Discord : introduit en septembre 2024, étendu aux bots et au Social SDK en 2025, migration terminée début mars 2026, audité par Trail of Bits.
- Le **support Discord** indique : *« Third-party applications and bots that connect to Discord voice must implement DAVE support… to continue functioning after March 2, 2026 »*. La doc voix parle du « March 1st, 2026 ». Un client déclare son support via `max_dave_protocol_version` dans l'Identify. La librairie de référence est **libdave** (open source).
- **Impact pour nous :** toute la pile voix doit être compatible DAVE, sinon le bot ne peut plus rejoindre un salon. Envoyer de l'audio reste possible (le bot chiffre ses frames). Les bots qui n'ont pas été mis à jour sont cassés : c'est la cause probable de nombreux tickets « encryption modes not supported » [D].

| Brique | Version compatible DAVE | Notes |
|---|---|---|
| **Lavalink v4** (Java) | **4.2.0** (28 fév.), puis 4.2.1 (libdave 0.1.0) et 4.2.2 (6 mars, dernière) | Le client (librairie côté bot) doit aussi être à jour. Image Docker en Java 17. La source YouTube intégrée est dépréciée depuis 4.0.5 : passer par le plugin youtube-source |
| **youtube-source** (plugin) | 1.18.x (dernière : 1.18.2, 27 juil.) | Rotation fréquente des clients (TVHTML5EMBEDDED supprimé, remplacé par TVHTML5_SIMPLY), serveur de cipher distant (yt-cipher), OAuth à risque, poToken. Maintenance lourde |
| **LavaSrc** (plugin) | Lavalink v4 | Lecture **directe** : Deezer, Yandex, VK, Qobuz, JioSaavn, yt-dlp, Flowery TTS. **Mirror** (métadonnées, puis lecture ailleurs) : Spotify, Apple Music, Tidal. Ordre par défaut des providers : `ytsearch:"%ISRC%"` puis `ytsearch:%QUERY%`, configurable |
| **LavaSearch / LavaLyrics** | Plugins officiels de recherche avancée et de paroles | Paroles (via LavaSrc) : Spotify, Deezer, Yandex, VK, YouTube Music, **LRCLIB** |
| **NodeLink** | Supporte DAVE nativement | Alternative en Node.js compatible API Lavalink v4. Lyrics et chapitres intégrés, **sans plugins** |
| **discord.js + @discordjs/voice** | 0.19.x avec **@snazzah/davey** préinstallé | **Node ≥ 22.12**. Opus (`@discordjs/opus` ou `opusscript`) + FFmpeg |
| **discord.py** | **2.7.0** (DAVE), 2.7.1 alerte si `davey` est absent | **wavelink 3.5.2** (23/04/2026) pour Lavalink v4 |
| **Rust : serenity + Songbird** | **Songbird 0.6.0 « Hoopoe » (05/04/2026)** ajoute DAVE. Dernière version 0.6.1 (10/10/2026) | Très léger en RAM. Un client Lavalink n'est pas nécessaire si on décode soi-même |

**Recommandation d'architecture [D] :** TypeScript avec discord.js et un client Lavalink v4 (ou NodeLink pour rester 100 % Node). Les sources sont branchées comme des « résolveurs » interchangeables : on peut couper YouTube par configuration (privé ou public) sans toucher au reste.

### 4.2 Hébergement [V pour les prix, D pour le dimensionnement]
- Dimensionnement : un petit bot avec Lavalink tient sur 2 vCPU et 2 à 4 Go de RAM (la JVM de Lavalink est la plus gourmande). NodeLink ou Songbird réduisent ce besoin.
- Prix relevés en **août 2026** (HT, IPv4 incluse) : Netcup VPS nano 2 Go **2,59 €/mois** (engagement 6 mois), Netcup VPS 500 4 Go **4,97 €**, Hetzner CX23 4 Go **5,99 €**, CAX11 (ARM) **6,49 €**. Ces petites offres Hetzner sont actuellement affichées « Currently not available », et Hetzner a augmenté ses prix en avril et juin 2026. L'alternative CPX22 est à 19,99 € selon findstack. DigitalOcean et Linode 4 Go : ~21 €.
- **Budget réaliste : 3 à 8 €/mois** pour un bot privé ou petit. 0 € possible en auto-hébergement (Raspberry Pi ou PC). **Attention :** les IP de datacenter sont plus souvent bloquées par YouTube (constat répandu dans les communautés, non confirmé par le guide yt-dlp, voir Incertitudes).

---

## 5. Idées de différenciation (crédibles pour un dev solo)

| Idée | Déjà vue chez | Faisabilité solo | Intérêt |
|---|---|---|---|
| **Blind test / quiz musical** (extraits de 30 s Deezer, légaux via l'API, classement serveur) | Jockie (« guess the song »), FredBoat (jeux dans la Music Room) | Bonne | **Fort** pour un groupe d'amis, faible risque légal (previews officielles) |
| **Paroles synchronisées** dans un embed rafraîchi (LRCLIB) | LavaLyrics, NodeLink | Bonne | Moyen à fort (attention au rate limit des edits Discord) |
| **Panneau « now playing » à boutons** (pause, skip, boucle, vote skip, j'aime) | Lara (boutons et menus) | Très bonne | Standard attendu |
| **Vote skip / DJ role / file démocratique** | Hydra (DJ role, historique) | Bonne | Moyen |
| **Playlists de serveur collaboratives + historique « ce qu'on a écouté »** | Jockie (collections avec code) | Bonne | Fort pour des amis |
| **Autoplay / radio par recommandations** (artistes liés via des métadonnées publiques, Last.fm ou MusicBrainz) | Jockie `m!radio`, Lara | Moyenne | Fort |
| **Reprise sans couture** (file persistante, reconnexion automatique à la position exacte après crash ou redémarrage) | Argument premium chez Jockie | Moyenne | **Fort** : répond à la plainte n°2 |
| **Filtres audio** (bass boost, nightcore, 8D, vitesse) | Lara, Beatra | Très bonne (filtres Lavalink natifs) | Moyen |
| **24/7 gratuit** | Premium chez la plupart | Triviale techniquement, mais coûte en ressources | Fort, argument marketing |
| **Dashboard web léger** (file, playlists, stats) | Hydra (feu), Jockie (stats) | Moyenne | Moyen à fort |
| **Radios web intégrées** (Radio Browser : catalogue communautaire) | Jockie (37 000 stations), LoFi Girl | Bonne | Moyen, faible risque |
| **Discord Activity « écoute ensemble »** où chacun écoute avec son propre compte Spotify ou Apple (synchronisation des positions) | Rythm v2 (Activity sous licence) | Faible à moyenne (SDK web, auth de chaque utilisateur, ToS à vérifier) | Fort, et la seule voie « mainstream » légalement propre sans licence. **À creuser** |
| **Mode privé « self-host en 1 commande »** (Docker compose) | FredBoat (self-host Java, réputé difficile) | Bonne | Fort pour les devs. Transfère le risque YouTube à l'utilisateur [D] |

---

## 6. Sources

**Primaires (conditions, docs officielles)**
- YouTube Terms of Service : https://www.youtube.com/terms
- Spotify Developer Policy (en vigueur au 15/05/2025) : https://developer.spotify.com/policy/
- Deezer API guidelines : https://developers.deezer.com/guidelines
- SoundCloud API Terms of Use (30/03/2024) : https://developers.soundcloud.com/docs/api/terms-of-use
- Jamendo API Terms of Use : https://devportal.jamendo.com/api_terms_of_use
- Bandcamp Terms of Use : https://bandcamp.com/terms_of_use
- Audius REST API : https://docs.audius.co/developers/guides/rest-api
- Discord Developer Policy : https://support-dev.discord.com/hc/en-us/articles/8563934450327-Discord-Developer-Policy
- Discord, vérification des apps : https://support-dev.discord.com/hc/en-us/articles/23926564536471-How-Do-I-Get-My-App-Verified
- Discord, voice connections (DAVE, `max_dave_protocol_version`) : https://docs.discord.com/developers/topics/voice-connections
- Discord Support, FAQ E2EE (deadline du 2 mars 2026, bots) : https://support.discord.com/hc/en-us/articles/38025123604631/
- Discord Blog, E2EE pour tous (18/05/2026) : https://discord.com/blog/every-voice-and-video-call-on-discord-is-now-end-to-end-encrypted
- Discord Activities : https://docs.discord.com/developers/activities/how-activities-work.md
- Spotify Support, Discord et Listen Along : https://support.spotify.com/lv-en/article/discord-and-spotify
- SACEM Luxembourg, licences webradio : https://www.sacem.lu/en/music-users/internet-multimedia
- yt-dlp, PO Token Guide : https://github.com/yt-dlp/yt-dlp/wiki/PO-Token-Guide
- Lavalink releases : https://github.com/lavalink-devs/Lavalink/releases et changelog https://mintlify.wiki/lavalink-devs/Lavalink/changelog/v4
- youtube-source (README + releases) : https://github.com/lavalink-devs/youtube-source et https://github.com/lavalink-devs/youtube-source/releases
- LavaSrc : https://github.com/topi314/LavaSrc
- NodeLink vs Lavalink : https://nodelink.js.org/docs/differences
- @discordjs/voice README 0.19.2 : https://cdn.jsdelivr.net/npm/@discordjs/voice@0.19.2/README.md et guide https://discordjs.guide/voice
- discord.py, What's new : https://discordpy.readthedocs.io/en/latest/whats_new.html
- wavelink (PyPI) : https://pypi.org/project/wavelink/
- Songbird CHANGELOG : https://raw.githubusercontent.com/serenity-rs/songbird/current/CHANGELOG.md
- jagrosh/MusicBot issues (plaintes techniques) : https://github.com/jagrosh/MusicBot/issues

**Fiches et sites des bots**
- Jockie Music (top.gg) : https://top.gg/bot/411916947773587456 et https://www.jockiemusic.com/
- FredBoat : https://fredboat.com/
- Green-bot : https://green-bot.app/
- Lara (discordbotlist) : https://discordbotlist.com/bots/lara
- Hydra : https://hydra.bot/

**Secondaires (presse, comparatifs : biais possibles)**
- Wikipedia, Rythm : https://en.wikipedia.org/wiki/Rythm_(software)
- MusicTech, retour de Rythm (12/06/2024) : https://musictech.com/news/industry/rythm-music-bot-returns-discord/
- AlternativeTo, Rythm Activity : https://alternativeto.net/news/2024/6/rythm-returns-popular-discord-music-bot-relaunches-as-activity-app-with-new-features/
- PC Gamer, Groovy : https://pcgamer.com/google-has-killed-discords-best-music-bot
- CommunityOne, comparatif 2026 : https://blog.communityone.io/best-discord-music-bots/
- VibeBot, comparatif 2026 (dates Hydra 2023) : https://www.vibebot.gg/blog/best-discord-music-bots-2026
- PeakBot, comparatif 2026 (écrit par un concurrent) : https://peakbot.pro/blog/best-discord-music-bots-2026
- BestForAndroid, Hydra : https://bestforandroid.com/hydra-music-bot-discord-commands.md
- Shulex/VOC.AI, YouTube vs bots (06/05/2024) : https://www.shulex.com/blog/is-youtube-killing-every-discord-music-bot%3f-en-us
- Comparatif de prix cloud (août 2026) : https://kimmo.suominen.com/stuff/cpc-2026-08.txt
- Hausse des prix Hetzner 2026 : https://findstack.com/resources/hetzner-price-increase-2026
- Beatra, causes de panne (15/03/2026) : https://beatra.app/blogs/fix-discord-music-bot-not-playing-audio

---

## 7. Incertitudes

1. **Plaintes utilisateurs :** Reddit est bloqué depuis mon environnement et le texte des avis top.gg n'était pas lisible. La section 2 repose sur des comparatifs, souvent écrits par des concurrents, et sur des tickets GitHub. **À recouper manuellement.**
2. **Prix des premium** (Jockie 6,99 $, Chip 4,99 ou 5,99 $, Lara 3,99 $, Vexera 4,99 $, FredBoat 3 $) : une seule source secondaire chacun, parfois contradictoire. Ni le site Jockie ni top.gg ne donnent de prix.
3. **Hydra :** dates divergentes. Deux sources concordantes donnent fin de YouTube en septembre 2021, mise en demeure le 31/01/2023 et retrait de la musique le 07/02/2023 ; une autre donne une fermeture le 24/09/2021. Seul fait vérifié : hydra.bot ne propose plus de musique en 2026.
4. **Sources par bot :** les comparatifs se contredisent (ex. YouTube chez Jockie). J'ai privilégié top.gg (Jockie, sans YouTube) et les sites officiels.
5. **Vexera et Green-bot :** statut réel et fonctionnalités peu documentés. Les chiffres de serveurs sont auto-déclarés.
6. **Seuil de 75 serveurs :** absent de la page officielle actuelle, qui ne mentionne que 100.
7. **IP de datacenter bloquées par YouTube :** constat communautaire répandu, mais non confirmé par le guide yt-dlp consulté.
8. **Apple MusicKit, Free Music Archive, Audius :** conditions non lues en entier. Classement du risque provisoire.
9. **Activities « chacun écoute avec son compte » :** faisabilité et conformité aux ToS Spotify et Apple (SDK web dans une iframe Discord) **non vérifiées**. C'est une piste, pas une conclusion.
10. **Juridique (L122-5, cercle de famille appliqué à Discord) :** déduction, pas d'avis de juriste ni de jurisprudence trouvée sur Discord.
11. **Pas de nouvelle mise en demeure YouTube identifiée entre 2024 et 2026**, mais l'absence de preuve ne prouve rien. La pression actuelle de YouTube semble surtout **technique** (PO tokens, ciphers).
