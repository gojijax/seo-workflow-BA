# Architecture brief SEO : données, rendu, format

Version : 3.0
Date : 17 février 2026

---

## Le problème

Sans contraintes formelles, le brief dérive : 50 pages, pas de standard, impossible de comparer deux briefs entre eux, impossible d'automatiser.

## La solution : 3 couches séparées

```
┌─────────────────────────────────────────────┐
│  COUCHE 1 — JSON schema (contrat de data)   │
│  brief-seo-schema.json                      │
│  → garantit que les mêmes champs existent   │
│    dans chaque brief, même structure,       │
│    mêmes types, mêmes contraintes           │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  COUCHE 2 — Pipeline d'extraction (SOP)     │
│  sop-brief-seo-amelioration-v3.md           │
│  → remplit le JSON via les appels MCP       │
│  → Phase 1 : extraction brute              │
│  → Phase 2 : diagnostic + remplissage JSON │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  COUCHE 3 — Template de rendu (ce doc)      │
│  → transforme le JSON en brief lisible      │
│  → format compact, max 8 pages             │
│  → intro executive en page 1               │
└─────────────────────────────────────────────┘
```

---

## Couche 1 — JSON schema : le contrat

Fichier : `brief-seo-schema.json`

### Pourquoi un JSON

Le JSON est la source de vérité. Le brief docx n'est qu'un rendu. Avantages :

- **Reproductibilité** — deux consultants sur la même URL produisent le même JSON (mêmes appels, mêmes données)
- **Validation automatique** — on peut vérifier qu'un brief est complet avant de le rendre (tous les champs requis remplis)
- **Historique** — on stocke les JSON, on compare brief N vs brief N-1 sur la même URL
- **Multi-rendu** — du même JSON on peut sortir un docx, un Google Doc, un dashboard Notion, un ticket Jira
- **Contraintes intégrées** — `maxItems: 10` sur les actions empêche l'inflation, `maxLength: 600` sur les réponses FAQ force la concision

### Sections du schema

| Section JSON | Ce qu'elle contient | Contrainte clé |
|-------------|--------------------|-|
| `meta` | URL, domaine, date, version SOP | Traçabilité |
| `contexte` | Persona cible, ton de voix, niveau technique, contraintes marque | **Obligatoire — guide le rédacteur** |
| `diagnostic` | Tableau diagnostic Phase 2 complet | Champ `verdict` obligatoire |
| `actions` | Recommandations priorisées | **Max 10 actions**, ref concurrent optionnelle |
| `title_meta` | Propositions title + meta desc | Longueurs calculées |
| `structure_hn` | Diff Hn actuelle vs recommandée | Statut par ligne (inchangé/modifié/ajout) |
| `faq` | Questions + réponses pré-écrites | **Max 8 questions, 600 car. par réponse** |
| `assets_recommandes` | Images, vidéos, tableaux, calculateurs à créer/intégrer | **Max 6 assets**, ref concurrent par asset |
| `ne_pas_toucher` | Éléments à préserver | Filet de sécurité |
| `technique` | Schemas, CWV éditoriaux, alertes dev | Séparé du rédactionnel |
| `metriques_succes` | KPI post-publication | **Max 6 KPI** |
| `donnees_brutes` | Livrables Phase 1 complets | **Annexe uniquement — jamais dans le brief** |

### Contraintes anti-inflation

```
actions              → maxItems: 10   (pas 25 recommandations noyées)
faq                  → maxItems: 8    (pas un FAQ de 30 questions)
assets_recommandes   → maxItems: 6    (les formats à vrai delta SERP)
metriques_succes     → maxItems: 6    (pas 15 KPI impossibles à suivre)
reponse FAQ          → maxLength: 600 (50-100 mots, pas des pavés)
```

---

## Couche 3 — Rendu compact : max 8 pages

### Principe fondamental

> Le rédacteur lit la page 1 et sait quoi faire.
> Les pages 2-5 lui disent comment.
> Les pages 6-8 sont la référence technique.
> Les données brutes sont en annexe JSON, pas dans le doc.

### Structure du brief rendu

```
PAGE 1 ─────────────────────────────────────────
  INTRO EXECUTIVE (le "dashboard papier")
  → lu en 30 secondes, décision immédiate
  → inclut le contexte éditorial (persona, ton, niveau)

PAGES 2-3 ──────────────────────────────────────
  ACTIONS PRIORISÉES (tableau + fiches courtes)
  → le quoi faire, dans quel ordre
  → chaque fiche P0/P1 inclut la ref concurrent

PAGE 4 ─────────────────────────────────────────
  TITLE, META, STRUCTURE Hn
  → les modifications exactes

PAGE 5 ─────────────────────────────────────────
  FAQ + ASSETS MÉDIA
  → contenus pré-rédigés + assets à créer

PAGES 6-7 ──────────────────────────────────────
  BRIEF TECHNIQUE
  → schemas, CWV, checklist

PAGE 8 ─────────────────────────────────────────
  KPI + SUIVI
  → métriques de succès, calendrier
```

