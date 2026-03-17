# seo-agent-claude

Un systeme complet de 7 commandes pour piloter un workflow SEO multi-clients depuis le terminal avec [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

Du brief data-driven a la publication WordPress, en passant par la redaction SEO et les rituels quotidiens — sans copier-coller, sans ouvrir la Search Console.

## Ce que ca fait

| Commande | Description |
|----------|-------------|
| `/brief` | Brief SEO complet : extraction data (GSC, Haloscan, DataForSEO), diagnostic, livrable structure |
| `/redaction` | Redaction SEO publiable avec gating en 3 etapes, personas lecteur, anti-patterns IA |
| `/publish` | Publication WordPress via API REST native — backup, conversion Markdown-HTML, verification |
| `/lp-locale` | Landing pages SEO local enrichies de donnees INSEE/DVF via data.gouv.fr |
| `/morning` | Rituel matin : taches Notion + emails + GSC en un bloc, plan du jour interactif |
| `/week-plan` | Planning hebdomadaire du lundi : bilan S-1, objectifs, priorisation par client |
| `/setup-client` | Onboarding client en 7 etapes : config GSC, Drive, Notion, contacts, PROJECT-CONTEXT auto-genere |

## Architecture

```
seo-agent-claude/
├── CLAUDE.md                          <- Point d'entree pour Claude Code
├── .claude/commands/                  <- Les 7 skills (commandes)
│   ├── brief.md
│   ├── redaction.md
│   ├── publish.md
│   ├── lp-locale.md
│   ├── morning.md
│   ├── week-plan.md
│   └── setup-client.md
├── consignes/                         <- SOPs et schemas
│   ├── sop-brief-seo-v3.md           <- Procedure complete brief SEO
│   ├── architecture-brief-seo.md     <- Format de rendu (3 couches)
│   ├── brief-seo-schema.json         <- Schema JSON du brief
│   └── maillon2-consigne-redacteur.md <- Transformation brief -> consigne redacteur
└── clients/
    └── example/                       <- Client exemple
        ├── PROJECT-CONTEXT.md         <- Config + memoire + etat projet
        ├── briefs/                    <- Output des briefs
        └── journal/                   <- Journal operationnel
```

Le principe : **un monorepo, des SOPs partagees, des donnees client isolees.**

Chaque client a son dossier avec un `PROJECT-CONTEXT.md` qui contient TOUTE sa config (domaine, GSC, contacts, ton, contraintes). Les skills lisent ce fichier pour s'adapter automatiquement.

## Pre-requis

### Claude Code
- [Installer Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- Cloner ce repo et ouvrir le dossier dans Claude Code

### MCP Servers (selon les skills utilises)

| MCP Server | Utilise par | Pourquoi |
|-----------|------------|---------|
| [Google Search Console](https://github.com/anthropics/claude-code) | `/brief`, `/morning`, `/week-plan` | Donnees de performance, requetes, positions |
| [Haloscan](https://haloscan.com) | `/brief`, `/redaction`, `/lp-locale` | Positions, concurrents, keywords |
| [DataForSEO](https://dataforseo.com) | `/brief`, `/redaction`, `/lp-locale` | SERP, content parsing, Lighthouse |
| [Notion](https://www.notion.so) | `/morning`, `/week-plan`, `/setup-client` | Taches, communications, projets |
| [Google Drive](https://drive.google.com) | `/brief`, `/setup-client` | Export des briefs |
| [Gmail](https://gmail.com) | `/morning` | Emails en attente |

Tous les MCP servers sont **optionnels**. Les skills s'adaptent : si un MCP n'est pas disponible, le skill note "donnees indisponibles" et continue.

## Quick Start

```bash
# 1. Cloner le repo
git clone https://github.com/bastienamoruso-art/seo-agent-claude.git
cd seo-agent-claude

# 2. Ouvrir dans Claude Code
claude

# 3. Configurer ton premier client
/setup-client mon-client

# 4. Lancer ton premier brief
/brief mon-client https://www.mon-site.fr/ma-page
```

## Comment ca marche

### Pipeline brief-to-publish

```
/brief                /redaction              /publish
  |                      |                      |
  v                      v                      v
Extraction data     Redaction SEO         Publication WP
(4 vagues MCP)      (3 etapes gatees)     (backup + verify)
  |                      |                      |
  v                      v                      v
Diagnostic +        Contenu pret         Page live +
checkpoint humain   a publier            journal mis a jour
```

### Rituels quotidiens

```
Lundi matin          Chaque matin
/week-plan           /morning
  |                    |
  v                    v
Bilan S-1 +         Taches + emails +
objectifs semaine   plan du jour
```

### Principes

- **Data first** — jamais de donnees inventees. Si un appel MCP echoue, c'est note "indisponible".
- **Checkpoints humains** — les phases critiques (diagnostic, strategie) requierent une validation.
- **Multi-clients** — chaque skill accepte un argument `client`. Le contexte se charge automatiquement.
- **Tracabilite** — chaque action est loggee dans le journal du client.

## SOPs incluses

Le dossier `consignes/` contient les procedures completes :

| Document | Ce qu'il contient |
|----------|------------------|
| `sop-brief-seo-v3.md` | Procedure complete de brief SEO : 4 vagues d'extraction, diagnostic, auto-skip/validation humaine |
| `architecture-brief-seo.md` | Architecture 3 couches (JSON schema, pipeline, rendu) — format compact max 8 pages |
| `brief-seo-schema.json` | Schema JSON du brief — contrat de donnees avec contraintes anti-inflation |
| `maillon2-consigne-redacteur.md` | Transformation brief JSON en consigne redacteur actionnable (10 blocs) |

## Adapter a ton workflow

1. **Fork ce repo**
2. **`/setup-client`** pour configurer tes clients
3. **Modifie les SOPs** dans `consignes/` pour les adapter a ta methodologie
4. **Ajoute tes propres skills** dans `.claude/commands/`

Les skills sont des fichiers Markdown — pas de code, pas de dependances. Tu peux les modifier, les combiner, en creer de nouveaux.

## Licence

MIT — utilise, modifie, partage librement.
