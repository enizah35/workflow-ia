# Mission : CircularRouter, site vitrine haut de gamme (cadrage)

- **Date :** 2026-10-12
- **Demande de Hugo :** « en utilisant le même workflow d'agent, j'aimerais voir pour créer un vrai site web qualitatif avec de belles animations, loin des sites vibe codés (AI slop) pour ce projet » + cahier des charges CircularRouter (docx, 9 oct. 2026).
- **Projet / dépôt :** CircularRouter, nouveau (pas encore de dépôt)

## Cadrage
- Objectif en une phrase : proposer une direction artistique, une stack et un plan de dev pour un site vitrine CircularRouter haut de gamme et animé, prouvés par une section prototype.
- Livrables : /mnt/project-files/circularrouter/ : cahier-des-charges.md, references.md, direction-artistique.md, plan-technique.md, prototype de l'ouverture (HTML).
- Critères de réussite : chaque choix tracé au cahier ; aucune promesse ni chiffre inventé ; prototype fluide, lisible sans JS, reduced-motion et pause OK.
- Défauts choisis :
  - Site de référence (chatgpt.site) inaccessible depuis le conteneur (proxy 403) : on part du cahier, pas de l'existant.
  - Le dev attend la validation de la DA par Hugo et la création d'un dépôt.

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| 1 | Références de sites et techniques d'animation | chercheur-references-web | variante de chercheur | fait |
| 2 | Stack, formulaire, hébergement, plan par phases | architecte | existante | fait |
| 3 | Direction artistique + prototype de l'ouverture | directeur-artistique-web | nouvelle | fait |

## Journal
- 17:55 Vague 1 lancée en parallèle : chercheur-references-web, architecte. Site de référence bloqué par le proxy (403).

- Architecte terminé : plan-technique.md (Astro 6 + GSAP natif, Netlify + Resend + Turnstile, phases 0-4, ~20 agents estimés à justifier). Alertes : alias contact@ au lieu d achille@ dans le mailto, identité éditeur LCEN.
- Chercheur terminé : references.md (15 réf., WebFetch texte seulement, aucun rendu vu ; site actuel lisible via WebFetch). Leçon : demander des captures navigateur quand le rendu compte.
- Directeur artistique lancé (DA + prototype ouverture + schéma, captures Playwright exigées).
- DA terminé : concept « le fil conducteur », Schibsted Grotesk + Newsreader Italic (OFL), prototype ouverture + schéma, 8 captures, 4 itérations. Contrôle du Chef : captures 1440/390 relues, cohérentes. Prototype publié en artifact https://claude.ai/artifact/2xDE5bHyNcNX96groQEqau.

## Résultat livré
Dossier /mnt/project-files/circularrouter/ : cahier-des-charges.md, references.md, plan-technique.md, direction-artistique.md, prototype/. En attente de hugo : validation DA (fil d'or, polices), création du dépôt, décisions propriétaire (alias contact@, mentions légales, hébergement Netlify, formulaire Resend).

## Leçons
- Exiger des captures Playwright relues par l'agent DA a produit 4 itérations utiles (visuel vide, lettre rognée, débordements) : à garder dans la fiche.
- Le chercheur sans navigateur ne voit pas le rendu : pour une recherche visuelle, lui faire prendre des captures.
- Le plan estime ~20 agents sur 5 phases (au-delà de 1-5) : à faire valider par Hugo avant la phase 0.

## Compteurs
- Agents lancés : 3
- Relances : 0
