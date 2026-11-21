# Game Timer — règles du projet

## Règles communes (le même bloc dans tous les dépôts d'Antoni, existants et futurs : le modifier ici, c'est le modifier partout)

Décidées par Antoni le 16/11/2026, mises à jour le 21/11/2026. Les règles propres à ce dépôt suivent ce bloc ; en cas de conflit, ce bloc l'emporte.

- **Avec Antoni** : français, tutoiement, réponses courtes, franches, sans flatterie. Il est seul utilisateur (Galaxy S26 Ultra, PC Windows).
- **Proposer d'abord** : présenter un plan détaillé (étapes, fichiers touchés, approche) et ses propres suggestions d'amélioration, attendre un « ok » explicite avant de toucher quoi que ce soit, ne jamais régénérer un fichier de son propre chef. « Pose des questions » veut dire `AskUserQuestion`, avec des choix concrets et un recommandé. Pour une nouvelle fonctionnalité ou une modification de fonctionnalité ou de design : proposer sans coder, poser des questions si besoin et proposer des visuels (maquette HTML simple sans canvas, ou schéma ASCII).
- **Fusion** : PR en brouillon, fusion par squash **seulement sur le « go » d'Antoni**, CI verte. Jamais de fusion de sa propre initiative, même verte.
- **Branche** : celle que la session impose. Aucun nom de branche n'est écrit dans ce dépôt : l'environnement le change à chaque session.
- **Identité de commit** : `AntoniMan31 <268696974+antoniman31@users.noreply.github.com>`. Jamais l'adresse personnelle d'Antoni : GitHub refuse le push qui l'expose.
- **Noms de modèle** : autorisés (ligne de co-auteur des commits comprise).
- **Secrets** : jamais de clé, de jeton ni de mot de passe dans le dépôt ou la conversation ; ils se saisissent dans l'application ou dans les secrets GitHub/Cloudflare. Jamais d'export de données personnelles ni de `.env` commité.
- **Tests** : le lint et les tests du dépôt passent avant chaque commit. Un test ne compte qu'après l'avoir **vu échouer** sur le code d'avant. Jamais affaiblir un contrôle pour obtenir du vert (test sauté, seuil baissé, exception sans raison écrite).
- **CI, trois niveaux** : *rapide* (lint, tests unitaires, données) à chaque PR ; *lourd* (navigateur, e2e, Windows, compilation Android) seulement si la PR touche autre chose que des `.md`, sinon le job est marqué « sauté », jamais retiré ; *appareil réel* : une PR qui touche l'interface finit par une courte liste « à vérifier sur le S26 ou le PC ». Sur `main`, tout tourne toujours. Une seule CI par PR. Un test qui échoue au hasard a droit à une relance, puis on en cherche la cause.
- **Interface** : de vrais boutons (jamais un `div` cliquable) ; dans une appli utilisée au doigt, **zones tactiles d'au moins 48 px** (48 dp sous Android) ; contraste et focus vérifiés par l'outil du dépôt.
- **Robustesse** : une écriture qui peut échouer (stockage, réseau) se voit à l'écran ; toute donnée venue d'un fichier, du stockage ou du réseau est une entrée non fiable. Exporter les données avant toute opération destructive.
- **Versions et API** : vérifier dans la doc officielle les versions à jour avant d'utiliser une librairie ou une API ; jamais de code déprécié sans le signaler.
- **Contradiction** : si une demande contredit une règle de ce fichier, le signaler avant d'appliquer.
- **Règles à jour** : avant chaque nouvelle tâche, `git fetch origin`, puis relire le bloc « Règles communes » de `CLAUDE.md` sur la branche par défaut (`git show origin/<branche par défaut>:CLAUDE.md`) ; s'il a changé, appliquer la nouvelle version et le dire en une phrase.
- **Écrit** : commentaires en français qui disent **pourquoi** ; aucun chiffre écrit sans l'avoir mesuré ; ne jamais écrire « ça marche » pour ce qui n'a pas tourné sur l'appareil réel.
- **Suivi, trois fichiers à la racine** : `README.md` dit l'**état actuel** ; `JOURNAL.md` garde l'**histoire** (chaque lot s'y ajoute en fin de fichier, sous `### Titre — jj/mm/aaaa`) ; `RESTE-A-FAIRE.md` dit **ce qui reste**, mis à jour à chaque lot. Un fichier de suivi retiré part en entier dans `JOURNAL.md`, sous « Ancien X.md ».

## Règles propres à ce dépôt

Compte à rebours des sorties de jeux vidéo, console par console, pour les seules machines cochées d'Antoni, avec un
message Discord à 5 h 30 (heure de Paris) quand une sortie approche ou bouge. PWA sur GitHub Pages
(`base: "/Game_Timer/"`), même stack que game-library (React 19, Vite, oxlint, `node --test`), pas d'APK. **Rien n'est
codé** : la spec (`tasks/spec.md`) et le plan (`tasks/plan.md`) ont été validés par Antoni le 21/11/2026 ; le code démarre
avec la PR 1 (le socle). État : `README.md` ; ce qui reste : `RESTE-A-FAIRE.md` ; l'histoire : `JOURNAL.md`.

## Commandes (prévues par la spec, aucune n'existe encore)

- `npm run dev`, `npm run build`, `npm run lint` (oxlint), `npm test` (`node --test`, modules et Worker) : lint et tests
  avant tout commit.
- `CHROMIUM=/opt/pw-browsers/chromium npm run verif:ui` : tactile, contraste, focus, débordements, dans les deux thèmes,
  pour tout changement d'interface ; le job « Interface » de la CI est sauté sur une PR qui ne touche que des `.md`.
- Le Worker `game-timer-proxy` se déploie par la CI (`worker.yml`), pas à la main.

## Conventions

- Logique pure dans `src/lib/` (un `*.test.mjs` à côté de chaque module) et `worker/discord.js` ; le réseau ne passe que
  par `src/lib/api.js` et les routes du Worker.
- Heures de Paris partout ; une date IGDB se lit en jour civil, jamais comme un instant UTC. Une date vague (mois,
  trimestre, année, TBD) n'affiche jamais de compte à rebours.
- Une seule horloge pour la page : une seconde seulement si une carte visible est à moins de 24 h, une minute sinon, arrêtée
  onglet caché.
- Style de gta6-backend (jetons, deux thèmes, JetBrains Mono), couleurs de console de game-library.

## Limites de sécurité

- `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET` et `DISCORD_WEBHOOK_URL` vivent dans les secrets Cloudflare du Worker,
  `CLOUDFLARE_API_TOKEN` dans les secrets GitHub. Aucune route ne les renvoie, aucun journal ne les écrit (URL du webhook
  masquée).
- Le corps de la requête IGDB est construit par le Worker à partir du seul paramètre `q`, borné : jamais transmis brut.
- Le code de liste est la seule clé de la copie en KV : jamais dans le dépôt, la conversation ni un journal.
- Demander d'abord : toute dépendance nouvelle, tout changement de CI, toute route ajoutée au Worker, tout message réel dans
  le salon Discord.
