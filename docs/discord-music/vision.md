# Bot musique Discord : vision produit

*Rédigé le 2026-10-12 par l'agent Product Vision pour Hugo. Sources : `reponses-fondateur.md` et `recherche-marche-et-legal.md`.*
*Ce document dit QUOI construire et POURQUOI. Il ne fait aucun choix technique et reste valable quelle que soit la source audio retenue (question tranchée à part).*

---

## 1. Pitch

**En une phrase :** un bot musique Discord pour un groupe d'amis, qui joue sans couper, trouve le bon morceau du premier coup et sait enchaîner dans l'ambiance du moment, avec des playlists de serveur gratuites.

**La promesse faite à l'utilisateur :**
> « Lance un son, il joue. Il ne coupe pas, il ne se trompe pas de titre, et quand la file est vide, la musique continue dans le même style. Tout est gratuit, rien n'est réservé au premium. »

**Pourquoi ce projet :** les frustrations de Hugo recoupent les plaintes relevées dans la recherche : coupures et déconnexions, titres introuvables ou mal appariés, aléatoire qui ignore le style, playlists absentes ou payantes. Les gros bots sont saturés aux heures de pointe et mettent l'essentiel derrière un abonnement. Un bot privé, petit et soigné peut faire mieux sur ces quatre points, sans chercher à rivaliser sur le nombre de fonctions.

**Positionnement :** moins de fonctions que Jockie ou Lara, mais celles qui sont là marchent à chaque fois. La fiabilité est la fonctionnalité n°1.

---

## 2. Personas

| Persona | Qui | Moment de déclenchement | Ce qu'il attend | Ce qui le fait partir |
|---|---|---|---|---|
| **Le lanceur de soirée** (Hugo ou un ami « DJ ») | Ouvre le vocal en premier, met l'ambiance | Vendredi soir, session de jeu ou apéro vocal : « mets un peu de son » | Lancer une playlist ou un morceau en 5 secondes, puis ne plus y toucher pendant 2 heures | Le bot qui coupe au milieu d'une partie, ou qu'il faut relancer toutes les 20 minutes |
| **Le contributeur de passage** | Un ami dans le vocal, qui joue à autre chose en même temps | Il entend un morceau, pense à un autre : « attends, je mets un truc » | Ajouter un titre à la file sans casser ce qui joue, trouver le bon titre même avec une recherche approximative | Le mauvais titre (remix, live, reprise) ou un « introuvable » sans explication |
| **Le curateur** | L'ami qui a des goûts marqués et aime partager | En semaine, hors session, il pense à un morceau pour la prochaine soirée | Construire la playlist du serveur (« Soirées », « Chill », « Rap FR ») et voir ce que les autres y ont ajouté | Playlists perso seulement, ou limitées / payantes |

*Futur persona (public, plus tard) :* l'admin d'un petit serveur qui cherche un bot gratuit sans vote forcé ni permission Administrateur. Il n'est pas visé par le MVP.

---

## 3. Métrique North Star

**Heures d'écoute en groupe par semaine** : temps de lecture effective pendant lequel au moins 2 membres humains sont dans le salon vocal avec le bot.

**Pourquoi celle-là :**
- Elle mesure la valeur réelle : de la musique partagée entre amis, pas des commandes tapées.
- Elle intègre la fiabilité : chaque coupure, chaque morceau introuvable, chaque file vide qui s'arrête fait baisser le chiffre ou pousse le groupe vers un autre bot.
- Elle récompense l'autoplay « ambiance » et les playlists, qui prolongent les sessions sans effort.
- Le seuil « au moins 2 humains » évite de compter un bot qui tourne seul dans un salon vide.

**Garde-fous** (suivis à côté, pour ne pas tricher avec la North Star) :
- **Taux de sessions sans incident** (aucune coupure non récupérée, aucune erreur de lecture visible). Cible de fiabilité, pas d'engagement.
- **Taux de « trouvé du premier coup »** (le morceau choisi n'est ni passé ni signalé « mauvais titre » dans les 30 secondes).
- **Taux de skip pendant l'autoplay** (si l'autoplay est skippé en rafale, il ne suit pas le style).

---

## 4. Boucle principale

```
Déclencheur  →  Action  →  Récompense  →  Partage  →  Retour
```