---

### Page 1 — Intro executive (format exact)

C'est la pièce critique. Le consultant ou le rédacteur ouvre le doc, lit cette page, et sait :
- où on en est
- quel est le potentiel
- quelles sont les 3 actions prioritaires
- quels risques existent

```
╔══════════════════════════════════════════════════════╗
║  BRIEF SEO AMÉLIORATION                             ║
║  [URL cible]                                        ║
║  [date] — SOP v3.0                                  ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  MOT-CLÉ PRINCIPAL : [keyword]                       ║
║  Position : [X] — Volume : [XXXX]/mois — CPC : [X]€ ║
║                                                      ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  SITUATION EN UN COUP D'ŒIL                          ║
║  ┌──────────────────┬──────────┬────────────┐        ║
║  │ Position MC      │ [X]      │ [signal]   │        ║
║  │ CTR moyen URL    │ [X.X]%   │ [signal]   │        ║
║  │ Gap keywords     │ [X] kw   │ +[XXXX] vol│        ║
║  │ Entités manquantes│ [X]     │ [signal]   │        ║
║  │ Featured snippet │ [statut] │ [signal]   │        ║
║  │ Intent match     │ [statut] │ [signal]   │        ║
║  │ SERP             │ [statut] │ [signal]   │        ║
║  │ CWV Performance  │ [XX/100] │ [signal]   │        ║
║  └──────────────────┴──────────┴────────────┘        ║
║                                                      ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  POTENTIEL ESTIMÉ                                    ║
║  Trafic additionnel : +[XXX] à +[XXX] clics/mois    ║
║  (basé sur gap keywords × CTR estimé par position)   ║
║                                                      ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  TOP 3 ACTIONS PRIORITAIRES                          ║
║  P0 ● [Action 1] — impact [X], effort [X]           ║
║  P0 ● [Action 2] — impact [X], effort [X]           ║
║  P1 ● [Action 3] — impact [X], effort [X]           ║
║                                                      ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  ALERTES                                             ║
║  [cannibalisation / intent shift / saisonnalité /    ║
║   CWV bloquant / "Aucune alerte"]                    ║
║                                                      ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  RECOMMANDATION                                      ║
║  "[recommandation_machine — 2-4 phrases]"            ║
║                                                      ║
║  NE PAS TOUCHER :                                    ║
║  [éléments à préserver, 1 ligne par élément]         ║
║                                                      ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  CONTEXTE ÉDITORIAL                                  ║
║  Persona : [persona_cible]                           ║
║  Ton : [tone_of_voice]                               ║
║  Niveau : [niveau_technique]                         ║
║  Contraintes : [contraintes_marque ou "Aucune"]      ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

### Règle : cette page 1 est générée automatiquement depuis `diagnostic` + les 3 premières `actions` triées par priorité. Aucune rédaction manuelle.

**Définitions dashboard :**
- **CTR moyen URL** = CTR moyen pondéré de l'URL sur toutes les requêtes (depuis GSC, `diagnostic.ctr_moyen`). C'est le CTR global de la page, pas celui d'un seul keyword. Exemple : si le MC a un CTR de 11% mais que 20 autres requêtes ont un CTR de 0.5%, le CTR moyen URL sera bien plus bas (~1-2%).
- **Position MC** = position Haloscan du MC principal (pas la position GSC, qui est une moyenne mobile).

---

### Pages 2-3 — Actions priorisées

**Tableau récapitulatif** (depuis `actions[]`) :

| # | Action | Angle/intention | Type | Mots | Impact | Effort | Priorité |
|---|--------|----------------|------|------|--------|--------|----------|

Puis **1 fiche par action P0/P1** (les P2 restent en tableau uniquement) :

```
ACTION [X] : [nom]                          [P0/P1]
────────────────────────────────────────────
Emplacement : [où dans la page]
Angle : [comment aborder + format attendu]
Directive : [quoi faire, max 3 phrases]
Keywords : [liste]
Entités : [liste]
Justification : [data source — gap kw, PAA, CTR]
Ref concurrent : [URL] — [ce qu'ils font bien + en quoi faire mieux]
```

**Contrainte** : une fiche = max 9 lignes (8 + ref concurrent optionnelle). Pas de paragraphes explicatifs. Si le rédacteur a besoin de plus de contexte, il consulte les données brutes en annexe JSON.

---

### Page 4 — Title, meta, structure Hn

**Title et meta** : tableau actuel → proposé (depuis `title_meta`)

**Structure Hn** : diff visuel (depuis `structure_hn`)
- Lignes noires = inchangé
- Lignes bleues = modifié
- Lignes vertes = ajout
- ~~Lignes barrées~~ = supprimé

---

### Page 5 — FAQ + assets média

**FAQ** (depuis `faq[]`) : questions + réponses pré-écrites, prêtes à copier-coller.

**Assets média** (depuis `assets_recommandes[]`) : tableau des assets à créer ou intégrer, avec le type (image, vidéo, tableau, calculateur...), l'emplacement dans la page, et la référence concurrent quand disponible. Ce tableau comble le gap multimédia identifié lors de l'analyse concurrentielle.

```
| Type | Action | Description | Emplacement | Ref concurrent |
|------|--------|------------|-------------|----------------|
| Tableau | Créer | Comparatif prix par type | Après H2 "Tarifs" | [URL conc.] |
| Vidéo | Embed | Tuto pas-à-pas 3 min | Après H2 "Comment faire" | [URL conc.] |
```

**Propositions de contenu unique** (depuis `angles_uniques[]`, optionnel) : si des angles différenciants ont été identifiés, les présenter dans un encadré séparé. Ce sont des suggestions pour le rédacteur, pas des directives. Ne comptent pas dans les 10 actions max. Si aucun angle identifié → section absente.

---

### Pages 6-7 — Brief technique

Depuis `technique` :
- Schemas JSON-LD recommandés (type + champs clés, pas de code complet)
- CWV leviers éditoriaux (tableau problème/action/métrique)
- Alertes dev (hors périmètre éditorial, à remonter)
- Checklist d'implémentation

---

### Page 8 — KPI et suivi

Depuis `metriques_succes[]` : tableau objectif/métrique/cible/délai.
Calendrier de suivi : J+7, J+14, J+30, J+45.

---

## Workflow complet d'exécution

```
ÉTAPE 1 — Extraction (Phase 1 SOP)
│  Appels MCP séquentiels → données brutes
│  Résultat : objet donnees_brutes rempli
│
ÉTAPE 2 — Diagnostic (Phase 2 SOP)
│  Compilation des données brutes → diagnostic
│  Résultat : sections diagnostic + actions + title_meta + etc. remplies
│  → JSON complet validé contre le schema
│
ÉTAPE 3 — Validation
│  Si auto-skip → passage direct à l'étape 4
│  Si validation_humaine → présentation du diagnostic,
│    attente confirmation, ajustement si nécessaire
│
ÉTAPE 4 — Rendu
│  JSON → brief docx via template
│  8 pages max, intro executive en page 1
│  Données brutes en annexe JSON séparée (pas dans le docx)
```

---

## Règles de compacité

| Règle | Contrainte | Pourquoi |
|-------|-----------|---------|
| Max 10 actions | Schema JSON `maxItems: 10` | Au-delà, le rédacteur ne priorise plus |
| Max 8 FAQ | Schema JSON `maxItems: 8` | Les 8 PAA les plus pertinentes suffisent |
| Max 6 assets média | Schema JSON `maxItems: 6` | On cible les formats à vrai impact SERP |
| Max 6 KPI | Schema JSON `maxItems: 6` | Pas 15 KPI impossibles à suivre |
| Fiche action = 8 lignes max | Template de rendu | Pas de pavés explicatifs |
| Ref concurrent par action | Schema JSON (optionnel) | Le rédacteur voit l'exemple concret |
| FAQ réponse = 100 mots max | Schema JSON `maxLength: 600` | Pré-rédigé, pas un article |
| Données brutes = annexe JSON | Architecture | Jamais dans le brief docx |
| P2 = tableau seul, pas de fiche | Template de rendu | Effort minimal sur les actions secondaires |
| Brief technique = leviers éditoriaux | SOP v2.1 | Le dev a son propre scope |

---

## Validation du JSON avant rendu

Avant de générer le docx, vérifier :

```
checklist_pre_rendu:
  - [ ] meta.url_cible renseigné
  - [ ] contexte.persona_cible renseigné
  - [ ] contexte.tone_of_voice renseigné
  - [ ] diagnostic.verdict renseigné
  - [ ] diagnostic.recommandation_machine non vide
  - [ ] actions[] contient au moins 1 action P0
  - [ ] actions[] contient max 10 actions
  - [ ] actions P0 ont un angle_intention non vide
  - [ ] title_meta.title_propose non vide
  - [ ] structure_hn.recommandee non vide
  - [ ] technique non vide
  - [ ] metriques_succes contient au moins 2 KPI
  - [ ] si intent_match = "alerte_pivot" → action de pivot présente dans actions[]
  - [ ] si cannibalisation.keywords_exclus > 0 → mentionné dans diagnostic.recommandation_machine
  - [ ] si concurrents ont des assets média (vidéo, calculateur) et pas nous → assets_recommandes non vide
```

Si un check échoue → le brief n'est pas généré, erreur explicite.
