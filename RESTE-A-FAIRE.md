# Reste à faire

## Par quoi reprendre

1. **Antoni valide la spec et le plan** (`tasks/spec.md`, `tasks/plan.md`). Rien ne se code avant.
2. **T0, le socle** (PR 1) : `package.json`, Vite, oxlint, `node --test`, manifeste PWA, thème et jetons, `deploy.yml`,
   `dependabot.yml`, `LICENSE`. Les règles et les fichiers de suivi sont déjà faits.
3. Puis les PR 2 à 6 du plan, dans l'ordre : recherche, liste, grille et fiche, Discord, mise en service.

## Questions ouvertes (spec)

- IGDB expose-t-il une date d'annonce fiable ? À vérifier dans la doc avant T2 ; sinon, la barre de la grille se cale sur
  les 30 derniers jours avant la sortie.
- Les identifiants IGDB exacts de Switch 2, Xbox Series X|S et PC, et ce que l'API renvoie pour Steam et Epic.

## À faire par Antoni

- Avant la PR 2 : une application Twitch (dev.twitch.tv), ses secrets dans le Worker, `CLOUDFLARE_API_TOKEN` dans les
  secrets GitHub du dépôt.
- Avant la PR 5 : un webhook Discord dans un salon à part, en secret du Worker.

## Plus tard (hors V1)

- Flux calendrier `.ics` des sorties suivies.
- Import de la liste de souhaits Steam.
