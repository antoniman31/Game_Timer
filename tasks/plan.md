# Plan : Game Timer V1

Décidé avec Antoni le 21/11/2026. La spec est dans `tasks/spec.md`. Rien n'est codé
avant son « ok » sur la spec et sur ce plan.

## Décisions d'architecture

- **App séparée**, même stack que game-library (React 19, Vite, oxlint, PWA, Pages),
  pour réutiliser les habitudes et les scripts plutôt que les réinventer.
- **Un Worker à soi** (`game-timer-proxy`), pas celui de game-library : celui-ci ne
  détient aucun secret par principe, et la session ne peut pas y pousser.
- **IGDB d'abord, RAWG en repli** : IGDB seul donne une date par plateforme et par
  région ; RAWG ne sert que si IGDB ne trouve rien ou tombe.
- **Discord par le cron du Worker**, pas par GitHub Actions : la liste est en KV, et
  le `schedule` de GitHub dérive (constaté dans gta6-backend, `update-feeds.yml`).
  Le cron tourne à 07:00 et 08:00 UTC ; le code n'envoie qu'à 9 h à Paris, ce qui
  règle l'heure d'été sans rien changer deux fois par an.
- **Logique pure dans `src/lib/` et `worker/discord.js`**, réseau confiné à
  `api.js` et aux routes : tout se teste sans téléphone ni IGDB.

## Graphe de dépendances

```
T0 socle ─▶ T1 dates.js ─┬─▶ T3 recherche + ajout ─▶ T4 carte ─▶ T5 tri/filtre
            T2 /igdb ────┘                                     └─▶ T6 repli RAWG
T2 ─▶ T7 /liste (KV) ─▶ T8 message Discord (pur) ─▶ T9 cron + essai TEST ─▶ T10 envoi réel
                                                                        (T11 .ics, si retenu)
```

T1 et T2 sont indépendantes. Le risque le plus fort (IGDB, jeton, champs) est en T2,
tôt exprès.

## Découpage en PR (une CI par PR)

| PR | Tâches | Ce qu'Antoni peut essayer à la fin |
|---|---|---|
| 1 · socle | T0 | le site vide en ligne, la CI verte |
| 2 · recherche | T1, T2, T3 | chercher un jeu et l'ajouter |
| 3 · affichage | T4, T5, T6 | la liste complète, triée, filtrée |
| 4 · Discord | T7, T8, T9 | un message TEST dans le salon |
| 5 · mise en service | T10 (+ T11) | les vrais jalons |

## Tâches

### T0 — le socle et les règles (M)
- [ ] `package.json`, Vite (`base: "/Game_Timer/"`), oxlint, `node --test`, manifeste PWA
- [ ] `CLAUDE.md` : **le bloc commun recopié mot pour mot** depuis game-library, puis
      les règles propres (commandes, conventions, limites de sécurité : secrets
      Twitch et Discord dans Cloudflare, `CLOUDFLARE_API_TOKEN` dans GitHub)
- [ ] `README.md` (état actuel), `JOURNAL.md` (premier lot), `RESTE-A-FAIRE.md`, `LICENSE`
- [ ] `deploy.yml` : lint + tests + build à chaque PR, déploiement Pages sur `main` ;
      job « Interface » (`verif:ui`) sauté si la PR ne touche que des `.md`
- [ ] `dependabot.yml` groupé, comme les autres dépôts ; actions épinglées par SHA
- Vérif : CI verte sur la PR, page vide servie sur Pages après fusion
- Fichiers : racine, `.github/`

### T1 — les dates (S)
- [ ] `dateRetenue` : EU > Monde > plus précoce, région écrite si non EU
- [ ] précisions `jour | mois | trimestre | annee | tbd`, texte lisible pour chacune
- [ ] `joursRestants` en jours civils à Paris ; `compteARebours` vers 00:00 Paris
- [ ] tests : minuit Paris, 28/03/2027 et 31/10/2027, date vague, date passée
- Fichiers : `src/lib/dates.js`, `src/lib/dates.test.mjs`

### T2 — le relais IGDB (M)
- [ ] `POST /igdb/recherche` : le corps Apicalypse est **construit par le Worker** à
      partir d'un seul paramètre `q` (borné à 80 caractères), jamais transmis brut
      par le client : le Worker n'est pas un proxy IGDB ouvert
- [ ] `POST /igdb/jeux` : détail par liste d'ids (≤ 50), pour le cron et le rafraîchissement
- [ ] jeton Twitch en KV, renouvelé à l'expiration ou sur un 401, une seule fois
- [ ] liste blanche d'origines (Pages + localhost) ; erreur IGDB → JSON lisible
- [ ] **avant d'écrire** : relire la doc IGDB des champs de `release_dates` (région,
      format de date) — ils ont changé de nom en 2025, ne pas se fier à la mémoire
- [ ] `src/lib/igdb.js` : normalisation vers le modèle de la spec, testée sur des
      réponses enregistrées
- Vérif : `npm test` ; après déploiement et secrets saisis par Antoni, une vraie
  recherche depuis `npm run dev`
- Fichiers : `worker/index.js`, `worker/test.mjs`, `worker/wrangler.toml`,
  `src/lib/igdb.js`, `src/lib/igdb.test.mjs`

### T3 — chercher et ajouter (M)
- [ ] champ de recherche, attente de 300 ms, requête précédente annulée
- [ ] suggestions (miniature, titre, année), vrais `<button>`, clavier et doigt
- [ ] ajout dans `gt_v1`, doublon refusé, échec d'écriture affiché
- [ ] `liste.js` : lecture validée (corrections annoncées), écriture
- Vérif : tests de `liste.js` ; `verif:ui` ; essai réel sur le PC
- Fichiers : `src/components/Recherche.jsx`, `src/lib/liste.js`, `src/lib/liste.test.mjs`,
  `src/lib/api.js`, `src/App.jsx`

