# Plan : Game Timer V1

Décidé avec Antoni le 21/11/2026 ; spec et plan validés par lui le même jour, avec la règle
« versions et API à jour dans la doc officielle ». La spec est dans `tasks/spec.md`. Le code
démarre avec la PR 1.

## Décisions d'architecture

- **App séparée**, même stack que game-library (React 19, Vite, oxlint, PWA, Pages), pour
  réutiliser les habitudes et les scripts plutôt que les réinventer.
- **Un Worker à soi** (`game-timer-proxy`) : celui de game-library ne détient aucun secret
  par principe, et cette session ne peut pas y pousser.
- **IGDB d'abord, RAWG en repli** : IGDB seul donne une date par plateforme et par région.
- **Discord par le cron du Worker**, pas par GitHub Actions : la liste est en KV, et le
  `schedule` de GitHub dérive (constaté dans gta6-backend, `update-feeds.yml`). Cron à
  03:30 et 04:30 UTC ; le code n'envoie qu'à 5 h 30 Paris.
- **Logique pure dans `src/lib/` et `worker/discord.js`**, réseau confiné à `api.js` et aux
  routes : tout se teste sans téléphone ni IGDB.
- **Style de gta6-backend** (thèmes, jetons, JetBrains Mono, halo et trait de contour des
  chiffres) et **couleurs de console de game-library** (pastilles pleines, texte blanc).
- **Une seule horloge** pour la page, à cadence variable (1 s sous 24 h, sinon 1 min).

## Graphe de dépendances

```
T0 socle ─▶ T1 dates ─┬─▶ T3 recherche + machines ─▶ T4 liste ─▶ T5 tri/filtre ─┬─▶ T5b grille
            T2 /igdb ─┘                                                          ├─▶ T5c fiche
                                                                                 └─▶ T5d Sortis
T3 ─▶ T6 réglages + repli RAWG
T2 ─▶ T7 /liste (KV) ─▶ T8 message Discord (pur) ─▶ T9 cron + essai TEST ─▶ T10 mise en service
```

T1 et T2 sont indépendantes. Le risque le plus fort (champs IGDB, date d'annonce, jeton)
est en T2, tôt exprès.

## Découpage en PR (une CI par PR)

| PR | Tâches | Ce qu'Antoni peut essayer à la fin |
|---|---|---|
| 1 · socle | T0 | le site vide en ligne, les deux thèmes, la CI verte |
| 2 · recherche | T1, T2, T3 | chercher un jeu, l'ajouter, voir l'avertissement hors machines |
| 3 · liste | T4, T5, T6 | la liste complète, compteurs, filtre, réglages |
| 4 · grille et fiche | T5b, T5c, T5d | la grille, la fiche et l'onglet Sortis |
| 5 · Discord | T7, T8, T9 | un message TEST dans le salon |
| 6 · mise en service | T10 | les vrais jalons à 5 h 30 |

## Tâches

### T0 — le socle et les règles (M)
- [ ] `package.json`, Vite (`base: "/Game_Timer/"`), oxlint, `node --test`, manifeste PWA
- [ ] `CLAUDE.md` : **le bloc commun recopié mot pour mot** depuis game-library (identique
      dans gta6-backend), puis les règles propres (commandes, conventions, secrets Twitch
      et Discord dans Cloudflare, `CLOUDFLARE_API_TOKEN` dans GitHub)
- [ ] `README.md` (état actuel), `JOURNAL.md` (premier lot), `RESTE-A-FAIRE.md`
      (y mettre le `.ics` et la liste de souhaits Steam), `LICENSE`
- [ ] `src/lib/theme.js`, script en ligne avant le premier affichage, jetons des deux
      thèmes, police JetBrains Mono, `theme-color`
- [ ] `deploy.yml` : lint + tests + build à chaque PR, déploiement Pages sur `main` ; job
      « Interface » (`verif:ui`) sauté si la PR ne touche que des `.md`
- [ ] `dependabot.yml` groupé ; actions épinglées par SHA
- [ ] **versions à jour relevées dans la doc officielle** (React, Vite, oxlint, Wrangler,
      Playwright, JetBrains Mono, actions GitHub) avant d'écrire `package.json` et les
      workflows : aucun numéro de version écrit de mémoire, aucun code déprécié
- Vérif : CI verte sur la PR, page vide servie sur Pages après fusion, pas d'éclair noir
  en thème clair
- Fichiers : racine, `.github/`, `src/lib/theme.js`

### T1 — les dates et le compteur (M)
- [ ] `dateRetenue` : Europe > Monde > plus précoce, région écrite si non Europe
- [ ] précisions `jour | mois | trimestre | annee | tbd`, texte lisible pour chacune
- [ ] `joursRestants` et mois civils à Paris ; `compteur(ms)` aux frontières 7 j, 24 h, 1 h
- [ ] `frise14(jours)` pour la fiche
- [ ] tests : minuit Paris, 28/03/2027 et 31/10/2027, chaque frontière de palier, date vague
- Fichiers : `src/lib/dates.js`, `src/lib/dates.test.mjs`

### T2 — le relais IGDB (M)
- [ ] **avant d'écrire** : relire la doc IGDB (`release_dates` : région, plateforme, format
      de date ; **existence d'une date d'annonce**) — ne pas se fier à la mémoire
