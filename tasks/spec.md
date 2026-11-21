# Spec — Game Timer (à valider)

Statut : **proposition, rien n'est codé.** Écrite le 21/11/2026 avec Antoni, qui a
tranché par questions : app séparée de game-library, IGDB pour les dates avec RAWG
en repli, date EU prioritaire, tri et filtre par console, notification par Discord
comme dans gta6-backend.

## Objectif

Antoni tape le nom d'un jeu à venir ; l'app trouve le jeu, sa jaquette et ses dates
de sortie **console par console**, et affiche un compte à rebours pour chacune. Le
jour où une sortie approche ou bouge, un message arrive dans son salon Discord, sans
que l'app soit ouverte.

Un seul utilisateur (Galaxy S26 Ultra, PC Windows). PWA sur GitHub Pages
(`antoniman31.github.io/Game_Timer/`), pas d'APK : Discord rend la notification
native inutile.

## Ce que l'app fait

1. **Recherche** : saisie → 300 ms sans frappe → jusqu'à 8 suggestions (miniature,
   titre, année). Insensible à la casse et aux accents. Rien sous 2 caractères.
2. **Ajout** : un choix ajoute le jeu à la liste suivie ; un doublon (même id IGDB)
   est refusé avec un message.
3. **Carte d'un jeu** : jaquette 2:3, titre, puis **une ligne par plateforme** :
   date, compte à rebours vivant (j h min s), ou « Sorti le … ». Une plateforme sans
   date exacte (mois, trimestre, année, « TBD ») montre ce texte, **jamais** un faux
   compte à rebours.
4. **Choix de la date d'une plateforme** : Europe, sinon Monde, sinon la plus
   précoce des autres régions, avec la région écrite à côté quand ce n'est pas
   l'Europe.
5. **Tri** : sortie la plus proche (défaut), titre, ajout. **Filtre** par
   plateforme, construit à partir des plateformes présentes dans la liste.
6. **Retrait** d'un jeu, avec annulation pendant 5 s (comme game-library).
7. **Discord** : une fois par jour, à 9 h heure de Paris, le Worker relit les dates
   d'IGDB et envoie **un seul message** s'il y a quelque chose à dire :
   - jalons **J-7, J-1, Jour J** par plateforme ;
   - **report** : une date exacte qui change, ou une date exacte qui redevient vague ;
   - **avancée** de date, signalée de la même façon.
   Comptes à rebours en balise Discord `<t:…:R>`, recalculés par Discord.
8. **Réglages** : clé RAWG (facultative), code de liste, URL du Worker.

## Hors périmètre, volontairement

- Possédé / voulu / prêts / notes : c'est game-library.
- APK Android, notifications push du navigateur.
- Import de la liste de souhaits Steam (gardé pour plus tard, voir RESTE-A-FAIRE).

## Architecture

```
PWA (GitHub Pages)  ──/igdb──▶  Worker Cloudflare  ──▶  api.igdb.com (+ id.twitch.tv)
       │            ──/liste─▶      │  KV : listes, état des notifications, jeton
       │                            │  cron quotidien ──▶ webhook Discord
       └──── direct ──────────────────────────────────▶  api.rawg.io (repli, clé d'Antoni)
```

- **IGDB n'accepte pas d'appel depuis un navigateur** (pas de CORS) et demande un
  secret Twitch : le Worker détient `TWITCH_CLIENT_ID` et `TWITCH_CLIENT_SECRET`
  (secrets Cloudflare), obtient le jeton d'application et le garde en KV jusqu'à
  son expiration. C'est un écart assumé avec game-library, dont le Worker ne
  détient aucun secret : ici le secret est inévitable, et il ne sort jamais du
  Worker.
- **La liste vit sur l'appareil** (`gt_v1`) **et dans KV** : sans cette copie, le
  cron ne saurait pas quoi surveiller. Elle est déposée sous l'empreinte SHA-256
  d'un **code de liste** de 26 caractères, comme `/sync` dans game-library.
- **Discord** : `DISCORD_WEBHOOK_URL` en secret Cloudflare, masquée dans tout
  journal (leçon de `masquer_urls` dans gta6-backend).
- **Jaquette** : `images.igdb.com/igdb/image/upload/t_cover_big/<image_id>.jpg`,
  déjà en 2:3. Pas de SteamGridDB en V1.

## Données

Un jeu suivi (`gt_v1`, tableau) :

```js
{
  igdbId: 119133,              // entier > 0, clé d'unicité
  titre: "Elden Ring",
  jaquette: "co4jni",          // image_id IGDB, ou URL RAWG en repli
  source: "igdb",              // "igdb" | "rawg"
  ajouteLe: "2026-11-21",
  sorties: [                   // une entrée par plateforme, déjà choisie (règle 4)
    { plateforme: "PS5", date: "2027-03-12", precision: "jour", region: "europe" },
    { plateforme: "Switch 2", date: null, precision: "tbd", texte: "TBD", region: "monde" }
  ]
}
```

