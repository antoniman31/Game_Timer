# Spec — Game Timer (à valider)

Statut : **proposition, rien n'est codé.** Écrite le 21/11/2026 avec Antoni, qui a tranché
chaque point par questions. Heures de Paris. Les aperçus visuels validés (liste, grille,
recherche, réglages, messages Discord, compteurs, couleurs de console) sont des pages
d'artefact privées de la session du 21/11 ; ce document en reprend les décisions.

## Objectif

Antoni tape le nom d'un jeu à venir ; l'app trouve le jeu, sa jaquette et ses dates de
sortie **console par console**, et affiche un compte à rebours pour chacune de **ses
machines**. Le jour où une sortie approche ou bouge, un message arrive dans un salon
Discord à part, sans que l'app soit ouverte.

Un seul utilisateur (Galaxy S26 Ultra, PC Windows). PWA sur GitHub Pages
(`antoniman31.github.io/Game_Timer/`), pas d'APK : Discord rend la notification native
inutile.

## Ce que l'app fait

1. **Recherche** : saisie, 300 ms sans frappe, jusqu'à 8 suggestions (miniature, titre,
   année, « à venir » ou « sorti »). Insensible à la casse et aux accents. Rien sous
   2 caractères.
2. **Mes machines** : un réglage coche les machines dont on affiche les sorties. Par
   défaut **Switch 2, Xbox Series X|S, PC**. Choix d'Antoni : **seules les machines
   cochées comptent**, sans rétrocompatibilité implicite (un jeu Xbox One seul
   n'apparaît pas pour la Series X, sauf si Xbox One est cochée).
3. **Ajout** : un choix ajoute le jeu ; un doublon (même id IGDB) est refusé avec un
   message. Si le jeu ne sort sur **aucune** des machines cochées, l'app prévient
   (« Pas sur tes machines : PS5 seulement ») et laisse choisir de le suivre quand même.
   Les consoles masquées n'ont ni ligne, ni compte à rebours, ni message Discord, et se
   réaffichent dans les réglages.
4. **Liste détaillée** (vue par défaut) : une carte par jeu, jaquette 2:3, titre, puis
   **une ligne par date de sortie**. Chaque ligne commence par la **pastille** de sa
   console ; deux consoles qui sortent le même jour partagent la ligne. Un jeu qui ne
   sort que sur une console porte un **liseré** de sa couleur sur la carte. Une plateforme
   sans date exacte (mois, trimestre, année, TBD) montre ce texte, **jamais** un faux
   compte à rebours.
5. **Compteur : unités progressives**, couleur selon le palier :

   | Temps restant | Affichage | Couleur |
   |---|---|---|
   | 1 mois ou plus | `3 mois 12 j` (mois civils à Paris) | accent |
   | de 7 jours à 1 mois | `12 j 4 h` | accent |
   | de 24 h à 7 jours | `6 j 14 h 13 min` | bleu-violet `#8488e7` |
   | de 1 h à 24 h | `5 h 12 min 08 s` | violet `#be82cf` |
   | moins d'1 h | `12 min 08 s` | rose `#f87db7` |
   | passé | `Sorti le 14 nov.` | vert |

   Les **secondes n'apparaissent que le dernier jour.** Il y a **une seule horloge** pour
   toute la page : cadence d'une seconde seulement si une carte visible est sous 24 h,
   d'une minute sinon, arrêtée quand l'onglet est caché. Sur la sortie la plus proche en
   tête de liste, des **tuiles** (jours, heures, min, s) remplacent le texte.
6. **Vue grille**, au choix par un bouton liste / grille (choix gardé sur l'appareil) :
   jaquette, **pastilles de console** en haut à gauche, titre, sortie la plus proche
   avec « +2 consoles », et une **barre** de progression calée sur la **date d'annonce**
   du jeu. Si IGDB ne donne pas de date d'annonce fiable, **repli : les 30 derniers jours
   avant la sortie** (voir Questions ouvertes). Une touche ouvre la fiche.
7. **Fiche d'un jeu** : une ligne par console avec sa date et son compteur, et la
   **frise de jours** : une case par jour sur les 14 prochains jours, le jour de sortie
   cerclé de rose ; au-delà de 14 jours, du texte.
8. **Onglet « Sortis »** : un jeu y passe le lendemain de sa sortie sur la dernière de
   ses machines à sortir ; il reste dans « À venir » tant qu'une de ses machines
   attend. Pas d'export vers game-library.
9. **Tri** : sortie la plus proche (défaut), titre, ajout. **Filtre** par console parmi
   les machines cochées ; le filtre actif **prend la couleur de sa console**.
10. **Retrait** d'un jeu, avec annulation pendant 5 s.
11. **Date retenue pour une console** : Europe, sinon Monde, sinon la plus précoce des
    autres régions, la région écrite à côté quand ce n'est pas l'Europe. Un réglage
    permet de préférer le Monde.
12. **Réglages** : thème Système / Clair / Sombre, mes machines, code de liste, clé RAWG
    (facultative, avec « Tester »), région prioritaire, message TEST Discord, état de
    l'envoi de la liste.

## Discord

Un message par jour, à **5 h 30 heure de Paris**, dans **un salon à part**, **un seul
message** même pour plusieurs jeux, rien si rien à dire. Uniquement les machines cochées.

- **Jalons** : J-30 (dates exactes au jour seulement), J-7, J-1, Jour J.
- **Reports** : une date exacte qui change, ou qui redevient vague. **Avancées** :
  signalées de la même façon.
- **Dimanche** : le même message gagne une section « Cette semaine » (sorties des 7 jours
  suivants), pas un deuxième message.
- **Anti-bruit** : deux jalons d'un même jeu le même jour sur deux consoles sont regroupés
  en une ligne.
- **Forme** : une carte par événement, **barre latérale à la couleur du palier** (rose sous
  24 h, orange sous 7 jours, comme GTA6_WATCH), la console dite par un **carré de couleur**
  dans le texte (🟦 PlayStation, 🟩 Xbox, 🟥 Nintendo, ⬛ PC), compte à rebours en balise
  Discord `<t:…:R>`.
- **Sonnerie** : tout notifie normalement (décision de gta6-backend). Le message sonne à
  5 h 30 : vérifier le mode « Ne pas déranger » du S26.

## Hors périmètre, volontairement

- Possédé, prêté, noté : c'est game-library. **Aucun export vers game-library.**
- Les dodos. L'APK. Les notifications push du navigateur.
- Plus tard (dans RESTE-A-FAIRE) : flux calendrier `.ics`, import de la liste de
  souhaits Steam.

## Architecture

```
PWA (GitHub Pages) ──/igdb───▶ Worker Cloudflare ──▶ api.igdb.com (+ id.twitch.tv)
       │            ──/liste──▶    │  KV : listes, état des notifications, jeton
       │                           │  cron 03:30 et 04:30 UTC ──▶ webhook Discord
       └──── direct ─────────────────────────────────▶ api.rawg.io (repli, clé d'Antoni)
```

- **IGDB n'accepte pas d'appel depuis un navigateur** et demande un secret Twitch : le
  Worker détient `TWITCH_CLIENT_ID` et `TWITCH_CLIENT_SECRET` (secrets Cloudflare), obtient
  le jeton d'application et le garde en KV jusqu'à son expiration. Écart assumé avec
  game-library, dont le Worker ne détient aucun secret : ici le secret est inévitable et ne
  sort jamais du Worker. Le corps de la requête IGDB est **construit par le Worker** à
  partir d'un seul paramètre borné : ce n'est pas un accès ouvert à IGDB.
- **Un Worker à soi** (`game-timer-proxy`), pas celui de game-library.
- **La liste vit sur l'appareil** (`gt_v1`) **et dans KV**, sous l'empreinte SHA-256 d'un
  **code de liste** de 26 caractères, comme `/sync` dans game-library : sans cette copie,
  le cron ne saurait pas quoi surveiller. Les machines cochées partent avec la liste.
- **Cron à 5 h 30 Paris** : Cloudflare planifie en UTC, donc le Worker se déclenche à
  03:30 **et** 04:30 UTC et n'agit que quand il est 5 h 30 à Paris, une fois par jour
  (état en KV). Cela règle l'heure d'été sans rien changer deux fois par an.
- **Discord** : `DISCORD_WEBHOOK_URL` en secret Cloudflare, masquée dans tout journal
  (leçon de `masquer_urls` dans gta6-backend).
- **Jaquette** : `images.igdb.com/igdb/image/upload/t_cover_big/<image_id>.jpg`, déjà en 2:3.

## Style (repris de gta6-backend, couleurs de console de game-library)

- **Thèmes** : `data-theme="light|dark"` sur `<html>`, réglage Système / Clair / Sombre,
  **posé par un script en ligne avant le premier affichage** (pas d'éclair noir en
  clair), `theme-color` mis à jour. Variables CSS, jamais de valeur en dur dans un
  composant.
- **Sombre** : fond `#0a0d0f`, panneau `#10151a`, surélevé `#161c22`, filet `#1e2730`,
  texte `#e4ece8`, atténué `#7e918b`, accent `#3380ff`. **Clair** : `#f4f7fa`, `#ffffff`,
  `#eef3f8`, `#d7e1ea`, `#10222f`, `#566b7e`, accent `#1f5fd9`. Grille discrète en
  fond (`--grid-tint`).
- **Police** : JetBrains Mono. **Rayons** : 4 (pastilles), 8 (commandes), 12 (cartes).
  **Zones tactiles** : 48 px.
- **Chiffres du compteur** : halo en sombre, **trait de contour de 1,5 px** en clair
  (`-webkit-text-stroke`, `paint-order: stroke fill`), pas de halo en clair.
- **Couleurs de console**, **en pastille pleine à texte blanc, jamais en texte**. Mesures
  du 21/11/2026 (blanc sur l'aplat ; la même couleur en texte sur `#0a0d0f`) :

  | Console | Couleur | Blanc / aplat | En texte, sombre |
  |---|---|---|---|
  | PS5 | `#0b3d91` | 10,04:1 | 1,94:1 |
  | PS4 | `#082c6b` | 13,27:1 | 1,47:1 |
  | Xbox Series X\|S | `#107C10` | 5,37:1 | 3,63:1 |
  | Xbox One | `#0a5c0a` | 8,24:1 | 2,37:1 |
  | Switch 2 | `#e4000f` | 4,88:1 | 4,00:1 |
  | Switch | `#b3000c` | 7,19:1 | 2,71:1 |
  | PC | `#4b5563` | 7,56:1 | 2,58:1 |

  Plateforme inconnue : gris neutre, nom tel quel. La couleur de console dit **quelle
  machine** ; la couleur de palier dit **si c'est bientôt** : les deux ne se mélangent
  jamais.

## Données

Un jeu suivi (`gt_v1`, tableau) :

```js
{
  igdbId: 119133,              // entier > 0, clé d'unicité
  titre: "Elden Ring",
  jaquette: "co4jni",          // image_id IGDB, ou URL RAWG en repli
  source: "igdb",              // "igdb" | "rawg"
  ajouteLe: "2026-11-21",
  annonce: "2026-06-04",       // facultatif : date d'annonce, si IGDB la donne
  horsMachines: false,         // suivi malgré l'absence sur les machines cochées
  sorties: [                   // une entrée par plateforme, déjà choisie (règle 11)
    { plateforme: "PS5", date: "2027-03-12", precision: "jour", region: "europe" },
    { plateforme: "Switch 2", date: null, precision: "tbd", texte: "TBD", region: "monde" }
  ]
}
```

`precision` vaut `jour`, `mois`, `trimestre`, `annee` ou `tbd`. Réglages : `gt_machines`
(liste d'id de plateformes), `gt_theme`, `gt_keys`, `gt_code`, `gt_affichage` (liste ou
grille). Tout ce qui est relu (stockage, KV, réponse IGDB ou RAWG) passe par une fonction
de validation : entrée invalide écartée, nombre de corrections annoncé à l'écran.

**Dates en jours civils, sans heure.** IGDB donne un horodatage à minuit UTC ; on n'en
garde que le jour. Le compte à rebours vise **00:00 heure de Paris** ce jour-là ; J-N et
les mois se comptent en jours et mois civils à Paris, jamais en divisant par 24 h.

## Commandes

```
npm install
npm run dev          # http://localhost:5173/Game_Timer/
npm run build
npm run lint         # oxlint
npm test             # node --test src/lib/*.test.mjs worker/test.mjs
CHROMIUM=/opt/pw-browsers/chromium npm run verif:ui   # tactile, contraste, focus, débordements, deux thèmes
cd worker && npx wrangler deploy                       # fait par la CI en temps normal
```

## Structure

```
index.html, vite.config.js, package.json
src/main.jsx, src/App.jsx, src/index.css
src/components/    Recherche, CarteJeu, LigneSortie, Compteur, Fiche, Sortis, Filtres, Reglages
src/lib/           logique pure, un *.test.mjs à côté de chaque module
  dates.js         date retenue, précision, J-N Paris, mois civils, compteur à unités progressives
  plateformes.js   table des consoles (id IGDB, nom court, couleur), machines cochées
  igdb.js          requête Apicalypse, normalisation de la réponse
  rawg.js          repli, normalisation
  liste.js         stockage local, validation, tri, filtre, passage dans « Sortis »
  theme.js         jetons, réglage Système / Clair / Sombre
  api.js           seul module qui appelle le réseau
worker/index.js    routes /igdb, /liste ; handler scheduled
worker/discord.js  jalons, reports, dimanche, message (pur)
worker/test.mjs
scripts/verif-ui.mjs
tasks/             spec et plan
.github/workflows/ deploy.yml (Pages), worker.yml (Worker), dependabot.yml
```

## Tests

`node --test`, comme game-library. Un test ne compte qu'après l'avoir vu échouer. Cas
obligatoires :

- J-N et mois civils justes à minuit à Paris, et autour des changements d'heure
  (28/03/2027, 31/10/2027).
- Compteur : chaque palier à sa frontière (7 j, 24 h, 1 h), secondes seulement sous 24 h,
  date vague sans compte à rebours.
- Choix de région : Europe > Monde > plus précoce ; machines cochées ; jeu hors machines.
- Validation : stockage corrompu, champ manquant, id en double, date impossible.
- Passage dans « Sortis » : lendemain de la dernière machine, pas avant.
- Worker : origine refusée, cible inconnue, jeton expiré renouvelé une seule fois, IGDB en
  erreur, URL du webhook absente de tout journal.
- Discord : jalon envoyé une seule fois, report détecté, regroupement, dimanche, rien à
  dire donc aucun message, limites Discord respectées, 5 h 30 Paris en été comme en hiver.
- `verif:ui` sur toute PR qui touche l'interface, **dans les deux thèmes**.

## Limites

- **Toujours** : lint et tests avant chaque commit ; entrées réseau et stockage traitées
  comme non fiables ; une écriture qui échoue (stockage, KV, réseau) se voit à l'écran ;
  README, JOURNAL et RESTE-A-FAIRE mis à jour à chaque lot.
- **Demander d'abord** : toute dépendance nouvelle ; tout changement de CI ; toute route
  ajoutée au Worker ; tout message réel dans le salon Discord.
- **Jamais** : clé, secret, URL de webhook ou code de liste dans le dépôt ou la
  conversation ; fusion sans le « go » d'Antoni ; test sauté pour obtenir du vert ;
  « ça marche » écrit pour ce qui n'a pas tourné sur le S26 ou le PC.

## Réussite

- « hollow knight silk » propose le jeu moins d'une seconde après la dernière frappe
  (mesuré sur le S26, réseau mobile).
- Une carte montre une ligne par date, avec la bonne date européenne, vérifiée à la main
  sur trois jeux contre la page IGDB.
- Une date vague n'affiche jamais de compte à rebours.
- Aucune seconde ne défile sur une carte à plus de 24 h de sa sortie.
- Le cron envoie un message TEST dans le salon, puis, en réel, un jalon une seule fois, et
  un report quand une date bouge (simulé en modifiant l'état en KV), à 5 h 30 Paris.
- Lint, tests et `verif:ui` verts dans les deux thèmes ; zones tactiles ≥ 48 px ; contraste
  mesuré.

## Questions ouvertes

1. **Date d'annonce** : à ma connaissance IGDB n'en expose pas de fiable (`created_at` dit
   quand la fiche a été créée, pas quand le jeu a été annoncé). À vérifier dans la doc
   avant d'écrire le relais (tâche T2). Si elle manque, la barre de la grille se cale sur
   les 30 derniers jours.
2. **Correspondance des plateformes** : relever les identifiants IGDB exacts de Switch 2,
   Xbox Series X|S et PC, et ce que l'API renvoie pour les versions Steam / Epic.
3. **Sonnerie à 5 h 30** : confirmer que le mode « Ne pas déranger » couvre cette heure.
