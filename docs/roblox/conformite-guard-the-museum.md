# Conformité Roblox : « Guard the Museum »

Vérification réalisée par l'agent `verificateur-conformite-roblox` le 2026-10-08. Toutes les pages ont été consultées ce jour-là.

**Concept vérifié.** Horreur légère en coop à 4 dans un musée hanté, avec jump scares et statues qui bougent, sans gore. Public : 10-17 ans et jeunes adultes. Objectif : le catalogue Select (9-15 ans), voire Kids (5-8 ans). Monétisation prévue : game passes cosmétiques, carnet de saison, emotes, revive payant à 25 R$, serveurs privés. Les caisses aléatoires payantes sont exclues. Une rediffusion de fin de nuit doit pouvoir être exportée vers TikTok.

**Conventions.**
- **Fait :** ce que dit la source officielle, cité ou paraphrasé au plus près.
- **Interprétation :** l'application au concept, propre à cette vérification.
- **Verdicts :** ✅ conforme, ⚠️ à adapter, ⛔ bloquant, ❓ source muette.
- **Pages sans date :** quand une page n'affiche pas de date de mise à jour, seule la date de consultation (2026-10-08) est indiquée.

---

## 1. Checklist

### A. Maturité (questionnaire Maturity & Compliance)

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| A1 | **Minimal :** « occasional mild violence and/or light unrealistic blood ». Ce label ne mentionne aucun contenu basé sur la peur. | [Content maturity](https://create.roblox.com/docs/production/promotion/content-maturity) (sans date, consultée le 2026-10-08) | — | Le concept dépasse Minimal dès qu'il contient un jump scare. |
| A2 | **Mild :** « mild fear-based content ». Exemples cités : « Loud/heavy breathing, pounding heart, shrieking or screaming, creepy-looking NPCs, jump scares, ominous music, and/or gameplay that builds suspense ». La violence Mild est « implied or unrealistic ». | idem et [source GitHub creator-docs](https://github.com/Roblox/creator-docs/blob/main/content/en-us/production/promotion/content-maturity.md) | ✅ | **Label probable : Mild.** Les jump scares, les statues inquiétantes (creepy-looking NPCs), la musique angoissante et le suspense figurent explicitement dans les exemples Mild. |
| A3 | **Moderate :** « moderate fear-based content ». Exemples cités : bouches défigurées avec sang réaliste, tissus, organes ou vaisseaux visibles, plaies ouvertes réalistes, yeux qui saignent. Violence « non-graphic, realistic-looking » et « light realistic blood ». | idem | ⚠️ | **À éviter pour rester Mild :** tout sang réaliste, même léger ; les corps ou visages abîmés de façon réaliste ; toute mort ou blessure réaliste. La doc précise : « if there is any moment that the consequence of violence is realistic », le contenu passe en Moderate ou Strong, même si ce moment est unique. |
| A4 | **Restricted (18+) :** violence forte, sang réaliste abondant. Le jeu est alors réservé aux joueurs 18+ dont l'âge est vérifié. | idem | ✅ | Aucun risque si l'on reste sans gore. |
| A5 | Un jeu sans informations de maturité voit sa jouabilité restreinte pour tous les joueurs. | idem | ⚠️ | Remplir le questionnaire **avant** la bêta. Le re-soumettre à chaque ajout de contenu plus intense, par exemple pour l'aile « Sous-sol ». |
| A6 | Les hangouts sociaux, la création libre (free-form user creation) et les « sensitive issues » sont réservés aux 16+. Le jeu simulé d'argent jouable est interdit. | idem et [Help : Roblox Kids et Select](https://en.help.roblox.com/hc/en-us/articles/48163847200532-What-are-Roblox-Kids-and-Roblox-Select) | ✅ | Le jeu n'a ni lobby de type hangout ni outil de création libre. Garder le lobby fonctionnel (création d'équipe) plutôt que social. |

**Notes sur les termes.**
- **« Horreur » :** la page de maturité n'emploie pas ce mot ; elle parle de « fear-based content ». Le mot « Horror » n'apparaît que comme descripteur sur la page IA, sous la forme « Horror elements ».
- **« Violence fantastique » :** la page n'emploie pas ce terme ; elle distingue seulement violence réaliste et irréaliste. Des statues qui « attrapent » un garde sans montrer de blessure relèvent donc probablement de la violence Mild, implicite ou irréaliste. C'est une interprétation.

### B. Accès Kids (5-8 ans) et Select (9-15 ans)

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| B1 | **Maturité maximale :** Kids accepte Minimal ou Mild ; Select accepte Minimal, Mild ou Moderate. | [Roblox Kids and Select](https://create.roblox.com/docs/production/publishing/kids-and-select) (sans date, consultée le 2026-10-08) | ✅ | Avec un label Mild, le jeu est **éligible à Kids comme à Select**. Les jump scares n'excluent pas Kids s'ils restent Mild. |
| B2 | **Créateur :** compte en règle ; 2FA activée ; vérification de l'âge (pièce d'identité si 18+) ; **soit** un abonnement Roblox Plus ou Premium actif depuis **2 mois consécutifs**, **soit** des frais uniques et remboursables par jeu. | idem | ⚠️ | Hugo doit vérifier son âge avec une pièce d'identité et activer la 2FA. Il choisit ensuite entre les frais et l'abonnement. |
| B3 | **Montant des frais :** **1 000 R$** par jeu, remboursés 90 jours après l'éligibilité, ou 90 jours après le paiement si le jeu ne devient jamais éligible. Les frais sont perdus en cas de retrait pour violation grave. | [Annonce staff du 11/05/2026](https://devforum.roblox.com/t/alternate-publishing-requirements-for-roblox-kids-and-select/4630944) (mises à jour le 19/05 et le 27/05) et [annonce du 16/06/2026](https://devforum.roblox.com/t/roblox-kids-and-select-global-launch-upcoming-updates-to-eligibility-ads-manager-and-expedited-review/4685717) | ⚠️ | Prévoir 1 000 R$ immobilisés. Le paiement en Robux est une dépense : **validation de Hugo requise**. |
| B4 | **Processus d'entrée :** (1) phase d'essai réservée aux 16+ dont l'âge est vérifié ; (2) analyse des joueurs « highly engaged » ; (3) revue de sécurité ; (4) seuil de **250 parties uniques de joueurs highly engaged dont l'âge est vérifié, sur 60 jours**. | doc Kids and Select et [annonce staff du 19/08/2026](https://devforum.roblox.com/t/highly-engaged-player-threshold-drops-to-250/4820164) (le seuil passe de 500 à 250) | ⚠️ | **Le jeu sera d'abord joué uniquement par des 16+ vérifiés.** Le lancement doit donc séduire les jeunes adultes, alors que le public 10-15 ans n'arrive qu'après. Suivre l'onglet *Audience Reach* du Creator Hub. |
| B5 | **Joueur « highly engaged » :** dépend de l'ancienneté du compte, du temps de jeu dans le jeu et d'une dépense minimale sur Roblox au cours des 60 derniers jours. Le joueur n'a pas besoin de dépenser dans ce jeu, et un jeu gratuit compte. | doc Kids and Select (FAQ) | ✅ | Le montant de la dépense minimale n'est pas publié. |
| B6 | **Maintien dans le catalogue :** 25 joueurs highly engaged sur 60 jours (au lieu de 100 depuis le 16/07/2026). | annonce staff du 16/06/2026, mise à jour du 16/07 | ✅ | — |
| B7 | **Revue accélérée :** frais remboursables par jeu ; elle accélère la revue mais les standards restent les mêmes. Montant : **50 000 R$** selon la doc actuelle, **100 000 R$** selon l'annonce de juin 2026. | doc Kids and Select ; annonce du 16/06/2026 | ❓ | Les deux sources se contredisent ; la doc est probablement la plus récente. Sans objet pour un projet solo. |
| B8 | **Mécaniques limitées pour les comptes Kids et Select :** chat désactivé par défaut pour Kids (activable par un parent) ; chat introduit progressivement pour Select si l'âge est vérifié ; hangouts sociaux et dessin libre réservés aux 16+. | doc Kids and Select ; Help Kids/Select | ⚠️ | **La coop ne doit pas dépendre du chat.** Prévoir une communication sans texte libre : pings, marqueurs, emotes et messages prédéfinis. C'est une interprétation, mais elle est nécessaire si un joueur sur deux n'a pas de chat. |
| B9 | **Flux de médias récompensés :** depuis le 25/08/2026, un jeu est exclu de Kids et Select s'il a **à la fois** (a) un flux de vidéos, images ou posts, (b) un défilement automatique (autoplay, boucle, scroll infini ou autoscroll) et (c) une récompense, réelle ou implicite, pour le visionnage. Les rewarded video ads ne sont pas concernées. | [Annonce Roblox du 25/08/2026](https://devforum.roblox.com/t/roblox-kids-and-select-new-restrictions-on-reward-driven-media-feeds/4829188) | ⚠️ | La rediffusion de fin de nuit ne doit **pas** tourner en boucle automatique **ni** donner de récompense pour être regardée. Une lecture unique, lancée par le joueur et passable, convient. |
| B10 | **Liste exhaustive des mécaniques interdites :** aucune page officielle consultée n'en publie une. | — | ❓ | La revue de sécurité reste au jugement de Roblox. |

### C. Monétisation

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| C1 | Le créateur touche **70 %** des Robux dépensés en passes et developer products. Les 30 % restants sont la « Marketplace Fee » de Roblox. | [How do I make money?](https://create.roblox.com/docs/get-started/monetization) (sans date, consultée le 2026-10-08) | ✅ | Recalculer les estimations du rapport avec ce taux de 70 %. |
| C2 | **Prix d'un developer product :** de 1 R$ à 1 milliard de R$. L'attribution doit passer par `ProcessReceipt`, jamais par `PromptProductPurchaseFinished`. Ne renvoyer `PurchaseGranted` qu'après l'attribution, et `NotProcessedYet` en cas d'échec. Un seul script serveur doit gérer ce callback. | [Developer products](https://create.roblox.com/docs/production/monetization/developer-products) (mise à jour le 2026-10-07) | ✅ | Le revive à 25 R$ respecte la fourchette. Implémenter `ProcessReceipt` de façon idempotente. |
| C3 | **Prix minimum des game passes :** la page officielle n'a pas pu être consultée (erreur 404). | — | ❓ | Vérifier le minimum dans le Creator Hub au moment de créer les passes. Les prix prévus (49 à 299 R$) sont très probablement au-dessus. |
| C4 | **Serveurs privés :** abonnement mensuel en Robux, ou gratuits si « Requires Robux » est désactivé. Ils sont incompatibles avec un accès payant. Changer le prix annule les abonnements. Les moins de 13 ans peuvent ne pas pouvoir les rejoindre selon les réglages parentaux. Aucun prix minimum ni part créateur n'est indiqué. | [Private servers](https://create.roblox.com/docs/production/monetization/private-servers) (mise à jour le 2026-10-07) | ✅ | Fixer le prix une fois pour toutes. Ne pas en faire la seule façon de jouer entre amis. |
| C5 | **Revive payant en coop :** aucune règle officielle consultée n'interdit un achat qui donne un avantage. Le staff a déclaré n'avoir « no plans to add any age-based restrictions on purchase of content ». Ce propos concerne le Creator Store et ne vise pas explicitement les achats en jeu. | [AMA staff du 16/04/2026](https://devforum.roblox.com/t/ama-publishing-requirements-roblox-kids-select/4580953) | ✅ (interprétation) | Aucune règle ne bloque le revive. **Interprétation :** il contredit en partie la promesse « aucun avantage acheté » du rapport. Recommandation : revive gratuit une fois par nuit (ou accordé via rewarded ad pour les 13+), le revive payant restant un raccourci. |
| C6 | **Caisses aléatoires payantes, si on en ajoute un jour :** il faut afficher tous les résultats possibles et leurs probabilités en %, dont la somme fait exactement 100 % (ou ajouter un avertissement d'arrondi). Les probabilités doivent être indiquées avant toute dépense indirecte (clés, tickets) et les boosts de chance chiffrés. Si `ArePaidRandomItemsRestricted` vaut `true`, le joueur ne doit pas accéder au générateur ; il faut appliquer un traitement de repli : voie gratuite, ordre fixe divulgué, achat direct, masquage ou blocage. | [Paid random items](https://create.roblox.com/docs/en-us/production/monetization/paid-random-items) (sans date, consultée le 2026-10-08) | ✅ (exclu du MVP) | Garder l'exclusion. En cas d'ajout, appliquer cette checklist. |
| C7 | **PolicyService :** `PolicyService:GetPolicyInfoForPlayerAsync(player)` renvoie notamment `AreAdsAllowed`, `ArePaidRandomItemsRestricted`, `IsPaidItemTradingAllowed`, `IsContentSharingAllowed`, `IsEligibleToPurchaseSubscription`, `IsEligibleToPurchaseCommerceProduct`, `IsSubjectToChinaPolicies`, `IsEndlessContentLoadAllowed` et `IsEndlessContentAutoplayAllowed`. `AllowedExternalLinkReferences` est un champ hérité qui renvoie toujours un tableau vide. | [PolicyService (référence API)](https://create.roblox.com/docs/en-us/reference/engine/classes/PolicyService) (consultée le 2026-10-08) | ⚠️ | Appeler cette fonction côté serveur, dans un `pcall`, à l'arrivée du joueur, puis mettre le résultat en cache. **À utiliser au MVP :** `IsContentSharingAllowed` (partage de la rediffusion), `IsEligibleToPurchaseSubscription` (si un abonnement est ajouté) et `IsPaidItemTradingAllowed` (si des échanges sont ajoutés). **Plus tard :** `ArePaidRandomItemsRestricted` si des caisses apparaissent. |
| C8 | **Paiement de l'accès :** l'accès payant est incompatible avec les serveurs privés. | Private servers | ✅ | Garder le jeu gratuit. |

### D. Rewarded Video Ads

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| D1 | **Jeu :** public et sans restriction ; **aucune création libre par les joueurs** ; **au moins 2 000 visiteurs uniques par mois en moyenne** ; questionnaire de maturité rempli et **approuvé**. | [Rewarded video ads](https://create.roblox.com/docs/en-us/production/promotion/rewarded-video-ads) (mise à jour le 2026-10-06) | ⚠️ | Les pubs ne sont accessibles qu'après la traction initiale. Une re-soumission du questionnaire suspend temporairement l'éligibilité (réponse sous 24 h). |
| D2 | **Éditeur :** 13 ans ou plus, compte en règle, 2FA activée et pièce d'identité vérifiée. | idem | ✅ | Hugo remplit ces conditions s'il fait les vérifications de B2. |
| D3 | **Récompense :** forcément un **developer product** normalement vendu en Robux. Pas de Robux, pas d'objet aléatoire, pas d'inflation de la monnaie. Elle est accordée immédiatement après la pub, via `ProcessReceipt` (canal `AdReward`). Montant conseillé : 3 à 10 R$. La pub est opt-in et clairement signalée. | idem et [AdService](https://create.roblox.com/docs/en-us/reference/engine/classes/AdService) | ⚠️ | Un revive par pub est cohérent, car c'est un developer product. Appeler `GetAdAvailabilityNowAsync` côté client juste avant l'affichage, puis `ShowRewardedVideoAdAsync` côté serveur. |
| D4 | **Âge des joueurs :** « Any rewarded advertising formats are not permitted for U13 users ». Les rewarded video indépendantes sont interdites depuis le 04/05/2026. | [New Advertising Policies & Standards](https://devforum.roblox.com/t/new-advertising-policies-standards/4527365), annonce staff du 20/03/2026 (mises à jour le 15/04 et le 17/06) | ⚠️ | Les 10-12 ans ne verront pas de pub récompensée. **Interprétation :** Roblox filtre probablement lui-même via la disponibilité de la pub, mais la doc ne le dit pas explicitement. Prévoir une voie gratuite équivalente pour les moins de 13 ans. |
| D5 | **Rémunération :** à l'impression. Le tarif n'est pas publié. | Rewarded video ads | ❓ | Ne pas chiffrer ce revenu dans le business plan sans données réelles. |

### E. Taux DevEx majoré (US 18+)

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| E1 | **Taux et date :** 0,0054 $ par Earned Robux, au lieu de 0,0038 $ en standard, depuis le 08/06/2026. | [U.S. 18+ exchange rate](https://create.roblox.com/docs/production/monetization/18-plus-devex-rate) (sans date, consultée le 2026-10-08) ; [annonce DevForum](https://devforum.roblox.com/t/introducing-the-us-18-devex-rate-earn-42-more-on-spend-from-18-us-players/4607091) | — | — |
| E2 | **Acheteurs concernés :** joueurs **des États-Unis** dont l'âge (18+) est vérifié par estimation faciale ou pièce d'identité. **Achats concernés :** developer products, passes, abonnements et serveurs privés. | idem | ⚠️ | Seule la part payée par des adultes américains vérifiés bénéficie du taux majoré. |
| E3 | **Avatars :** les joueurs doivent passer **100 % du temps de jeu actif** en R15 (standard ou avancé), ou avec un personnage humanoïde ou non humanoïde personnalisé qui remplit les critères. Le réglage d'avatar du projet doit être **R15 Only**, et les packs d'animation doivent être en R15. Un seul passage possible en R6 rend le jeu inéligible. Roblox fait des vérifications continues. | idem | ⚠️ | Régler le projet en **R15 Only dès la semaine 1**. Cela ne coûte rien pour ce concept. |
| E4 | **Lieu de résidence du créateur :** la page ne dit rien sur la résidence du créateur ; elle ne parle que des joueurs américains. | idem | ❓ | Rien n'indique qu'un créateur français soit exclu, mais ce n'est pas confirmé. **Interprétation :** le gain sera marginal, car le public visé a surtout 10-17 ans. |

### F. DevEx pour Hugo (France, majeur)

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| F1 | **Âge minimum :** 13 ans. | [Developer Exchange](https://create.roblox.com/docs/production/monetization/developer-exchange) (sans date, consultée le 2026-10-08) | ✅ | — |
| F2 | **Seuil :** **30 000 Earned Robux** au minimum, soit 114 $ au taux standard. | idem | ⚠️ | Il faut 30 000 R$ nets côté créateur, soit environ 42 900 R$ dépensés par les joueurs après la commission de 30 %. C'est un calcul, pas une donnée de la source. |
| F3 | **Compte :** e-mail vérifié, compte DevEx Portal (géré par Tipalti), formulaire fiscal **W-8** pour les non-Américains, respect des Terms of Use et des Community Standards. Moyen de paiement et formulaire à renseigner sous 7 jours, sinon la demande est refusée. | idem | ⚠️ | **Interprétation :** pour une personne physique résidant en France, le formulaire est en pratique le W-8BEN. La page dit seulement « W-8 ». |
| F4 | **Fréquence :** une demande aboutie par mois civil. Traitement : environ 10 jours ouvrés la première fois, environ 5 ensuite. Les Robux gagnés avant le 05/09/2025 sont convertis à 0,0035 $ et encaissés en premier. | idem | ✅ | — |
| F5 | **Vérification d'identité pour DevEx :** la page la mentionne seulement en cas d'écart de nom. Premium n'est pas exigé. | idem | ❓ | La vérification par pièce d'identité sera de toute façon faite pour Kids/Select et pour les pubs (B2, D2). |
| F6 | **Fiscalité française** des revenus DevEx. | — | ❓ | Hors du périmètre des sources Roblox. Consulter un comptable ou les impôts (revenus BNC ou micro-entreprise). |

### G. Autres règles 2025-2026 pertinentes

| # | Règle (fait) | Source | Verdict | Action |
|---|---|---|---|---|
| G1 | **Liens et pseudos de réseaux sociaux :** interdits **dans le jeu** depuis début 2026. Ils sont autorisés sur la page du jeu, le profil et les Communities, visibles seulement par les 13+ vérifiés (16+ depuis le 30/06/2026 selon l'annonce Kids/Select). Une phrase comme « Check out our social media links on our game's page » est permise. | [Annonce staff du 18/11/2025](https://devforum.roblox.com/t/age-checks-to-access-chat-studio-team-create-and-links-on-roblox/4079702) (mise à jour le 07/01/2026) ; annonce du 16/06/2026 | ⛔ si on affiche « @… sur TikTok » en jeu | **Aucune mention TikTok ni lien dans l'interface.** Mettre le compte TikTok sur la page du jeu seulement. |
| G2 | **Captures vidéo :** `CaptureService` permet `StartVideoCaptureAsync`, `StopVideoCapture` et `PromptShareCapture`. La clé `IsContentSharingAllowed` indique si le joueur peut partager du contenu. | [CaptureService](https://create.roblox.com/docs/reference/engine/classes/CaptureService) ; PolicyService | ⚠️ | Pour la rediffusion, utiliser l'API native de capture et de partage, et masquer le bouton si `IsContentSharingAllowed` vaut `false`. La doc ne dit pas si l'export **hors plateforme** (vers TikTok) est possible : ❓. **Interprétation :** l'export direct vers TikTok depuis le jeu n'est pas prévu ; le joueur enregistre lui-même son écran. |
| G3 | **Flux de médias récompensés :** voir B9. | — | ⚠️ | — |
| G4 | **IA générative :** tout contenu, y compris généré par IA, doit respecter les Community Standards, et le créateur en est responsable. Les **interactions IA en jeu** (chat, voix, images, génération 3D) doivent être déclarées dans le questionnaire. Les interactions IA prolongées (style chatbot ou avec mémoire) imposent le label Restricted. | [Games with Generative AI](https://create.roblox.com/docs/generative-AI) (sans date, consultée le 2026-10-08) | ✅ | Des assets **pré-générés** par IA (textures, modèles) ne sont pas visés par une règle de déclaration spécifique : la source est muette, ❓. **Ne pas ajouter de PNJ conversationnel IA** : il ferait passer le jeu en 18+. Vérifier les droits de propriété intellectuelle des assets IA et du Creator Store. |
| G5 | **Données analytics de mineurs :** aucune règle propre aux créateurs n'a été trouvée dans les sources consultées. | — | ❓ | **Interprétation et prudence :** ne collecter aucune donnée personnelle hors de Roblox, n'envoyer aucune donnée de joueur à un service tiers, et s'en tenir à l'AnalyticsService ou aux Creator Analytics natifs. Les SDK tiers de mesure publicitaire sont interdits (annonce Advertising). |
| G6 | **Règle propre aux jeux d'horreur en 2026 :** aucune annonce officielle spécifique n'a été trouvée. Le genre est encadré par les seuls labels de maturité (A1 à A4). | recherche sur DevForum et la doc, 2026-10-08 | ❓ | — |
| G7 | **Seuil d'entrée Kids/Select abaissé à 100 en novembre 2026 :** relayé par un compte non officiel (Bloxy News sur X). **Aucune annonce staff ne le confirme** dans les sources consultées. | — | ❓ | Ne pas en tenir compte avant une annonce officielle. |

---

## 2. Points bloquants

Aucun point n'est bloquant si les adaptations ci-dessous sont faites.

**Bloquants conditionnels, à respecter impérativement :**
1. **Pas de lien ni de pseudo TikTok dans le jeu (G1).** La viralité passe par la page du jeu et par le partage natif.
2. **Pas de sang ni de blessure réalistes, même un instant (A3).** Sinon le jeu passe en Moderate et perd Kids ; au pire il passe en Restricted et perd tout le public de moins de 18 ans.
3. **La rediffusion ne doit pas être à la fois automatique, en boucle et récompensée (B9).** Sinon le jeu est exclu de Kids et Select.
4. **Questionnaire de maturité rempli avant la bêta (A5).** Sans lui, la jouabilité est restreinte ; il est aussi exigé pour les rewarded ads.
5. **Conditions créateur pour Kids/Select (B2, B3) :** vérification d'identité, 2FA, et 1 000 R$ de frais ou 2 mois de Plus/Premium. **Toute dépense en Robux demande la validation de Hugo.**

**Risque stratégique, non réglementaire (B4).** Pendant la phase d'essai, seuls les 16+ vérifiés peuvent jouer. Il faut 250 joueurs highly engaged en 60 jours avant d'atteindre les 10-15 ans visés. Le lancement doit donc fonctionner auprès des 16-25 ans.

---

## 3. Incertitudes

- **Prix minimum des game passes (C3) :** la page officielle était en erreur 404 lors de la consultation.
- **Montant de la revue accélérée (B7) :** 50 000 R$ selon la doc, 100 000 R$ selon l'annonce de juin.
- **Montant de la dépense minimale d'un joueur « highly engaged » (B5) :** non publié.
- **Taux majoré pour un créateur non américain (E4) :** la source est muette.
- **Filtrage des moins de 13 ans pour les rewarded ads (D4) :** la règle est claire, mais la doc ne dit pas comment le filtrage se fait techniquement.
- **Export de clips hors plateforme (G2) :** la source est muette.
- **Assets générés par IA hors interactions en jeu (G4) et données analytics de mineurs (G5) :** les sources sont muettes.
- **Seuil de 100 joueurs en novembre 2026 (G7) :** non confirmé par Roblox.
- **Classement Mild ou Moderate :** le label final est attribué par Roblox après le questionnaire. Le classement Mild retenu ici repose sur les exemples officiels, mais **c'est une interprétation**. Des statues au visage très dégradé ou une aile « Sous-sol » plus crue pourraient le faire basculer en Moderate.
- **Fiscalité française des revenus DevEx (F6) :** hors du périmètre.