`precision` vaut `jour`, `mois`, `trimestre`, `annee` ou `tbd`. Tout ce qui est relu
(stockage, KV, réponse IGDB ou RAWG) passe par une fonction de validation : entrée
invalide écartée, nombre de corrections annoncé à l'écran.

**Dates en jours civils, sans heure.** IGDB donne un horodatage à minuit UTC ; on
n'en garde que le jour. Le compte à rebours vise **00:00 heure de Paris** ce jour-là,
et J-N se compte en jours civils à Paris, jamais en divisant par 24 h (le piège des
dodos de gta6-backend).

## Commandes

```
npm install
npm run dev          # http://localhost:5173/Game_Timer/
npm run build
npm run lint         # oxlint
npm test             # node --test src/lib/*.test.mjs worker/test.mjs
CHROMIUM=/opt/pw-browsers/chromium npm run verif:ui   # zones tactiles, contraste, focus, débordements
cd worker && npx wrangler deploy                       # fait par la CI en temps normal
```

## Structure

```
index.html, vite.config.js, package.json
src/main.jsx, src/App.jsx, src/index.css
src/components/    Recherche, CarteJeu, LigneSortie, Filtres, Reglages
src/lib/           logique pure, un *.test.mjs à côté de chaque module
  dates.js         choix de la date par plateforme, précision, J-N Paris, compte à rebours
  igdb.js          requête Apicalypse, normalisation de la réponse
  rawg.js          repli, normalisation
  liste.js         stockage local, validation, tri, filtre
  api.js           seul module qui appelle le réseau (mince, comme dans game-library)
worker/index.js    routes /igdb, /liste ; handler scheduled
worker/discord.js  jalons, reports, construction du message (pur)
worker/test.mjs
scripts/verif-ui.mjs
tasks/             spec et plan
.github/workflows/ deploy.yml (Pages), worker.yml (Worker), dependabot.yml
```

## Style de code

Celui de game-library : React 19 + Vite, JS sans TypeScript, noms en français,
commentaires qui disent **pourquoi** et le défaut rencontré, jetons CSS (`--tap`,
couleurs) jamais de valeur en dur, `const` déclaré avant usage.

```js
// La date retenue pour une plateforme.
//
// L'Europe d'abord : c'est là qu'Antoni achète. Sans date européenne, la date
// mondiale, puis la plus précoce des autres régions — une sortie japonaise
// trois mois avant l'Europe reste une information, à condition d'écrire la
// région à côté pour qu'elle ne passe pas pour la nôtre.
export function dateRetenue(sorties) { … }
```

## Tests

`node --test`, comme game-library. Un test ne compte qu'après l'avoir vu échouer.
Cas obligatoires :

- J-N juste à minuit à Paris, et autour des changements d'heure (28/03/2027,
  31/10/2027).
- Choix de région : EU > Monde > plus précoce ; précision vague → pas de compte à
  rebours.
- Validation : stockage corrompu, champ manquant, id en double, date impossible.
- Worker : origine refusée, cible inconnue, jeton expiré renouvelé une seule fois,
  IGDB en erreur → message lisible, URL du webhook absente de tout journal.
- Discord : jalon envoyé une seule fois (état en KV), report détecté, rien à dire →
  aucun message, message sous les limites Discord (10 embeds, 4 096 caractères).
- `verif:ui` sur toute PR qui touche l'interface.

## Limites

- **Toujours** : lint et tests avant chaque commit ; entrées réseau et stockage
  traitées comme non fiables ; une écriture qui échoue (stockage, KV, réseau) se
  voit à l'écran ; README / JOURNAL / RESTE-A-FAIRE mis à jour à chaque lot.
- **Demander d'abord** : toute dépendance nouvelle ; tout changement de CI ; toute
  route ajoutée au Worker ; tout message réel dans le salon Discord.
- **Jamais** : clé, secret, URL de webhook ou code de liste dans le dépôt ou la
  conversation ; fusion sans le « go » d'Antoni ; test sauté pour obtenir du vert ;
  « ça marche » écrit pour ce qui n'a pas tourné sur le S26 ou le PC.

## Réussite

- Taper « hollow knight silk » propose le jeu en moins d'une seconde après la
  dernière frappe (mesuré sur le S26, réseau mobile).
- Une carte montre une ligne par plateforme avec la bonne date EU, vérifiée à la
  main sur trois jeux contre la page IGDB.
- Une date vague n'affiche jamais de compte à rebours.
- Le cron envoie un message TEST dans le salon, puis, en réel, un jalon une seule
  fois, et un report quand une date bouge (simulé en modifiant l'état en KV).
- Lint, tests et `verif:ui` verts ; zones tactiles ≥ 48 px ; contraste mesuré dans
  les deux thèmes.

## Questions ouvertes

1. **Flux .ics** pour Google Agenda (≈ 20 lignes dans le Worker) : en V1, plus tard,
   ou jamais ?
2. **Heure du message** : 9 h à Paris convient-il ?
3. **Jalons** : J-7, J-1, Jour J suffisent-ils, ou ajouter J-30 ?
4. **Discord** : même salon que gta6-backend, ou un salon à part ? (Un salon à part
   évite de noyer les sorties dans le fil GTA.)