### Point d'étape 1 — après PR 2
- [ ] Antoni a saisi les secrets Twitch et le jeton Cloudflare
- [ ] une recherche réelle marche sur le S26 et le PC
- [ ] trois jeux vérifiés à la main contre leur page IGDB

### T4 — la carte d'un jeu (M)
- [ ] jaquette 2:3 (cadre réservé, pas de saut de mise en page), titre
- [ ] une ligne par plateforme : compte à rebours vivant, « Sorti le … », ou texte vague
- [ ] **une seule horloge** pour toute la page (pas un `setInterval` par ligne),
      arrêtée quand l'onglet est caché
- [ ] retrait avec annulation de 5 s
- Fichiers : `src/components/CarteJeu.jsx`, `LigneSortie.jsx`, `src/index.css`, `src/App.jsx`

### T5 — tri et filtre (S)
- [ ] tri : sortie la plus proche (une date vague passe après les dates exactes),
      titre, ajout ; filtre par plateforme présente ; choix gardés sur l'appareil
- Fichiers : `src/components/Filtres.jsx`, `src/lib/liste.js`, `src/lib/liste.test.mjs`

### T6 — repli RAWG et réglages (S)
- [ ] si IGDB ne rend rien ou échoue : RAWG avec la clé d'Antoni (`gt_keys`), une
      seule date globale, marquée « source RAWG » sur la carte
- [ ] écran Réglages : clé RAWG avec bouton « Tester », URL du Worker
- Fichiers : `src/lib/rawg.js`, `src/lib/rawg.test.mjs`, `src/components/Reglages.jsx`

### Point d'étape 2 — après PR 3
- [ ] liste de 10 jeux réels lisible à 320 px, thème clair et sombre
- [ ] à vérifier sur le S26 et le PC : liste écrite dans la PR

### T7 — la liste dans KV (S)
- [ ] code de liste de 26 caractères, engendré par l'app, gardé dans `gt_code`, jamais
      dans l'export ; `PUT/GET /liste` sous l'empreinte SHA-256, plafond de taille
- [ ] dépôt à chaque modification ; échec visible (pastille « non envoyée »)
- Fichiers : `worker/index.js`, `worker/test.mjs`, `src/lib/api.js`, `src/App.jsx`

### T8 — le message Discord, en pur (M)
- [ ] `aAnnoncer(listes, datesFraiches, etat, maintenant)` : jalons J-7 / J-1 / Jour J,
      reports, avancées ; chaque annonce une seule fois (clé d'état par jeu,
      plateforme, jalon et date)
- [ ] `message(annonces)` : un message, sections colorées, `<t:…:R>`, limites Discord
      respectées, rien à dire → `null`
- Fichiers : `worker/discord.js`, `worker/test.mjs`

### T9 — le cron et l'essai TEST (S)
- [ ] handler `scheduled` : n'agit qu'à 9 h à Paris ; relit IGDB par lots de 50 ;
      met à jour l'état ; URL du webhook masquée dans les journaux
- [ ] route `POST /essai` protégée par le code de liste : envoie un message marqué
      **TEST** — déclenchée par Antoni, pas par moi
- Fichiers : `worker/index.js`, `worker/wrangler.toml`, `worker/test.mjs`

### Point d'étape 3 — après PR 4
- [ ] Antoni a créé le webhook et saisi `DISCORD_WEBHOOK_URL`
- [ ] le message TEST est arrivé et se lit bien sur le S26

### T10 — mise en service (XS)
- [ ] le cron envoie pour de vrai ; un report simulé (état KV modifié) produit
      bien son message ; README et JOURNAL à jour

### T11 — flux .ics (S, seulement si retenu)
- [ ] `GET /calendrier.ics?code=…` : un événement par plateforme datée au jour,
      valide pour Google Agenda

### Point final
- [ ] critères de réussite de la spec tous cochés
- [ ] RESTE-A-FAIRE ne garde que ce qui reste vraiment

## Risques

| Risque | Impact | Parade |
|---|---|---|
| Champs IGDB renommés ou mal retenus | Haut | relire la doc avant T2, tester sur des réponses enregistrées |
| Secret Twitch qui fuit | Haut | secret Cloudflare uniquement ; aucune route ne le renvoie ; aucun journal ne l'écrit |
| `/igdb` utilisé par un tiers (la liste d'origines n'est pas une barrière hors navigateur) | Moyen | corps construit par le Worker, `q` borné, résultats limités ; quota IGDB 4 req/s : un abus casse le service, pas le compte |
| Titres français mal trouvés par IGDB | Moyen | chercher aussi dans `alternative_names` ; repli RAWG |
| Date IGDB en UTC lue comme la veille à Paris | Moyen | jours civils seulement, testés aux changements d'heure |
| Jalon envoyé deux fois (cron relancé) | Faible | état en KV écrit avant l'envoi suivant, clé par jalon |
| Webhook Discord révoqué ou supprimé | Faible | 404 du webhook journalisé (masqué) et signalé dans l'app au prochain dépôt de liste |

## Ce que fait Antoni, et quand

1. Avant PR 2 : application Twitch (dev.twitch.tv) → `wrangler secret put
   TWITCH_CLIENT_ID` et `TWITCH_CLIENT_SECRET` ; `CLOUDFLARE_API_TOKEN` dans les
   secrets GitHub du dépôt.
2. Avant PR 4 : webhook Discord → `wrangler secret put DISCORD_WEBHOOK_URL`.
3. À chaque point d'étape : la liste « à vérifier sur le S26 ou le PC ».