- [ ] `POST /igdb/recherche` : corps Apicalypse **construit par le Worker** à partir du seul
      paramètre `q` (borné à 80 caractères), jamais transmis brut
- [ ] `POST /igdb/jeux` : détail par liste d'ids (≤ 50), pour le cron et le rafraîchissement
- [ ] jeton Twitch en KV, renouvelé à l'expiration ou sur un 401, une seule fois
- [ ] liste blanche d'origines (Pages + localhost) ; erreur IGDB → JSON lisible
- [ ] `src/lib/igdb.js` : normalisation vers le modèle de la spec, testée sur des réponses
      enregistrées ; `src/lib/plateformes.js` : table des consoles et couleurs
- Vérif : `npm test` ; après déploiement et secrets saisis par Antoni, une vraie recherche
  depuis `npm run dev`
- Fichiers : `worker/index.js`, `worker/test.mjs`, `worker/wrangler.toml`,
  `src/lib/igdb.js`, `src/lib/plateformes.js` (+ leurs tests)

### T3 — chercher, ajouter, machines (M)
- [ ] champ de recherche, 300 ms, requête précédente annulée, suggestions en vrais boutons
- [ ] ajout dans `gt_v1`, doublon refusé, échec d'écriture affiché
- [ ] machines cochées (défaut Switch 2, Xbox Series X|S, PC) ; **jeu hors machines :
      avertissement et choix de le suivre quand même**
- [ ] `liste.js` : lecture validée (corrections annoncées), écriture
- Vérif : tests de `liste.js` ; `verif:ui` ; essai réel sur le PC
- Fichiers : `src/components/Recherche.jsx`, `src/lib/liste.js`, `src/lib/liste.test.mjs`,
  `src/lib/api.js`, `src/App.jsx`

### Point d'étape 1 — après PR 2
- [ ] Antoni a saisi les secrets Twitch et le jeton Cloudflare
- [ ] une recherche réelle marche sur le S26 et le PC
- [ ] trois jeux vérifiés à la main contre leur page IGDB
- [ ] la question « date d'annonce » est tranchée

### T4 — la liste détaillée (M)
- [ ] carte : jaquette 2:3 (cadre réservé), titre, une ligne par date, **pastille de console**
      pleine à texte blanc, **liseré** si une seule console
- [ ] compteur à unités progressives et couleurs de palier ; tuiles sur la sortie la plus
      proche ; halo en sombre, trait de contour en clair
- [ ] **une seule horloge**, 1 s sous 24 h, 1 min sinon, arrêtée onglet caché
- [ ] retrait avec annulation de 5 s
- Fichiers : `src/components/CarteJeu.jsx`, `LigneSortie.jsx`, `Compteur.jsx`, `src/index.css`

### T5 — tri et filtre (S)
- [ ] tri : sortie la plus proche (une date vague passe après les dates exactes), titre,
      ajout ; filtre par console parmi les machines cochées, **couleur de la console quand
      il est actif** ; choix gardés sur l'appareil
- Fichiers : `src/components/Filtres.jsx`, `src/lib/liste.js` (+ test)

### T6 — réglages et repli RAWG (M)
- [ ] thème Système / Clair / Sombre, machines, région, clé RAWG avec « Tester »
- [ ] repli RAWG si IGDB ne rend rien ou échoue : une date globale, marquée « source RAWG »
- Fichiers : `src/lib/rawg.js` (+ test), `src/components/Reglages.jsx`

### Point d'étape 2 — après PR 3
- [ ] liste de 10 jeux réels lisible à 320 px, dans les deux thèmes
- [ ] aucune seconde ne défile à plus de 24 h
- [ ] à vérifier sur le S26 et le PC : liste écrite dans la PR

### T5b — la grille (S)
- [ ] bascule liste / grille mémorisée ; jaquette, pastilles de console, sortie la plus
      proche avec « +N consoles »
- [ ] barre sur la date d'annonce, repli sur les 30 derniers jours (selon T2)
- Fichiers : `src/components/Grille.jsx`, `src/index.css`, `src/lib/dates.js` (+ test)

### T5c — la fiche (S)
- [ ] une ligne par console avec date et compteur ; frise de 14 jours
- Fichiers : `src/components/Fiche.jsx`, `src/index.css`

### T5d — l'onglet « Sortis » (S)
- [ ] passage le lendemain de la dernière machine à sortir ; liste séparée ; pas d'export
- Fichiers : `src/components/Sortis.jsx`, `src/lib/liste.js` (+ test)

### Point d'étape 3 — après PR 4
- [ ] grille, fiche et Sortis essayés sur le S26

