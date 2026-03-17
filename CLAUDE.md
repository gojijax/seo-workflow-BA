# Agent SEO — Monorepo multi-clients

## Demarrage

Au debut de chaque session :
1. `git pull` (recuperer les derniers changements)
2. Identifier le client actif (demander si pas clair)
3. Lire `clients/{client}/PROJECT-CONTEXT.md` — source de verite du client

## Architecture

Monorepo SEO : SOPs partagees, donnees client isolees.

```
seo/
├── CLAUDE.md                    <- ce fichier (point d'entree global)
├── .claude/commands/            <- skills projet (partages entre clients)
├── consignes/                   <- SOPs partagees (brief SEO, architecture, schema…)
├── clients/
│   ├── {client-a}/
│   │   ├── PROJECT-CONTEXT.md   <- config + memoire + etat projet
│   │   ├── briefs/{slug}/       <- outputs (JSON + brief + consigne)
│   │   └── journal/             <- journal operationnel
│   └── {client-b}/
│       ├── PROJECT-CONTEXT.md
│       ├── briefs/
│       └── journal/
```

## Convention multi-client

- Chaque skill accepte un argument `client` (ex: `/brief mon-client <url>`)
- Si le client n'est pas precise et qu'il y a ambiguite → demander
- Les chemins sont toujours relatifs au client : `clients/{client}/briefs/`, `clients/{client}/journal/`
- Le PROJECT-CONTEXT.md de chaque client contient TOUTE la config specifique (GSC, Drive, contacts, regles apprises, etat)

## Workflow brief SEO

L'utilisateur donne un client + une URL. Process complet :
1. Phase 1 : extraction brute (4 vagues sequentielles)
2. Phase 2 : diagnostic + checkpoint auto-skip
3. Phase 3 : JSON + brief SEO + consigne redacteur + export Google Drive

## Fichiers de reference — a lire AVANT de commencer un brief

Lis les fichiers dans `consignes/` AVANT de commencer. Ils contiennent TOUTES les regles, garde-fous, fallbacks et formats.

1. `consignes/sop-brief-seo-v3.md` — procedure complete
2. `consignes/architecture-brief-seo.md` — format de rendu
3. `consignes/brief-seo-schema.json` — schema JSON
4. `consignes/maillon2-consigne-redacteur.md` — consigne redacteur

## Regles cardinales

- Jamais de donnees inventees. Si un appel MCP echoue → noter "donnees indisponibles".
- Checkpoint Phase 2 non court-circuitable. Si auto-skip non rempli → STOP + validation humaine.
- Intent match ALERTE PIVOT ou cannibalisation > 2 conflits → STOP + validation humaine.
- Sequencement par vagues strict : Vague 1 → 2 → 3 → 4.
- Toute la substance est dans `consignes/`. Ce CLAUDE.md est un point d'entree, pas un substitut.

## Regles de communication

- Jamais d'affirmation sans data. Fournir la source (GSC, Haloscan, DataForSEO).
- Toujours lire PROJECT-CONTEXT.md du client AVANT de formuler un diagnostic.
- Cannibalisation : toujours croiser GSC (query overlap) + positions comparees avant de conclure.
- Communications : exposer les options, ne pas imposer les conclusions.