1. **Déclencheur** : des amis se retrouvent en vocal (soirée, jeu). Quelqu'un veut du son.
2. **Action** : il lance une playlist du serveur ou cherche un morceau. Un panneau « en cours » s'affiche avec des boutons.
3. **Récompense** : ça joue immédiatement, le bon titre, sans coupure. Quand la file se vide, l'autoplay « ambiance » enchaîne dans le même style. Personne n'a à s'en occuper.
4. **Partage** : un ami entend un morceau qui lui plaît, il clique « ajouter à la playlist du serveur » ou « j'aime ». Les autres ajoutent leurs titres à la file. La playlist commune grossit.
5. **Retour** : à la soirée suivante, la playlist du serveur est plus riche et l'historique nourrit l'autoplay. Relancer la musique devient le réflexe du groupe.

**Engagement sans dark patterns :** pas de notifications push, pas de série à maintenir, pas de vote forcé, pas de pub, pas de fonctions bloquées. La seule raison de revenir doit être que la musique marche bien. Le bot quitte le salon quand il est seul, sans relance.

---

## 5. Séparer « catalogue » et « lecture »

La vision distingue deux couches, pour rester valable quelle que soit la source audio choisie :

| Couche | Rôle produit | Ce qui en dépend |
|---|---|---|
| **Catalogue / recherche / métadonnées** | Savoir **quel** morceau l'utilisateur veut : titre, artiste, durée, pochette, style/genre, ambiance, artistes proches, paroles | Trouver le bon titre, autoplay qui suit un style, recommandations, paroles synchronisées, playlists (qui stockent des **morceaux**, pas des liens audio) |
| **Lecture audio** | Obtenir un flux jouable pour ce morceau et le diffuser dans le salon sans coupure | Fiabilité, qualité audio, reprise après incident |

**Exigences produit qui en découlent :**
- Une playlist, un historique ou un « j'aime » désigne un **morceau** (identité catalogue), jamais un lien vers une source précise. Si la source de lecture change ou tombe, les playlists restent intactes.
- Si un morceau n'est pas jouable, le bot le dit clairement (« introuvable en lecture, passé au suivant »), essaie une alternative si possible, et ne s'arrête jamais en silence.
- La qualité de l'autoplay et des recommandations dépend surtout du catalogue (métadonnées de style, artistes proches), pas de la source audio.
- Le bouton « mauvais titre ? » sert aux deux couches : il corrige l'appariement et améliore les recherches suivantes du serveur.

---

## 6. Fonctionnalités

### MVP : un serveur d'amis, quelques semaines de travail solo

Objectif : régler les 4 frustrations principales de Hugo (coupures, introuvable, aléatoire sans style, playlists), rien de plus.