### T7 — la liste dans KV (S)
- [ ] code de liste de 26 caractères, engendré par l'app, gardé dans `gt_code`, jamais dans
      l'export ; `PUT/GET /liste` sous l'empreinte SHA-256, plafond de taille ; machines
      cochées incluses
- [ ] dépôt à chaque modification ; échec visible (pastille « non envoyée »)
- [ ] **avant d'écrire** : relever dans la doc Cloudflare l'API KV et ses limites (taille d'une
      valeur, cohérence entre lecture et écriture)
- Fichiers : `worker/index.js`, `worker/test.mjs`, `src/lib/api.js`, `src/App.jsx`

### T8 — le message Discord, en pur (M)
- [ ] **avant d'écrire** : relever dans la doc Discord les limites d'un webhook et d'un embed
      (nombre, longueurs) et le format des balises de temps `<t:…:R>`
- [ ] `aAnnoncer(listes, datesFraiches, etat, maintenant)` : jalons J-30 (dates exactes),
      J-7, J-1, Jour J ; reports et avancées ; regroupement anti-bruit ; machines cochées
      seulement ; chaque annonce une seule fois (clé d'état par jeu, console, jalon, date)
- [ ] section « Cette semaine » le dimanche
- [ ] `message(annonces)` : un message, une carte par événement, barre au palier, carrés de
      couleur par console, `<t:…:R>`, limites Discord respectées, rien à dire → `null`
- Fichiers : `worker/discord.js`, `worker/test.mjs`

### T9 — le cron et l'essai TEST (S)
- [ ] **avant d'écrire** : relire la doc Cloudflare sur les cron triggers (fuseau UTC, retards
      possibles, syntaxe de la planification)
- [ ] handler `scheduled` : n'agit qu'à 5 h 30 Paris, une fois par jour ; relit IGDB par lots
      de 50 ; met à jour l'état ; URL du webhook masquée dans les journaux
- [ ] route `POST /essai` protégée par le code de liste : message marqué **TEST**, déclenché
      par Antoni, pas par moi
- Fichiers : `worker/index.js`, `worker/wrangler.toml`, `worker/test.mjs`

### Point d'étape 4 — après PR 5
- [ ] Antoni a créé le webhook du salon à part et saisi `DISCORD_WEBHOOK_URL`
- [ ] le message TEST est arrivé et se lit bien sur le S26
- [ ] le mode « Ne pas déranger » couvre 5 h 30

### T10 — mise en service (XS)
- [ ] le cron envoie pour de vrai ; un report simulé (état KV modifié) produit son message ;
      README et JOURNAL à jour

### Point final
- [ ] critères de réussite de la spec tous cochés
- [ ] RESTE-A-FAIRE ne garde que ce qui reste vraiment

## Risques

| Risque | Impact | Parade |
|---|---|---|
| Champs IGDB renommés ou mal retenus | Haut | relire la doc avant T2, tester sur des réponses enregistrées |
| Pas de date d'annonce dans IGDB | Moyen | repli sur les 30 derniers jours, décidé avant T5b |
| Secret Twitch qui fuit | Haut | secret Cloudflare uniquement ; aucune route ne le renvoie ; aucun journal ne l'écrit |
| `/igdb` utilisé par un tiers (la liste d'origines n'est pas une barrière hors navigateur) | Moyen | corps construit par le Worker, `q` borné, résultats limités ; quota IGDB à relever dans la doc (T2) |
| Jeu invisible car hors de mes machines | Moyen | avertissement à l'ajout et choix de le suivre ; réaffichage dans les réglages |
| Titres français mal trouvés par IGDB | Moyen | chercher aussi dans `alternative_names` ; repli RAWG |
| Date IGDB en UTC lue comme la veille à Paris | Moyen | jours civils seulement, testés aux changements d'heure |
| Mois civils faux (`3 mois 12 j`) | Faible | tests aux fins de mois et années bissextiles |
| Secondes qui vident la batterie | Faible | une horloge, 1 s seulement sous 24 h, arrêt onglet caché |
| Jalon envoyé deux fois | Faible | état en KV écrit avant l'envoi suivant, clé par jalon |
| Message de 5 h 30 qui réveille | Faible | point d'étape 4 : vérifier « Ne pas déranger » |
| Webhook Discord révoqué | Faible | 404 journalisé (masqué), signalé dans l'app au prochain dépôt de liste |
| Éclair noir en thème clair | Faible | script de thème en ligne avant le premier affichage, testé par `verif:ui` |

## Ce que fait Antoni, et quand

1. Avant la PR 2 : application Twitch (dev.twitch.tv), puis `wrangler secret put
   TWITCH_CLIENT_ID` et `TWITCH_CLIENT_SECRET` ; `CLOUDFLARE_API_TOKEN` dans les secrets
   GitHub du dépôt.
2. Avant la PR 5 : webhook Discord dans un salon à part, puis `wrangler secret put
   DISCORD_WEBHOOK_URL`.
3. À chaque point d'étape : la liste « à vérifier sur le S26 ou le PC ».
