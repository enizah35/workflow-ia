# Mission : CircularRouter, construction du vrai site

- **Date :** 2026-10-12
- **Demande de Hugo :** « fais-moi le vrai site avec des intégrations discrètes des animations, qu'on les remarque très subtilement, histoire de dynamiser le site »
- **Projet / dépôt :** CircularRouter ; dépôt local /home/claude/circularrouter (création de enizah35/circularrouter refusée par l'intégration GitHub : 403, Hugo doit le créer)

## Cadrage
- Objectif : site Astro complet (toutes sections du cahier, contact serveur codé, pages légales à compléter), DA piste A + motion v4 très discret, vérifié.
- Livrables : dépôt local prêt à pousser, aperçu publié, rapport de tests.
- Critères : build + astro check + tests Playwright/axe/Vitest verts ; contenu lisible sans JS ; 0 violation axe ; aucun nom personnel ni chiffre inventé.
- Défauts choisis : phases 0 à 3 du plan compressées en 3 vagues (5 agents au lieu de ~20) ; Netlify/Resend/Turnstile codés mais non branchés (comptes à créer) ; brief commun /mnt/project-files/circularrouter/brief-dev.md.

## Découpage
| # | Tâche | Agent | Fiche | Statut |
|---|-------|-------|-------|--------|
| 0 | Socle Astro, tokens/polices, layout, header, noyau motion, toutes sections en HTML, harnais de tests | dev-feature (variante Astro socle) | variante | fait (8503c1a) |
| 1a | Hero + Schéma (motion v4) | motion-designer-web | existante | en cours |
| 1b | Approche, Économie circulaire, Étapes (onglets) | dev-feature | existante | en cours |
| 1c | Repreneurs, FAQ, Footer, Contact (UI + fonction) + pages légales | dev-feature | existante | en cours |
| 2 | Intégration, coordination motion, tests, aperçu | integrateur | existante | à venir |

## Journal
- 23:58 Vague 0 lancée : socle (agent/socle). Brief commun écrit avant la vague.
- Socle terminé : Astro 7.3.8, main 8503c1a, build/check/vitest 5/5/e2e 30/30 verts. API registerMotion figée et ajoutée au brief. Ordre des sections et textes de synthèse à valider par Hugo.
- Vague 1 lancée en parallèle : 3 worktrees (cr-ouverture, cr-parcours, cr-contact), ports E2E distincts, budget mouvement commun (2 effets liés au défilement max).

## Résultat livré

## Leçons

## Compteurs
- Agents lancés : 4
- Relances : 0