| # | Fonctionnalité | Frustration traitée |
|---|---|---|
| 1 | **Lecture de base** : rejoindre le vocal, jouer, pause, reprise, passer, arrêter, file d'attente (voir, retirer, mélanger, vider), boucle titre / file | Socle |
| 2 | **Panneau « en cours » avec boutons** (pause, passer, boucle, j'aime, ajouter à une playlist), mis à jour à chaque titre | Fini les commandes à retenir |
| 3 | **Fiabilité d'abord** : reconnexion automatique avec reprise au bon morceau et à la bonne position, file conservée après un redémarrage, morceau injouable passé avec un message clair | Coupures, bugs de lecture |
| 4 | **Recherche avec choix** : la recherche propose les 3 à 5 meilleurs résultats (titre, artiste, durée) quand elle hésite ; bouton « mauvais titre ? » qui propose les alternatives | Musiques introuvables, mauvais appariement |
| 5 | **Playlists de serveur gratuites et partagées** : créer, ajouter le morceau en cours ou une recherche, jouer, jouer en aléatoire, voir le contenu. Tout le monde ajoute ; le créateur et les admins suppriment | Playlists impossibles ou payantes |
| 6 | **Autoplay « ambiance »** : quand la file se vide, le bot enchaîne sur des morceaux du même style que ce qui vient de jouer, sans répéter les titres récents. Activable / désactivable. Variante : « radio à partir de ce morceau » | Aléatoire qui ne suit pas le style |
| 7 | **Historique du serveur** (ce qui a été joué, par qui, aimé ou passé). Invisible au début, il sert l'autoplay et prépare les recommandations | Prépare la V2 |
| 8 | **FR + EN** : textes du bot dans la langue du serveur, réglable | Exigence de Hugo |
| 9 | **Comportement propre** : permissions minimales, départ automatique quand le bot reste seul, aucune pub, aucun vote | Plaintes marché |

**Hors MVP, assumé :** paroles, dashboard web, recommandations personnalisées, filtres audio, blind test, multi-serveurs.

### V2 : après 3 à 4 semaines d'usage réel sans incident majeur

| Fonctionnalité | Pourquoi en V2 |
|---|---|
| **Paroles synchronisées** dans le panneau « en cours » (avec repli sur paroles simples ou « paroles indisponibles ») | Fonction « waouh » demandée, mais elle ne règle aucune frustration de base. Elle dépend de la couche catalogue et doit respecter les limites de mise à jour des messages Discord, donc à soigner. **Premier élément de la V2.** |
| **Dashboard web personnalisable** : file en direct, playlists (réorganiser, renommer, importer), historique, réglages du serveur, thème | Voir justification ci-dessous |
| **Recommandations auto** : « vous aimerez aussi » pour le serveur, à partir de l'historique et des « j'aime » ; playlist automatique hebdo du serveur | Il faut d'abord quelques semaines d'historique pour qu'elles valent quelque chose |
| **Vote skip et rôle DJ** (optionnels) | Utile dès que le groupe grandit ; inutile à 4 amis |
| **Import de playlists** depuis un lien d'une autre plateforme (métadonnées seulement, rejouées via la source retenue) | Accélère l'adoption, mais dépend des choix de catalogue |
| **Filtres audio** (bass boost, nightcore, vitesse) | Fun, peu coûteux, mais pas prioritaire |

**Pourquoi le dashboard web passe en V2 :**
1. Il double la surface à construire et à sécuriser : connexion Discord sur le web, gestion des droits par serveur, une seconde interface à maintenir, une page exposée sur internet. C'est facilement autant de travail que le bot lui-même.
2. Pour un seul serveur d'amis, les boutons du panneau et les commandes couvrent 90 % des besoins du MVP (lancer, gérer la file, ajouter à une playlist).
3. Ce qui rend le dashboard utile (playlists riches, historique, réglages) n'existe qu'après quelques semaines d'usage. Le construire avant, c'est construire une vitrine vide.
4. Condition posée dès le MVP : playlists, historique et réglages sont rangés par serveur et par morceau, pour que le dashboard puisse s'y brancher sans refonte.

### Plus tard

- **Blind test / quiz musical** du serveur avec classement (fort potentiel entre amis, déjà vu chez Jockie et FredBoat).
- **Radios web** intégrées.
- **Ouverture publique** : plusieurs serveurs, page de présentation, procédure de retrait de contenu, conditions d'utilisation. Au-delà de 100 serveurs, vérification Discord obligatoire avec contrôle d'identité. Suppose que la question des sources soit compatible avec un usage public.
- **Auto-hébergement facile** pour d'autres groupes d'amis.
- **Statistiques du serveur** (« Wrapped » de fin d'année, top morceaux, top contributeurs).
- **Écoute ensemble via une Activity Discord** où chacun écoute avec son propre compte (piste signalée par la recherche, non vérifiée).
- **Premium cosmétique** (si un jour public) : uniquement des extras qui ne touchent pas au contenu. Jamais l'essentiel derrière un paiement.

---

## 7. Nom et ton

**Ton :** complice, court, un peu chaleureux, jamais corporate. Le bot parle comme un ami qui gère le son : « C'est parti », « Je n'ai pas trouvé ce titre, voilà ce qui s'en approche », « Plus personne ? Je m'éclipse ». Pas d'emoji à outrance, pas de majuscules criardes. Mêmes registres en FR et EN.

| Nom | Justification |
|---|---|
| **Platine** | En français, la platine qui joue les disques ; en anglais, « platinum » s'entend dans le mot et évoque le disque de platine. Court, image claire du « DJ du serveur ». |
| **Refrain** | Mot identique en français et en anglais, sens identique. Évoque le morceau qu'on reprend ensemble et l'idée de revenir. |
| **Ambiance** | Mot identique en français et en anglais. Dit exactement la promesse différenciante : l'autoplay qui garde l'ambiance. Un peu plus long et plus générique, donc plus dur à protéger. |

⚠️ **Disponibilité à vérifier** avant de choisir : nom d'application Discord, présence sur top.gg et autres listes de bots (homonymes), nom de domaine, comptes GitHub, et marques existantes (INPI / EUIPO) si le projet devient public.

---

## 8. Hypothèses risquées et tests rapides

| # | Hypothèse | Pourquoi c'est risqué | Test rapide (avant ou pendant le MVP) | Critère de validation |
|---|---|---|---|---|
| H1 | Le groupe d'amis abandonnera son bot actuel pour celui-ci | L'habitude est forte ; un bot « suffisant » existe déjà | Faire tourner le MVP en parallèle de l'ancien bot sur 2 semaines de soirées | Plus de la moitié des heures d'écoute en groupe passent par le nouveau bot en semaine 2 |
| H2 | On peut atteindre une fiabilité clairement meilleure que les bots actuels | Les coupures viennent souvent de causes externes (source audio, voix Discord) | Journal d'incidents dès le premier jour ; test de stress : redémarrage forcé et coupure réseau pendant une lecture | 95 % des sessions sans incident visible ; reprise automatique au bon endroit dans 100 % des redémarrages testés |
| H3 | L'autoplay peut suivre un style de façon convaincante avec des métadonnées accessibles | Le « style » est subjectif, les métadonnées de genre sont souvent pauvres | Avant de coder : à la main, partir de 5 morceaux typiques du groupe et produire les 10 suivants avec les métadonnées disponibles ; faire noter les enchaînements par 3 amis | Au moins 7 morceaux sur 10 jugés « dans le style » ; puis en usage, moins de 30 % de skip pendant l'autoplay |
| H4 | La recherche trouve le bon titre du premier coup dans la grande majorité des cas | Remix, live, reprises, titres FR peu connus | Récupérer 50 recherches réelles du groupe (historique de l'ancien bot ou liste dictée) et les rejouer | 85 % de bons titres en premier résultat, 95 % dans les 5 premiers |
| H5 | Les playlists de serveur seront réellement alimentées par plusieurs personnes | Souvent, une seule personne fait tout | Après 3 semaines, compter les contributeurs distincts | Au moins 3 contributeurs distincts et 2 playlists de plus de 20 titres |
| H6 | Les paroles synchronisées seront regardées et pas jugées gênantes | Un panneau qui bouge en permanence peut agacer dans un salon textuel actif | Maquette : afficher les paroles sur 3 soirées, avec un bouton pour les masquer | Moins de la moitié des sessions où elles sont masquées ; retours spontanés positifs |
| H7 | Le dashboard web vaut l'effort | Le groupe peut très bien se contenter des boutons Discord | En fin de MVP, lister les actions que les amis ont trouvées pénibles dans Discord | Au moins 3 actions récurrentes qui seraient nettement plus simples sur une page web |

---

## 9. Questions au fondateur

*La question des sources audio est déjà posée séparément ; elle n'est pas reprise ici.*

1. **Qui contrôle la lecture ?** Tout le monde peut passer, vider la file, changer de playlist, ou seulement certains rôles ?
   *Défaut proposé : tout le monde a tous les droits (serveur d'amis) ; vote skip et rôle DJ en option V2.*
2. **Autoplay activé par défaut ?** Quand la file se vide, le bot continue dans le style ou s'arrête ?
   *Défaut proposé : activé par défaut, désactivable par une commande ou un bouton.*
3. **Quand le bot quitte-t-il le vocal ?** Reste-t-il en permanence (24/7) ou part-il quand il est seul ?
   *Défaut proposé : il part après 5 minutes seul, et la file est conservée pour la reprise.*
4. **Qui peut modifier ou supprimer une playlist du serveur ?**
   *Défaut proposé : tout le monde ajoute ; seul le créateur ou un admin retire des titres ou supprime la playlist.*
5. **Combien de temps garder l'historique d'écoute ?** (il nourrit l'autoplay et les recommandations, et contient qui a joué quoi)
   *Défaut proposé : 12 mois, sans donnée autre que l'identifiant Discord et le morceau ; une commande pour l'effacer.*
6. **Quel nom ?** Platine, Refrain, Ambiance, ou autre ?
   *Défaut proposé : Platine, sous réserve de disponibilité.*
7. **Code ouvert ou privé ?**
   *Défaut proposé : dépôt privé pendant le MVP, décision d'ouverture au moment d'un éventuel passage en public.*
