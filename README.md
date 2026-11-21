# Game Timer

Compte à rebours des sorties de jeux vidéo, console par console, sur les seules machines d'Antoni (par défaut Switch 2,
Xbox Series X|S et PC), avec un message Discord à 5 h 30, heure de Paris, quand une sortie approche ou change de date.

## État actuel

**Rien n'est codé.** Le dépôt contient la spec (`tasks/spec.md`) et le plan de réalisation (`tasks/plan.md`), écrits et
validés par Antoni le 21/11/2026, plus les fichiers de règles et de suivi. Pas encore de site en ligne, pas de
CI, pas de Worker.

Prévu par la spec : une PWA sur GitHub Pages (`antoniman31.github.io/Game_Timer/`, React 19 et Vite), un Worker Cloudflare
`game-timer-proxy` qui interroge IGDB (RAWG en repli) et envoie les messages Discord par son cron.

## Où lire la suite

- `tasks/spec.md` : ce que l'app fera, et ce qu'elle ne fera pas.
- `tasks/plan.md` : les tâches T0 à T10 et leur découpage en six PR.
- `RESTE-A-FAIRE.md` : par quoi reprendre.
- `JOURNAL.md` : l'histoire du dépôt.
- `CLAUDE.md` : les règles de travail.
