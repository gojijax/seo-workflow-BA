# SOP : création de brief SEO d'amélioration de contenu existant

Version : 3.0
Date : 17 février 2026
Auteur : [Consultant SEO]

---

## Objectif

Cette procédure produit un brief SEO d'amélioration à partir d'une URL existante. Le process est industrialisable : une seule pipeline d'extraction quelle que soit la situation, un diagnostic synthétique avec validation humaine (ou auto-skip si cas évident), puis génération du brief final.

## Principes fondamentaux

1. **Data first, interprétation ensuite** — chaque phase d'extraction produit des données brutes avant toute recommandation
2. **Un seul pipeline** — l'extraction est toujours la même, seul le brief final s'adapte au diagnostic
3. **Format standardisé** — même structure de brief quel que soit le sujet
4. **Priorisation systématique** — chaque action est classée impact/effort/priorité
5. **Conservatisme par défaut** — on enrichit l'existant, on ne casse pas ce qui fonctionne
6. **Raisonner, documenter, dévier** — chaque règle de cette SOP a une méthode par défaut. Un agent (humain ou IA) peut dévier de la méthode par défaut si un signal fort le justifie, à condition de **documenter la déviation et sa justification** dans le livrable concerné. Le problème n'est jamais de dévier, c'est de dévier sans laisser de trace.

## Entrées requises

| Entrée | Obligatoire | Exemple |
|--------|:-----------:|---------|
| URL cible | Oui | `https://www.example.com/blog/article-exemple/` |
| Propriété GSC | Oui | `sc-domain:example.com` OU `https://www.example.com/` |
| Domaine (pour Haloscan/DataForSEO) | Oui | `example.com` |
| Pays/langue cible | Oui | France / fr |

**⚠️ Format propriété GSC :** la propriété GSC peut être au format domaine (`sc-domain:example.com`) ou au format URL (`https://www.example.com/`). Les deux formats sont courants. Si l'opérateur ne connaît pas le format exact : commencer par lister les propriétés GSC disponibles (endpoint `sites.list`) et utiliser le `siteUrl` retourné tel quel. Ne jamais deviner le format — un mauvais `siteUrl` fait échouer silencieusement les requêtes.

## Outils et répartition des rôles

| Outil | Rôle principal | Données extraites |
|-------|----------------|-------------------|
| **GSC** | Données internes de performance | Impressions, clics, CTR par requête (90 jours + 12 mois saisonnalité) |
| **Haloscan** | Données keyword, positions, gap analysis (référence FR) | Volumes, CPC, positions organiques, SERP organique (top 100), PAA (`related_question`), keywords d'une URL (`page_best_keywords`), keywords similaires/match/synonymes, concurrents domaine, historique positions |
| **DataForSEO** | Données techniques, on-page et SERP features | CWV (Lighthouse), on-page parsing (structure Hn, metas, nombre de mots, contenu textuel), SERP features uniquement (featured snippets, vidéos, images, local pack, knowledge panel), intent classification |

**Règles de non-redondance :**
- **Positions organiques** → Haloscan (meilleure data FR), jamais GSC
- **SERP organique** (quelles URLs rankent, top 10/100) → Haloscan `keywords_overview` avec `requested_data: ["serp"]`
- **SERP features** (featured snippet, PAA positions dans la SERP, vidéos, images, local pack) → DataForSEO `serp_organic_live_advanced`
- **PAA questions** (texte des questions + volumes) → Haloscan `keywords_overview` avec `requested_data: ["related_question"]`
- **Gap analysis keywords** → Haloscan `page_best_keywords` en premier choix, DataForSEO `page_intersection` en fallback
- **Contenu textuel des pages** (scraping, entités, E-E-A-T) → DataForSEO `on_page_content_parsing`
- **Impressions et clics** → GSC uniquement

---

## Phase 1 — Extraction brute de l'existant

Cette phase est systématique et identique pour chaque brief. Elle produit des tableaux de données brutes sans interprétation.

### Graphe de dépendances et séquencement

Les sous-étapes ont des dépendances entre elles. Un agent IA ou un opérateur humain **doit** respecter l'ordre suivant. Les étapes d'une même vague n'ont pas de dépendance entre elles et peuvent être exécutées dans n'importe quel ordre.

**⚠️ Parallélisation technique :** les étapes d'une même vague n'ont pas de dépendance entre elles et peuvent être exécutées dans n'importe quel ordre. Si l'agent IA peut paralléliser les appels MCP (ex: appels simultanés), c'est autorisé. **Attention :** si des sous-agents/workers de fond sont utilisés pour paralléliser, vérifier qu'ils ont bien accès aux MCP avant de lancer. En cas d'échec (timeout, erreurs de permissions), basculer sur l'exécution séquentielle dans le contexte principal. Ne jamais relancer en boucle un mécanisme de parallélisation qui échoue.

```
VAGUE 1 (aucune dépendance — parallélisable) :
  1a — Scraping technique page cible
  1b — Performance GSC (90j + 12 mois)
  1f — CWV Lighthouse

VAGUE 2 (dépend de vague 1) :
  1c — Keyword MC principal     ← dépend de 1b (MC identifié via GSC)
       Appel unique Haloscan keywords_overview avec requested_data complet
       Produit : volumes, SERP top 100, PAA, synonymes, long-tail
       ⚠️ C'est 1c qui fournit la liste des URLs concurrentes (top 10 SERP)
       pour toutes les phases suivantes

VAGUE 3 (dépend de vague 2) :
  1d — Gap analysis + entités   ← dépend de 1c (URLs concurrentes top 10 + keywords)
       Haloscan page_best_keywords sur chaque concurrent
  1e — SERP features + intent   ← dépend de 1c (MC + URLs top 10 pour classification)
       DataForSEO serp_organic_live_advanced (features uniquement)
       Classification intent sur top 10 depuis données Haloscan 1c
  1g — Structure + E-E-A-T      ← dépend de 1c (3 URLs concurrentes sélectionnées)
       DataForSEO on_page scraping sur 3 concurrents

VAGUE 4 (dépend de vague 3) :
  1h — Cannibalisation          ← dépend de 1d (gap keywords identifiés)
```

**Changement vs v2.1 :** 1e passe en vague 3 (avec 1d et 1g) au lieu de vague 2. Raison : la classification intent se fait sur les URLs du top 10, qui viennent de 1c. DataForSEO `serp_organic_live_advanced` n'est plus utilisé pour les positions organiques (c'est Haloscan via 1c), uniquement pour les features enrichies (featured snippet, PAA positions, vidéos, images, local pack).

**Règle absolue :** ne jamais lancer une vague N+1 avant que la vague N soit complète. Un agent IA qui tente de paralléliser 1e avec 1b va chercher la SERP d'un MC qu'il ne connaît pas encore → données fausses ou hallucination.

**Gestion des erreurs entre vagues :** si un appel MCP d'une vague échoue, retenter une fois. Si l'échec persiste, noter "données indisponibles" pour cette étape et continuer avec les données disponibles. Ne jamais inventer de données pour compenser un échec.

### 1a. Scraping technique de la page cible

#### Extraction de la structure Hn (headings)

**Outil principal : Bash `curl` + parsing HTML**

Les pages example.com sont en server-side rendering (SSR) — les headings sont dans le HTML brut. On extrait directement via curl, c'est la méthode la plus fiable et la plus rapide.

**Commande d'extraction :**
```bash
curl -s -L "[URL]" | python3 -c "
import sys, re, json, html as htmlmod
raw = sys.stdin.read()
headings = re.findall(r'<(h[1-4])[^>]*>(.*?)</\1>', raw, re.IGNORECASE | re.DOTALL)
template = re.compile(r'rejoignez.nous|notre.soci|aide..sav|liens.utiles|produit.associ|d.couvrir.aussi|plaques.homologu|exp.dition.en.24|plaques.haut.de.gamme|service.client|panier|mon.compte|plaque.immatriculation.voiture.*homologu', re.IGNORECASE)
result = []
for tag, text in headings:
    clean = re.sub(r'<[^>]+>', '', text).strip()
    clean = htmlmod.unescape(clean)
    clean = re.sub(r'\s+', ' ', clean)
    if clean and not template.search(clean):
        result.append({'niveau': int(tag[1]), 'texte': clean[:150]})
print(json.dumps(result, indent=2, ensure_ascii=False))
"
```

Ce script :
1. Récupère le HTML brut de la page via curl
2. Extrait toutes les balises H1-H4 avec leur contenu (regex)
3. Nettoie les balises HTML internes (ex: `<span>`, `<strong>`)
4. Décode les entités HTML (`&#8217;` → `'`, `&nbsp;` → espace, etc.)
5. Filtre les headings du template site (footer, sidebar, CTA, produits) via regex
6. Retourne un JSON propre avec niveau + texte exact

**⚠️ Règle de validation Hn :**
- Si le résultat contient **< 3 H2 de contenu éditorial** → flag `⚠️ STRUCTURE HN SUSPECTE` dans le livrable
- **Interdit** de reconstruire/inférer un plan Hn à partir du contenu textuel. Si le curl échoue ou retourne 0 heading éditorial → noter `structure Hn indisponible` et ne pas continuer vers la Phase 3 sans validation humaine.
- Le résultat du curl est stocké tel quel dans `donnees_brutes.fiche_technique_page.htags_bruts` (cf. schéma JSON). C'est la preuve vérifiable du plan réel.

**Fallback si curl échoue (403, timeout, etc.) :**
1. DataForSEO `on_page_instant_pages` (champ `htags`) — appliquer le même filtre template
2. Si toujours insuffisant : Chrome `javascript_tool` (même logique de filtrage)
3. Dans tous les cas : noter la source dans `htags_source` et tout avertissement dans `htags_avertissement`

#### Données techniques complémentaires

**Outil : DataForSEO `on_page_instant_pages`**

Données à extraire (hors structure Hn, qui vient du curl) :
- Nombre de mots
- Title actuel (longueur en caractères)
- Meta description actuelle (longueur en caractères)
- URL canonical
- Meta robots
- Nombre de liens internes sortants
- Nombre de liens externes sortants
- Nombre d'images (avec/sans attribut alt)

**Outil : DataForSEO `on_page_content_parsing`**

Données complémentaires :
- Contenu textuel structuré de la page
- Liens internes avec ancres

**Livrable 1a : fiche technique de la page**

```
FICHE TECHNIQUE — [URL]
─────────────────────────
Title : [title actuel] ([XX] caractères)
Meta description : [meta actuelle] ([XX] caractères)
Canonical : [url]
Robots : [index/noindex, follow/nofollow]
Nombre de mots : [XXXX]
─────────────────────────
Structure Hn : [source : curl / DataForSEO fallback ⚠️ / Chrome fallback ⚠️]
H1 : [texte]
  H2 : [texte]
    H3 : [texte]
    H3 : [texte]
  H2 : [texte]
    H3 : [texte]
  [...]
─────────────────────────
Liens internes sortants : [XX]
Liens externes sortants : [XX]
Images : [XX] total, [XX] sans alt
```

### 1b. Performance GSC de l'URL

**Outil : GSC `search_analytics`**

**Étape préalable — découverte de la propriété GSC :**

Si la propriété GSC fournie en entrée fait échouer les requêtes (erreur 403, 404, ou résultats vides alors que la page existe) :
1. Appeler `sites.list` pour lister les propriétés GSC accessibles
2. Identifier la bonne propriété (celle qui correspond au domaine cible)
3. Utiliser le `siteUrl` exact retourné par l'API (ne pas modifier le format)
4. Continuer avec cette propriété pour toutes les requêtes GSC suivantes

**Étape préalable — vérification de l'âge de la page :**

Avant de lancer les requêtes 90 jours, vérifier si la page a au moins 90 jours d'existence dans l'index GSC :
- Lancer la requête 12 mois d'abord (requête 2 ci-dessous)
- Si la première date avec des données est < 90 jours : la page est trop récente pour les données 90 jours
- Dans ce cas : adapter la fenêtre de la requête 1 à la durée réelle d'existence (ex: page publiée il y a 45 jours → requête sur 45 jours)
- Mentionner dans le livrable : "Page publiée le [date] — données disponibles sur [X] jours (< 90 jours standard)"
- Le diagnostic Phase 2 devra nuancer les conclusions (volume d'impressions plus faible = normal, pas un signal de faiblesse)

**Requête 1 — Performance récente (90 jours ou durée réelle si page récente) :**
- URL cible en filtre page
- Période : 90 derniers jours
- Dimensions : query
- Tri : impressions décroissantes
- Limite : 50 requêtes

Données extraites par requête :
- Clics
- Impressions
- CTR
- (Position moyenne GSC ignorée volontairement — on utilise Haloscan pour ça)

**Requête 2 — Détection de saisonnalité (12 mois) :**
- URL cible en filtre page
- Période : 12 derniers mois
- Dimensions : date (par mois)
- Pas de filtre query (total URL)

Données extraites :
- Impressions mensuelles sur 12 mois
- Identification du pic saisonnier (mois le plus fort)
- Ratio pic/creux

**Règle saisonnalité :** si le ratio impressions du mois le plus fort / mois le plus faible est > 3x, la page est saisonnière. Dans ce cas, les données récentes doivent être interprétées avec précaution. Le diagnostic (Phase 2) mentionne explicitement la saisonnalité et recommande de comparer les performances à N-1 (même période année précédente) plutôt qu'au trimestre précédent. Si la page a moins de 6 mois de données, noter "saisonnalité non mesurable — données insuffisantes" et ne pas conclure.

**Identification du mot-clé principal (MC) :**

1. **Méthode par défaut** : regrouper les variantes proches (avec/sans accent, avec/sans article, singulier/pluriel, ordre des mots) en sommant leurs impressions. Sélectionner le groupe avec le plus d'impressions cumulées. Au sein du groupe, utiliser la forme avec le plus de volume Haloscan pour les appels suivants.

2. **Déviation autorisée si justifiée** : l'agent peut choisir un MC différent de la méthode par défaut si un signal fort le justifie (ex: meilleur CTR, meilleure position, meilleur alignement avec le H1/contenu de la page, volume Haloscan très supérieur). Dans ce cas, **documenter explicitement** dans le livrable 1b : "MC par défaut (impressions) : [X]. MC retenu : [Y]. Raison : [justification en 1-2 phrases]."

3. Si saisonnalité détectée et période en creux, vérifier aussi le MC sur la période haute de l'année précédente. Si page < 30 jours de données GSC, MC fragile — le mentionner.

**Principe général :** la SOP donne une méthode reproductible. L'agent peut raisonner et dévier, à condition de laisser une trace auditable. Le problème n'est jamais de dévier, c'est de dévier sans l'écrire.

**Identification des mots-clés secondaires :** requêtes avec impressions > 10% du MC principal.

**Analyse CTR :**
- CTR moyen de l'URL
- Requêtes à CTR anormalement bas (< 2% avec impressions significatives) → signal de problème title/meta
- Requêtes à CTR élevé (> 10%) → signal de bonne adéquation contenu/intention

**Livrable 1b : tableau de performance GSC**

```
PERFORMANCE GSC — [URL]
──────────────────────────────────────────
Propriété GSC utilisée : [siteUrl exact]
Âge de la page (dans l'index GSC) : [X] jours (depuis [date])
Fenêtre de données : [90 jours / X jours si page récente]
MC principal identifié : [requête] ([XXXX] impressions / [fenêtre])
CTR moyen URL : [X.XX]%
Saisonnalité : [OUI ratio Xx / NON / données insuffisantes si < 6 mois]
Mois pic : [mois] — Mois creux : [mois]
──────────────────────────────────────────
Performance 90 jours :
| Requête | Clics | Impressions | CTR |
|---------|-------|-------------|-----|
| [req 1] | XX    | XXXX        | X%  |
| [req 2] | XX    | XXXX        | X%  |
| [...]   | ...   | ...         | ... |
──────────────────────────────────────────
Courbe saisonnière (12 mois, impressions totales URL) :
| Mois | Impressions |
|------|-------------|
| [M-12] | XXXX |
| [M-11] | XXXX |
| [...] | ... |
──────────────────────────────────────────
Alertes CTR :
- CTR < 2% avec impressions > 100 : [liste requêtes]
- CTR > 10% : [liste requêtes]
```

### 1c. Données keyword du MC principal

**Outil : Haloscan `keywords_overview` — appel unique combiné**

Un seul appel avec `requested_data` complet pour éviter la multiplication des requêtes :

```json
{
  "keyword": "[MC principal identifié en 1b]",
  "requested_data": [
    "metrics",
    "keyword_match",
    "similar_highlight",
    "related_question",
    "similar_serp",
    "top_sites",
    "serp",
    "volume_history",
    "synonyms",
    "categories"
  ]
}
```

Données extraites depuis la réponse :
- `seo_metrics` + `ads_metrics` → volume, CPC, concurrence, KGR
- `serp` → SERP organique top 100 (URLs, positions, titles, descriptions) — **c'est cette liste qui fournit les URLs concurrentes pour les phases 1d, 1e et 1g**
- `related_question` → questions PAA associées avec volumes
- `keyword_match` → variations long-tail contenant le MC
- `similar_highlight` → keywords sémantiquement proches (plus gros volumes)
- `volume_history` → tendance de volume sur 12 mois
- `synonyms` → synonymes du MC (utile pour les recommandations rédactionnelles Phase 3)
- `top_sites` → domaines dominants sur la thématique

**Position actuelle de notre URL :** chercher notre URL dans les résultats `serp`. Si absente du top 100, compléter avec Haloscan `domains_positions` filtré sur le MC.

**Identification des URLs concurrentes (top 10) :** depuis le champ `serp`, prendre les 10 premières URLs. Exclure Wikipedia, YouTube, sites gouvernementaux, Wiktionary et tout résultat non pertinent (navigational, hors sujet). Ce sont ces URLs filtrées qui alimentent les phases suivantes.

**Livrable 1c : fiche keyword MC principal**

```
FICHE MOT-CLÉ PRINCIPAL
────────────────────────
Mot-clé : [MC]
Volume mensuel : [XXXX]
CPC : [X.XX]€
KGR : [X.XX]
Position actuelle (Haloscan) : [X]
────────────────────────
SERP organique top 10 :
| Pos | URL | Domaine | Pertinent pour gap |
|-----|-----|---------|:------------------:|
| 1   | ... | ...     | OUI/NON            |
| 2   | ... | ...     | OUI/NON            |
| ... | ... | ...     | ...                |
→ URLs concurrentes retenues : [X] sur 10
────────────────────────
PAA détectées (related_question) :
- [question 1] (vol : [XX])
- [question 2] (vol : [XX])
- [...]
────────────────────────
Keywords long-tail (keyword_match, top 10 par volume) :
- [kw 1] (vol : [XX])
- [kw 2] (vol : [XX])
────────────────────────
Synonymes identifiés :
- [syn 1] (vol : [XX])
- [syn 2] (vol : [XX])
```

### 1d. Analyse concurrents et gap analysis

**Outil principal : Haloscan `page_best_keywords`**

Pour chaque URL concurrente retenue en 1c (jusqu'à 10 URLs du top 10 SERP, après exclusion des non pertinentes) :
- Appeler `page_best_keywords` sur chaque URL concurrente → liste des keywords sur lesquels cette page ranke
- Comparer avec les keywords de notre page (obtenus via `page_best_keywords` sur notre URL ou via GSC)
- Identifier les **gap keywords** : keywords sur lesquels au moins 1 concurrent ranke et notre page ne ranke pas

**Outil complémentaire : Haloscan `keywords_highlights`**

Sur le MC principal, pour enrichir la liste de gap potentiels avec un score de similarité :
- `keywords_highlights` avec `volume_min=100` (ou `cpc_min=2`) → keywords sémantiquement proches du MC, filtrés directement par volume/CPC, avec **score de similarité** pour chaque résultat
- Ce score de similarité sert aussi à la vérification sémantique de l'auto-skip (Phase 2)
- `lineCount=50` pour couvrir large

**Outil complémentaire secondaire : Haloscan `find_keywords`**

Si `page_best_keywords` + `keywords_highlights` ne retournent pas assez de gap keywords (< 5 après filtrage) :
- `find_keywords` avec `keywords_sources: ["serp", "related", "highlights", "questions"]`, `volume_min=100`, `lineCount=30`
- Combine plusieurs stratégies de découverte en un seul appel

**Outil fallback : DataForSEO `dataforseo_labs_google_page_intersection`**

Si `page_best_keywords` Haloscan ne retourne pas suffisamment de données (< 5 keywords par concurrent), utiliser en complément :
- Intersection entre ma page et les pages concurrentes
- Keywords communs (où je suis et eux aussi)
- Keywords exclusifs concurrents (où eux sont et pas moi)

**Important :** ne jamais utiliser `domains_competitors_keywords_diff` au niveau domaine pour un gap analysis de page. Ça retourne les keywords du domaine entier (hors sujet garanti sur les sites généralistes). Toujours travailler au niveau **URL/page**.

**Filtrage des gap keywords :**
- Volume > 100/mois → opportunité trafic
- OU CPC > 2€ (quel que soit le volume) → opportunité business

**Analyse des entités (gap sémantique) :**

Au-delà des mots-clés, identifier les entités nommées présentes dans le contenu des concurrents mais absentes de notre page :
- Noms de marques, produits, services mentionnés
- Lieux géographiques spécifiques
- Concepts techniques ou termes métier
- Noms propres (personnes, organisations, normes, lois)
- Données chiffrées récurrentes (prix, dates, statistiques)

**Méthode :** comparer le contenu textuel (issu du scraping DataForSEO `on_page_content_parsing`) de notre page vs les 3 concurrents les mieux positionnés. Lister les entités récurrentes chez au moins 2 concurrents sur 3 et absentes chez nous.

**Fallback si scraping concurrent échoue :** si `on_page_content_parsing` échoue sur un ou plusieurs concurrents, l'analyse d'entités est dégradée. Dans ce cas :
- Utiliser les titles + descriptions SERP (depuis 1c) pour identifier les thèmes couverts par les concurrents
- Utiliser les keywords de `page_best_keywords` (Haloscan) de chaque concurrent comme proxy des entités couvertes
- Mentionner dans le livrable : "Analyse entités dégradée — scraping bloqué sur [X] concurrents sur 3"
- Ne jamais inventer des entités pour compenser

**Livrable 1d : tableau de gap analysis + entités**

```
GAP ANALYSIS — [URL cible] vs concurrents top 10
─────────────────────────────────────────────────
URLs concurrentes analysées : [X] (depuis SERP top 10 de 1c)
Gap keywords identifiés : [XX] (après filtrage vol > 100 OU CPC > 2€)
Volume cumulé gap : [XXXXX]/mois
Source gap : Haloscan page_best_keywords [+ DataForSEO page_intersection si utilisé]
─────────────────────────────────────────────────
| Keyword gap | Volume | CPC | Type | Concurrent(s) qui ranke(nt) |
|-------------|--------|-----|------|----------------------------|
| [kw 1]      | XXXX   | X€  | Trafic | [concurrent 1] pos X       |
| [kw 2]      | 30     | 4.5€| Business | [concurrent 2] pos X     |
| [...]       | ...    | ... | ...    | ...                        |
─────────────────────────────────────────────────
Keywords communs (déjà positionnés) :
| Keyword | Mon pos | Meilleur concurrent | Sa pos |
|---------|---------|---------------------|--------|
| [kw]    | X       | [concurrent]        | X      |
─────────────────────────────────────────────────
Entités manquantes (présentes chez 2+ concurrents top 3, absentes chez nous) :
| Entité | Type | Présent chez | Contexte d'utilisation |
|--------|------|-------------|----------------------|
| [entité] | Marque | Conc. 1, Conc. 3 | [dans quelle section/contexte] |
| [entité] | Lieu | Conc. 1, Conc. 2, Conc. 3 | [...] |
| [entité] | Concept | Conc. 2, Conc. 3 | [...] |
```

### 1e. Analyse SERP features et intent shift

**SERP features uniquement → DataForSEO `serp_organic_live_advanced`**

Sur le MC principal :
- Présence/absence de featured snippet (et format : paragraphe, liste, tableau)
- PAA (People Also Ask) — positions dans la SERP et questions affichées
- Résultats enrichis (vidéo, images, local pack, knowledge panel)
- Type de résultats dominants
- Sitelinks, indented results

**Note :** la liste des URLs organiques et leurs positions vient de Haloscan (Phase 1c, champ `serp`). DataForSEO ici ne sert qu'à identifier les **features enrichies** de la SERP, pas les résultats organiques eux-mêmes.

**Intent classification → analyse manuelle sur le top 10 complet**

Classifier chaque URL du **top 10** SERP (depuis les données Haloscan 1c) par type de page :
- Informationnelle (guide, blog, tutoriel, wiki)
- Transactionnelle (e-commerce, fiche produit, comparateur prix)
- Navigationnelle (page officielle, portail)
- Locale (annuaire, Google Maps)

Puis comparer avec le type de ma page.

**Règle de décision intent shift (basée sur le top 10, pas 5) :**

| Ma page | SERP dominante (6+ résultats sur 10) | Verdict |
|---------|--------------------------------------|---------|
| Info (blog/guide) | Info | OK — optimisation standard |
| Info (blog/guide) | Transac (e-commerce) | ALERTE PIVOT — la SERP a basculé, optimiser le contenu ne suffira pas |
| Info (blog/guide) | Mixte (4-5 info + 4-5 transac) | VIGILANCE — adapter le contenu avec des éléments transactionnels (tableaux prix, CTA, comparatifs) |
| Transac | Transac | OK — optimisation standard |
| Transac | Info | ALERTE PIVOT — envisager un contenu plus éducatif ou créer une page complémentaire |

**Seuil :** "dominante" = 6+ résultats sur 10 du même type. Entre 4 et 6 = mixte → VIGILANCE. En-dessous de 4 = le type minoritaire ne domine pas.

**Si le label DataForSEO (via `dataforseo_labs_search_intent`) contredit l'analyse manuelle du top 10 :** la SERP effective (top 10) prime toujours. Conserver le label DataForSEO dans le diagnostic avec mention "overridé — justification : [raison basée sur le top 10 réel]". Cela laisse une trace auditable.

**Si ALERTE PIVOT détectée :** le diagnostic Phase 2 bloque l'auto-skip et force la validation humaine. La recommandation machine propose un changement de format de page (pas juste une optimisation de contenu). L'option "abandonner ce MC et pivoter vers un MC secondaire mieux aligné" est explicitement mentionnée.

**Livrable 1e : fiche SERP features + intent**

```
ANALYSE SERP — "[MC principal]"
────────────────────────────────
Intent SERP dominant : [Informationnel / Transactionnel / Mixte / Local]
Type de ma page : [Informationnel / Transactionnel / ...]
Match intent : [OK / VIGILANCE / ALERTE PIVOT]
Label DataForSEO : [label] (conforme / overridé — [raison])
────────────────────────────────
Classification top 10 :
| Pos | URL | Type page | Intent |
|-----|-----|-----------|--------|
| 1   | ... | Wiki       | Info   |
| 2   | ... | E-commerce | Transac |
| 3   | ... | Blog       | Info   |
| 4   | ... | Guide      | Info   |
| 5   | ... | Comparateur| Transac |
| 6   | ... | Blog       | Info   |
| 7   | ... | E-commerce | Transac |
| 8   | ... | Guide      | Info   |
| 9   | ... | E-commerce | Transac |
| 10  | ... | Blog       | Info   |
→ SERP dominante : Informationnelle (6/10)
────────────────────────────────
Featured Snippet : [OUI format X / NON = opportunité / NON = pas pertinent]
PAA : [OUI — X questions] / [NON]
Résultats vidéo : [OUI pos X / NON]
Images : [OUI / NON]
Local pack : [OUI / NON]
Knowledge panel : [OUI / NON]
Sitelinks : [OUI — quel domaine / NON]
────────────────────────────────
SERP features détaillées :
- FS : [description si présent, ou "Absent = opportunité à capter"]
- PAA questions (depuis la SERP DataForSEO) :
  1. [question]
  2. [question]
  [...]
```

### 1f. Audit technique CWV

**Outil : DataForSEO `on_page_lighthouse`**

Données extraites :
- Score Performance
- Score Accessibilité
- Score SEO
- LCP (Largest Contentful Paint)
- FID/INP (Interaction to Next Paint)
- CLS (Cumulative Layout Shift)
- Principales recommandations si score < 90

**Livrable 1f : fiche technique CWV**

```
CORE WEB VITALS — [URL]
───────────────────────
| Métrique | Valeur | Statut |
|----------|--------|--------|
| Performance | XX/100 | ✅/⚠️/🔴 |
| Accessibilité | XX/100 | ✅/⚠️/🔴 |
| SEO | XX/100 | ✅/⚠️/🔴 |
| LCP | X.Xs | ✅/⚠️/🔴 |
| INP | XXms | ✅/⚠️/🔴 |
| CLS | X.XX | ✅/⚠️/🔴 |
───────────────────────
Problèmes critiques (si score < 90) :
- [problème 1]
- [problème 2]
```

### 1g. Structure Hn et signaux E-E-A-T des concurrents top 3

**Sélection des 3 concurrents à scraper :** parmi les URLs concurrentes retenues en 1c (top 10 filtré), choisir les 3 les plus pertinentes pour la comparaison. Critères de sélection : même type de page (info vs info, transac vs transac), meilleure position, domaine le plus comparable au nôtre (exclure les mastodontes type Wikipedia si possible).

**Outil : DataForSEO `on_page_instant_pages`** (sur chaque URL concurrente)

Pour les 3 premières URLs de la SERP :
- Structure Hn complète
- Nombre de mots
- Title et meta description

**⚠️ Fiabilité des htags concurrents :** les htags retournés par `on_page_instant_pages` sur les sites concurrents peuvent être incomplets si le contenu est rendu en JavaScript (même problème que pour notre page, cf. 1a). Pour les concurrents, on n'a pas accès à Chrome → accepter cette limitation mais **la documenter** :
- Vérifier que les htags retournés contiennent au moins 3 H2 de contenu éditorial (pas seulement template/footer)
- Si les htags semblent pauvres ou ne contiennent que du template → noter `⚠️ Hn concurrent potentiellement incomplet (rendu JS)` dans le livrable
- Ne jamais compléter/reconstruire un plan Hn concurrent par inférence du texte brut

**Outil : DataForSEO `on_page_content_parsing`** (sur chaque URL concurrente)

**⚠️ Fallback scraping concurrent :** DataForSEO `on_page_content_parsing` échoue fréquemment sur les URLs concurrentes (sites qui bloquent le scraping, timeouts, erreurs serveur). Si l'appel échoue après retry :
1. Essayer `on_page_instant_pages` sur la même URL (souvent plus robuste, donne au moins la structure Hn et les metas)
2. Si ça échoue aussi : utiliser les données déjà disponibles depuis le champ `serp` de Haloscan (1c) — title et description de chaque résultat SERP
3. Pour l'E-E-A-T : noter "scraping bloqué — E-E-A-T non évalué pour [URL]" et réduire le benchmark (comparer sur 2 concurrents au lieu de 3, ou sur les seuls signaux observables depuis la SERP : présence d'auteur dans le title, date dans la description)
4. Ne jamais inventer des signaux E-E-A-T pour compenser un échec de scraping

Signaux E-E-A-T à relever dans le contenu des concurrents :
- Auteur identifié (nom, bio, lien vers page auteur) → signal "Expertise"
- Date de publication et/ou de mise à jour visible → signal "Freshness"
- Sources citées (liens externes vers études, rapports, normes, institutions) → signal "Trustworthiness"
- Témoignages, retours d'expérience, cas concrets → signal "Experience"
- Certifications, mentions presse, affiliations professionnelles → signal "Authoritativeness"

**Comparaison E-E-A-T : ma page vs concurrents**

Si les 3 concurrents top 3 ont tous un auteur identifié + des sources citées et que ma page n'en a pas, c'est un content gap critique indépendant des mots-clés.

**Livrable 1g : comparatif structure + E-E-A-T**

```
COMPARATIF STRUCTURE — MC : "[MC principal]"
──────────────────────────────────────────────
| Élément | Mon URL | Concurrent 1 | Concurrent 2 | Concurrent 3 |
|---------|---------|-------------|-------------|-------------|
| Mots | XXXX | XXXX | XXXX | XXXX |
| H2 | X | X | X | X |
| H3 | X | X | X | X |
| H4 | X | X | X | X |
| Title (car.) | XX | XX | XX | XX |
──────────────────────────────────────────────
Structure Hn concurrent 1 — [domaine] :
  H1 : [texte]
    H2 : [texte]
      H3 : [texte]
  [...]

Structure Hn concurrent 2 — [domaine] :
  [...]

Structure Hn concurrent 3 — [domaine] :
  [...]
──────────────────────────────────────────────
SIGNAUX E-E-A-T
──────────────────────────────────────────────
| Signal | Mon URL | Conc. 1 | Conc. 2 | Conc. 3 |
|--------|---------|---------|---------|---------|
| Auteur identifié | OUI/NON | OUI/NON | OUI/NON | OUI/NON |
| Date mise à jour visible | OUI/NON | OUI/NON | OUI/NON | OUI/NON |
| Sources/études citées | X liens | X liens | X liens | X liens |
| Témoignages/expérience | OUI/NON | OUI/NON | OUI/NON | OUI/NON |
| Schema auteur/org | OUI/NON | OUI/NON | OUI/NON | OUI/NON |
──────────────────────────────────────────────
Gaps E-E-A-T critiques (présent chez 2+ concurrents, absent chez nous) :
- [signal manquant 1]
- [signal manquant 2]
```

### 1h. Audit de cannibalisation préventif

**Objectif :** vérifier AVANT la génération du brief qu'on ne va pas créer de conflit interne en ciblant des gap keywords déjà travaillés par une autre page du site.

**Outil : DataForSEO `dataforseo_labs_google_ranked_keywords`** (sur le domaine entier)
ou **Haloscan `get_domains_positions`** (filtre sur les gap keywords)

**Action :** pour chaque gap keyword identifié en 1d (volume > 100 OU CPC > 2€), vérifier si une autre URL du domaine ranke déjà dans le top 50.

**Règles de décision :**

| Situation | Action |
|-----------|--------|
| Aucune autre URL ne ranke sur ce gap keyword | Keyword validé → intégrable au brief |
| Une autre URL ranke en top 50, mais position > 20 et contenu faible | Keyword validé avec mention → envisager redirection ou fusion à terme |
| Une autre URL ranke en top 20 avec contenu dédié | Keyword exclu du brief → risque de cannibalisation |
| Plusieurs URLs du site rankent sur le même keyword | Alerte cannibalisation existante → traitement prioritaire hors brief |

**Livrable 1h : rapport de cannibalisation**

```
AUDIT CANNIBALISATION — [domaine]
─────────────────────────────────────────────────
Gap keywords testés : [XX]
Conflits détectés : [X]
─────────────────────────────────────────────────
| Gap keyword | Autre URL qui ranke | Sa position | Décision |
|-------------|--------------------|-----------:|----------|
| [kw 1]      | /page-existante/   | 18         | Validé (mention) |
| [kw 2]      | /autre-page/       | 8          | Exclu |
| [kw 3]      | —                  | —          | Validé |
| [...]       | ...                | ...        | ... |
─────────────────────────────────────────────────
Keywords exclus du brief (cannibalisation) : [liste]
Keywords avec alerte fusion : [liste + URLs concernées]
```

**1h bis — Analyse des pages satellites faibles (recommandation 301)**

Quand l'audit 1h identifie une ou plusieurs URL(s) conflictuelle(s), vérifier systématiquement si une **redirection 301** se justifie. Ne pas se contenter de "envisager à terme" — trancher maintenant avec des données.

**Méthode :**

1. **GSC (3 derniers mois)** : récupérer les clics totaux de la page satellite
2. **Requêtes croisées** : GSC dimensions=query pour chaque page → identifier les requêtes en commun + positions comparées
3. **Requêtes propres** : Haloscan `get_domains_overview` (best_keywords) sur la page satellite → identifier les requêtes qu'elle couvre et que la page cible ne couvre pas
4. **Produire un verdict argumenté** (pour/contre) avec les données

**Seuil de déclenchement :** si la page satellite fait **< 10 clics / 3 mois** sur des requêtes où la page cible du brief domine déjà → produire obligatoirement une recommandation 301.

**Livrable 1h bis : recommandation 301**

```
RECOMMANDATION 301 — [URL satellite] → [URL cible]
─────────────────────────────────────────────────
Clics page satellite (3 mois) : [X]
Clics page cible (3 mois) : [X]

Requêtes en commun :
| Requête | Page cible (clics / pos) | Page satellite (clics / pos) |
|---------|--------------------------|------------------------------|
| [kw]    | XX clics / pos X.X       | X clics / pos XX.X           |

Requêtes propres à la page satellite (à absorber avant 301) :
| Requête | Volume | Position satellite |
|---------|--------|--------------------|
| [kw]    | XXX    | X                  |

Arguments POUR la 301 : [liste]
Arguments CONTRE la 301 : [liste]
Prérequis avant 301 : [contenus à intégrer dans la page cible]
─────────────────────────────────────────────────
```

**Important :** cette recommandation doit être **signalée explicitement à l'opérateur humain** au moment du rendu du brief (Phase 3), dans un bloc dédié AVANT le brief SEO. Ne pas l'enterrer dans le JSON — c'est une décision stratégique qui nécessite une validation humaine.

**Important :** cette étape modifie la liste finale de gap keywords exploitables. Le diagnostic (Phase 2) utilise la liste filtrée post-cannibalisation, pas la liste brute de 1d.

---

## Phase 2 — Diagnostic synthétique

**⚠️ CHECKPOINT OBLIGATOIRE** — Cette phase est un point d'arrêt. Le diagnostic doit être affiché à l'opérateur humain avant toute génération de brief. Ce checkpoint n'est **jamais** court-circuitable, même si l'opérateur humain a demandé "exécute tout d'un coup" ou "lance sans attendre ma validation". La seule exception est l'auto-skip (voir conditions strictes ci-dessous).

Cette phase compile les données de la Phase 1 en un tableau de diagnostic rapide, suivi d'une recommandation.

### Tableau de diagnostic

```
DIAGNOSTIC — [URL cible]
═══════════════════════════════════════════════════
| Indicateur | Valeur | Signal |
|------------|--------|--------|
| MC principal | "[kw]" (vol : XXXX, CPC : X€) | — |
| Position actuelle (Haloscan) | X | Bonne/Moyenne/Faible |
| Intent match | [OK / VIGILANCE / ALERTE PIVOT] | Bloquant si PIVOT |
| CTR moyen URL (GSC) | X.X% | Bon/À optimiser |
| Saisonnalité | Ratio pic/creux : Xx | Saisonnière/Stable |
| Gap keywords validés (post-canni.) | X mots-clés, ~XXXX vol cumulé | Fort/Moyen/Faible potentiel |
| Similarité sémantique gap kw | Haute/Moyenne/Faible | Risque dilution si faible |
| Entités manquantes | X entités chez 2+ concurrents | À intégrer/— |
| Gaps E-E-A-T | X signaux manquants (auteur, sources...) | Critique/Mineur/— |
| Cannibalisation | X conflits détectés, X exclus | Alerte/OK |
| Featured snippet | Absent/Présent (concurrent)/Capturé | Opportunité/Défense/— |
| Volatilité SERP | Stable/Modérée/Forte | Effort continu/Effort ponctuel |
| Score CWV Performance | XX/100 | OK/À corriger |
| Delta mots vs concurrent #1 | +/-XXX mots | Suffisant/Insuffisant |
| Anomalies CTR | X requêtes CTR < 2% | Signal title-meta/— |
═══════════════════════════════════════════════════
```

**Volatilité SERP :** mesurée via Haloscan `get_keywords_serp_compare` (comparaison SERP à 2 dates espacées de 30 jours). Si plus de 3 URLs du top 10 ont changé en 30 jours → SERP volatile (QDF probable, effort continu nécessaire). Si le top 5 est identique → SERP figée (concurrents historiques, effort initial fort mais maintenance faible).

**CTR moyen URL ≠ CTR du MC.** Le CTR moyen URL est la moyenne pondérée de toutes les requêtes (depuis GSC). Ne pas utiliser le CTR d'une seule requête (même le MC) dans le dashboard — c'est trompeur si le MC capte 10% des impressions et les 90% restants ont un CTR de 0.5%.

### Recommandation machine

Une synthèse en 2-4 phrases qui résume la situation et oriente le brief :

> Exemple : "Position 2 sur MC principal (5400 vol), intent match OK (SERP info, ma page info). 7 gap keywords validés (~7500 vol cumulé, 1 exclu pour cannibalisation, similarité sémantique haute). 4 entités manquantes. 2 gaps E-E-A-T : pas d'auteur identifié ni de sources citées vs 3/3 concurrents. Featured snippet absent = opportunité (format tableau). SERP stable. Page non saisonnière. CTR optimisable sur 2 requêtes secondaires."

### Logique d'auto-skip

Le brief se génère directement sans pause humaine si **toutes** ces conditions sont réunies :

**Conditions bloquantes (forcent la validation humaine, quel que soit le cas) :**
- Intent match = ALERTE PIVOT → impossible d'auto-skip, le consultant doit décider du pivot stratégique
- Cannibalisation = plus de 2 conflits détectés → risque trop élevé sans validation

**Auto-skip cas 1 — position faible, travail évident :**
- Position MC principal > 10
- Intent match = OK (pas de mismatch)
- Au moins 3 gap keywords identifiés
- Similarité sémantique des gap keywords = haute (vérifiée via Haloscan `keywords_highlights` : chaque gap keyword doit apparaître dans les résultats de `keywords_highlights` du MC principal avec un `similarity` > 0.5. Si un gap keyword n'apparaît pas ou a un similarity < 0.5, la similarité globale est considérée comme faible → pas d'auto-skip)
- Pas de problème CWV bloquant (Performance > 50)

**Auto-skip cas 2 — position forte, peu d'opportunités :**
- Position MC principal ≤ 3
- Intent match = OK
- Moins de 3 gap keywords identifiés
- → Brief léger : micro-optimisations + featured snippet + E-E-A-T si gaps détectés

**Tous les autres cas → validation humaine** sur le tableau de diagnostic.

**Comportement au checkpoint :**

Si auto-skip → afficher le diagnostic complet + mention "Auto-skip appliqué (cas X)" + continuer vers Phase 3.

Si validation humaine → afficher le diagnostic complet + recommandation machine + STOP. Attendre la réponse de l'opérateur. Ne **jamais** continuer vers Phase 3 sans réponse explicite, même si l'instruction initiale disait "exécute tout". L'opérateur doit répondre l'une de ces options :

Le consultant peut alors :
- Valider tel quel → génération du brief
- Ajuster les priorités → modifier le focus du brief
- Exclure des gap keywords jugés hors sujet → éviter la dilution sémantique
- Décider d'un pivot stratégique si ALERTE PIVOT → changement de format de page ou abandon du MC
- Ajouter un mot-clé secondaire comme cible → enrichir le périmètre

---

### Séparation fait / interprétation (gate Phase 2 → Phase 3)

L'agent analyse, synthétise et recommande — c'est son rôle. Mais le brief contient deux types d'information et le lecteur (consultant, rédacteur) doit pouvoir les distinguer :

- **Fait** = donnée retournée par un outil, vérifiable. Ex : "le H1 de la page est X", "le volume du MC est 1 500/mois", "eplaque.fr a un auteur identifié".
- **Interprétation** = analyse, déduction, recommandation de l'agent. Ex : "la structure Hn recommandée devrait inclure un H2 sur les plaques rouges", "ce gap keyword est pertinent parce que…", "l'angle éditorial devrait être…".

**Règle : ce qui est présenté comme un fait doit être un fait.** L'agent peut interpréter autant qu'il veut, mais il ne doit jamais présenter une interprétation comme une observation.

**Application concrète — le contrôle à faire avant de générer le brief :**

Pour chaque élément du brief qui décrit **l'état actuel de la page** (pas les recommandations), vérifier qu'il est traçable à une donnée brute :

| Ce que le brief affirme | Comment vérifier |
|------------------------|-----------------|
| "Le plan actuel de la page est : H1 → H2 → H3…" | Chaque heading de `structure_hn.actuelle` doit exister mot pour mot dans `htags_bruts` (données brutes). Si un heading est absent des données brutes → c'est une reconstruction, pas un fait → le retirer ou le signaler. |
| "Le MC principal est X en position Y" | Les chiffres doivent correspondre aux retours GSC / Haloscan dans `donnees_brutes`. |
| "Le concurrent Z a un auteur identifié" | Doit venir du scraping effectif de ce concurrent. Si le scraping a échoué → ne pas affirmer le signal. |

Ce qui ne nécessite **pas** ce contrôle (c'est de l'interprétation assumée) :
- La structure Hn **recommandée** (c'est une proposition, pas une observation)
- Le choix des gap keywords à cibler et leur priorisation
- Les fiches actions et leurs directives
- Le diagnostic et la recommandation machine

**Si un fait n'est pas traçable :** ne pas le supprimer silencieusement. Le signaler dans le brief (`⚠️ donnée reconstituée — non vérifiée`) pour que le lecteur sache qu'il ne peut pas s'y fier à 100%.

---

### Angles de contenu unique (optionnel)

À ce stade, toutes les données concurrentielles sont disponibles (gap keywords, entités manquantes, structures Hn, E-E-A-T). Avant de passer à Phase 3, identifier **jusqu'à 3 angles de contenu différenciants** — des sujets, formats ou perspectives que les concurrents du top 10 ne couvrent PAS et qui seraient pertinents pour le persona cible.

**Ce que c'est :** des propositions, pas des directives. Le rédacteur est libre de les intégrer, de les adapter, ou de les ignorer. Ce sont des pistes de valeur ajoutée unique, pas des optimisations SEO mécaniques.

**Comment les identifier :**

1. **Croiser les lacunes SERP avec le persona.** Relire les structures Hn des 3 concurrents scrapés + les PAA : quelles questions du persona restent sans réponse dans la SERP actuelle ?
2. **Chercher les angles éditoriaux absents.** Exemples : retour d'expérience concret, cas pratique spécifique au contexte français, aspect réglementaire non traité, comparatif absent, donnée chiffrée que personne ne fournit.
3. **Vérifier que l'angle n'est pas déjà couvert par une action.** Si l'angle correspond à un gap keyword déjà traité dans les actions, ne pas le dupliquer ici. Les angles uniques = ce qui va AU-DELÀ de l'optimisation SEO classique.

**Format dans le JSON** (`angles_uniques[]`, max 3) :

```json
{
  "angle": "Comparatif des délais réels de livraison par département",
  "persona_lien": "L'acheteur potentiel veut savoir combien de temps attendre — info absente de tous les concurrents",
  "justification": "0/5 concurrents traitent les délais. PAA 'combien de temps pour recevoir sa plaque' (720 vol) sans réponse satisfaisante en SERP.",
  "format_suggere": "Tableau par zone géographique + encadré 'bon à savoir'"
}
```

**Si aucun angle pertinent n'est identifié** (la SERP couvre déjà tout, ou le sujet est trop technique pour des angles originaux) : ne rien forcer. Laisser le tableau vide. Mieux vaut 0 proposition pertinente que 3 propositions artificielles.

---

## Phase 3 — Génération du brief final

Le brief final est structuré en **deux parties** (deux onglets dans Google Docs) et s'adapte au diagnostic validé.

**Transparence sur les données manquantes :** si des appels MCP ont échoué pendant la Phase 1 (scraping concurrent bloqué, API indisponible, données insuffisantes), le brief doit le mentionner explicitement. Ajouter une ligne dans l'intro executive :

```
DONNÉES DÉGRADÉES : [liste des étapes impactées, ex: "E-E-A-T partiel (scraping bloqué sur 2/3 concurrents)", "entités manquantes non vérifiées (content_parsing échoué)"]
```

Si aucune donnée ne manque, ne rien afficher. Le principe : le consultant qui lit le brief doit savoir sur quelles données il peut se fier à 100% et lesquelles sont incomplètes.

**⚠️ ALERTE 301 (si applicable) :** si l'étape 1h bis a produit une recommandation 301, elle doit être **signalée à l'opérateur humain AVANT le brief**, dans un bloc dédié bien visible :

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ RECOMMANDATION 301 DÉTECTÉE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Page satellite : [URL]
Clics (3 mois) : [X]
→ Voir analyse complète dans le livrable 1h bis ci-dessous.
→ DÉCISION REQUISE avant publication du brief.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Suivi du livrable 1h bis complet (recommandation 301 avec arguments pour/contre et données). L'opérateur humain doit valider ou rejeter la 301 avant de transmettre le brief au rédacteur. Ne pas enterrer cette info dans le JSON ou dans une note de bas de page.

---

### ONGLET 1 : Brief rédactionnel

#### Section 1 — État des lieux

**1.1 Performances actuelles (données brutes)**
- Tableau GSC (impressions, clics, CTR par requête)
- Position Haloscan du MC principal
- Signalement des anomalies CTR

**1.2 Fiche de la page actuelle**
- URL, title, meta description actuels
- Nombre de mots actuel
- Structure Hn actuelle complète

**1.3 Diagnostic résumé**
- Tableau de diagnostic (Phase 2)
- Recommandation synthétique

#### Section 2 — Analyse concurrentielle

**2.1 Comparatif structure**
- Tableau comparatif (mots, H2, H3, title) : ma page vs top 3
- Structures Hn des concurrents top 3

**2.2 Gap keywords (post-cannibalisation)**
- Tableau des gap keywords validés (après audit 1h) avec type (trafic/business)
- Volume et CPC pour chaque gap
- Keywords exclus pour cannibalisation (avec URL conflictuelle)

**2.3 Entités manquantes**
- Tableau des entités identifiées chez 2+ concurrents et absentes de notre page
- Contexte d'utilisation chez les concurrents

**2.4 SERP features**
- Featured snippet : statut et format recommandé
- PAA : questions identifiées avec volumes

**2.5 Intent match**
- Classification du top 5 SERP par type de page
- Verdict : OK / VIGILANCE / ALERTE PIVOT
- Si VIGILANCE ou PIVOT : recommandations d'adaptation du format de page

**2.6 Gaps E-E-A-T**
- Tableau comparatif des signaux E-E-A-T (ma page vs concurrents)
- Signaux critiques manquants avec recommandation concrète (ajouter un auteur, citer des sources, ajouter une date de mise à jour...)

#### Section 3 — Recommandations de contenu (priorisées)

Tableau récapitulatif des actions :

```
| # | Action | Angle / intention | Type | Mots ajoutés | Impact | Effort | Priorité |
|---|--------|-------------------|------|-------------|--------|--------|----------|
| 1 | [action] | [comment aborder le sujet + format SERP attendu] | Contenu | +XX | Fort | Faible | P0 |
| 2 | [action] | [angle éditorial + justification] | Meta | 0 | Fort | Faible | P0 |
| 3 | [action] | [angle + format] | Contenu | +XXX | Moyen | Moyen | P1 |
| [...]
```

**Règle "Angle / intention" :** ne jamais donner un mot-clé seul. Toujours préciser comment l'aborder (format de contenu attendu par Google, angle éditorial, type de réponse). Exemples :
- "prix rénovation" → "Créer un tableau comparatif des prix par type de travaux (Google favorise les tableaux sur cette requête)"
- "avis [produit]" → "Ajouter une section témoignages structurée avec schema Review (intention transactionnelle, SERP dominée par des avis)"
- "comment [action]" → "Rédiger un pas-à-pas numéroté (featured snippet liste sur cette requête)"

**Total mots ajoutés estimé : +XXX (→ XXXX mots total)**

Puis pour chaque action, une fiche détaillée :

**Action [X] : [Nom de l'action]**
- Emplacement : [où dans la page — après quel H2/H3 existant]
- Contenu actuel : [ce qui existe, si applicable]
- Angle / intention : [comment aborder le sujet, quel format, quel bénéfice utilisateur]
- Directive : [ce qu'il faut faire concrètement]
- Exemple de contenu : [texte pré-rédigé ou ébauche]
- Justification data : [pourquoi cette action — keyword gap, entité manquante, PAA, CTR]
- Mots-clés à intégrer : [liste]
- Entités à mentionner : [si identifiées en 1d]

#### Section 3 bis — Propositions de contenu unique (optionnel)

Si des `angles_uniques` ont été identifiés en Phase 2, les présenter dans un encadré distinct des actions :

```
💡 PROPOSITIONS DE CONTENU DIFFÉRENCIANT (optionnel)
Ces angles ne sont pas couverts par les concurrents actuels.
Le rédacteur est libre de les intégrer ou non.

1. [angle] — [format_suggere]
   Persona : [persona_lien]
   Constat : [justification]

2. [angle] — [format_suggere]
   ...
```

**Règle :** ces propositions ne comptent PAS dans les 10 actions max. Elles ne sont pas assorties d'un comptage de mots (le rédacteur décide de l'ampleur). Si aucun angle pertinent n'a été identifié, cette section n'apparaît pas.

#### Section 4 — Title et meta description

```
| Élément | Actuel | Optimisé | Justification |
|---------|--------|----------|---------------|
| Title | [actuel] (XX car.) | [proposé] (XX car.) | [raison data-driven] |
| Meta desc. | [actuelle] (XX car.) | [proposée] (XX car.) | [raison data-driven] |
```

Différenciation vs concurrence : tableau montrant les titles concurrents et l'angle choisi.

#### Section 5 — Structure Hn recommandée

Présentation en diff (actuelle → recommandée) :

```
STRUCTURE Hn — ACTUELLE vs RECOMMANDÉE
───────────────────────────────────────
ACTUEL :                    RECOMMANDÉ :
H1 : [texte]                H1 : [texte optimisé]
  H2 : [texte]                H2 : [texte] (inchangé)
    H3 : [texte]                 H3 : [texte]
  H2 : [texte]                H2 : [texte modifié] ← MODIFIÉ
                               H2 : [nouveau] ← AJOUT
                                 H3 : [nouveau]
                                 H3 : [nouveau]
  H2 : [texte]                H2 : [texte] (inchangé)
  [...]                        H2 : FAQ ← AJOUT
```

#### Section 6 — FAQ recommandée

Pour chaque question :
- Question (issue des PAA réelles)
- Réponse pré-rédigée (50-100 mots)
- Justification : source PAA + volume si disponible

#### Section 7 — Ce qu'on ne touche pas

Liste explicite des éléments à préserver :
- Sections existantes à conserver
- Structure à ne pas modifier
- Contenus performants (CTR élevé) à ne pas altérer

Justification : score actuel, CTR, risque de régression.

---

### ONGLET 2 : Brief technique

#### Section 1 — Métadonnées

| Élément | Valeur actuelle | Valeur recommandée |
|---------|----------------|-------------------|
| URL canonical | [actuelle] | [recommandée ou inchangée] |
| Title | [actuel] | [optimisé] |
| Meta description | [actuelle] | [optimisée] |
| Meta robots | [actuel] | [recommandé] |
| Langue | [actuelle] | [recommandée] |

#### Section 2 — Données structurées

Pour chaque schema recommandé :
- Type de schema (Article, FAQPage, BreadcrumbList, Table, etc.)
- Éléments clés à inclure (pas de code complet, juste les champs)
- Données structurées existantes à conserver/modifier

Exemple :
```
Schema FAQPage recommandé :
- X questions issues de la section FAQ du brief rédactionnel
- Réponses complètes à inclure
- Emplacement : balise <head>, script JSON-LD séparé

Schema Article existant :
- Mettre à jour dateModified à la date de publication
- Vérifier présence auteur et organisation
```

#### Section 3 — Core Web Vitals (leviers éditoriaux uniquement)

**Principe :** ce brief est destiné à un rédacteur/intégrateur de contenu, pas à un développeur backend. On ne demande pas de corriger le LCP serveur ou le TTFB (c'est un problème d'hébergement/dev). On se concentre sur les leviers que l'éditeur de contenu maîtrise réellement.

Tableau CWV actuel (issu de Phase 1f) — pour information :

| Métrique | Valeur | Statut | Levier éditorial ? |
|----------|--------|--------|:------------------:|
| Performance | XX/100 | ... | Partiel |
| LCP | X.Xs | ... | OUI (images) |
| INP | XXms | ... | NON (dev) |
| CLS | X.XX | ... | OUI (images/embed) |

**Leviers actionnables par l'éditeur de contenu :**

**CLS (stabilité visuelle) :**
- Toutes les images doivent avoir les attributs `width` et `height` explicites
- Les embeds (vidéo, iframe, widgets) doivent avoir un conteneur avec dimensions fixes
- Pas de contenu injecté dynamiquement au-dessus du fold sans espace réservé

**LCP content-based :**
- L'image la plus haute dans la page (hero/bannière) ne doit pas dépasser 200 Ko
- Format WebP obligatoire pour les images au-dessus du fold
- Pas de lazy loading sur l'image LCP (celle visible sans scroll)
- Les images sous le fold doivent avoir `loading="lazy"`

**Ce qui est hors périmètre de ce brief** (à remonter à l'équipe technique si problème détecté) :
- TTFB > 600ms → problème serveur
- INP > 200ms → problème JavaScript
- Render-blocking resources → problème d'architecture front
- Cache policy → problème infrastructure

#### Section 4 — Recommandations balisage HTML

- Tableaux : présence/absence de `<caption>`, `<thead>`, `<tbody>`, `scope="col"`
- Images : vérification alt text, formats recommandés (WebP), lazy loading
- Liens internes : ancres descriptives, vérification URLs
- Balisage sémantique : `<header>`, `<main>`, `<section>`, `<footer>`
- Hiérarchie titres : respect séquentiel H1 → H2 → H3 (pas de saut)

#### Section 5 — Optimisation mobile

Points d'attention spécifiques :
- Tableaux responsives (scroll horizontal si nécessaire)
- Taille zones cliquables
- Adaptation éléments interactifs
- Vérification affichage structure Hn sur mobile

#### Section 6 — Checklist d'implémentation

**Avant publication :**
- [ ] Backup de l'article actuel
- [ ] Validation factuelle des contenus ajoutés
- [ ] Test affichage mobile
- [ ] Test données structurées (Google Rich Results Test)
- [ ] Vérification liens internes (pas de 404)

**Après publication :**
- [ ] Demande réindexation via GSC
- [ ] Surveillance positions J+7 (requêtes existantes)
- [ ] Monitoring featured snippet J+14
- [ ] Comparaison trafic J+7, J+14, J+30
- [ ] Vérification cannibalisation post-publication (les gap keywords ciblés n'ont pas fait chuter les autres pages du site)

**Suivi long terme :**
- [ ] Analyse positions gap keywords tous les 15 jours
- [ ] Vérification CTR post-optimisation (30 jours)
- [ ] Itération si pas de résultats visibles après 6 semaines

---

## Métriques de succès (template)

| Objectif | Métrique | Cible | Délai |
|----------|----------|-------|-------|
| Position MC principal | Position Haloscan | [cible selon diagnostic] | 1-2 mois |
| Featured snippet | Présence SERP | Capturé | 1-2 mois |
| Gap keywords | Positions top 10 | X/Y keywords capturés | 2-3 mois |
| Trafic organique | Clics GSC | +XX% | 2-3 mois |
| CTR | CTR moyen GSC | > X% | 1 mois |

---

## Annexe : correspondance outils/actions

| Action dans le process | Phase | Outil | Fonction/endpoint | Notes |
|------------------------|-------|-------|-------------------|-------|
| Découverte propriété GSC | 1b | GSC | `sites.list` | Si format fourni échoue (sc-domain vs https://) |
| Scraping page (Hn, metas, mots) | 1a | DataForSEO | `on_page_instant_pages` | |
| Contenu structuré page | 1a | DataForSEO | `on_page_content_parsing` | |
| Performance URL (clics, impressions, CTR) | 1b | GSC | `search_analytics` (dim: query) | Fenêtre adaptée si page < 90j |
| Détection saisonnalité (12 mois) | 1b | GSC | `search_analytics` (dim: date) | Non mesurable si page < 6 mois |
| Données MC : volume, CPC, KGR | 1c | Haloscan | `keywords_overview` → `metrics` | Appel unique combiné |
| SERP organique top 100 (URLs concurrentes) | 1c | Haloscan | `keywords_overview` → `serp` | Source des URLs pour 1d/1e/1g |
| PAA questions + volumes | 1c | Haloscan | `keywords_overview` → `related_question` | Remplace `get_keywords_questions` |
| Long-tail du MC | 1c | Haloscan | `keywords_overview` → `keyword_match` | |
| Keywords sémantiquement proches | 1c | Haloscan | `keywords_overview` → `similar_highlight` | |
| Synonymes du MC | 1c | Haloscan | `keywords_overview` → `synonyms` | Utile pour recommandations rédac |
| Historique volume MC | 1c | Haloscan | `keywords_overview` → `volume_history` | Complément saisonnalité |
| Domaines dominants thématique | 1c | Haloscan | `keywords_overview` → `top_sites` | |
| Gap analysis (keywords par URL concurrente) | 1d | Haloscan | `page_best_keywords` | **Premier choix** — 1 appel par concurrent |
| Gap analysis (intersection pages) | 1d | DataForSEO | `page_intersection` | **Fallback** si page_best_keywords insuffisant |
| Enrichissement gap (similarité + volume) | 1d | Haloscan | `keywords_highlights` (volume_min=100) | Premier choix enrichissement — score similarité inclus |
| Analyse entités concurrents | 1d | DataForSEO | `on_page_content_parsing` (URLs conc.) | Scraping contenu pour entités |
| SERP features (FS, PAA pos, vidéos, images) | 1e | DataForSEO | `serp_organic_live_advanced` | Features uniquement, pas positions |
| Intent label (informatif) | 1e | DataForSEO | `dataforseo_labs_search_intent` | Overridable par analyse top 10 |
| CWV / Lighthouse | 1f | DataForSEO | `on_page_lighthouse` | |
| Structure Hn concurrents (3 URLs) | 1g | DataForSEO | `on_page_instant_pages` | Htags potentiellement incomplets si rendu JS — documenter |
| E-E-A-T concurrents (3 URLs) | 1g | DataForSEO | `on_page_content_parsing` | |
| Cannibalisation (gap kw vs domaine) | 1h | Haloscan | `domains_positions` (filtre keywords) | |
| Volatilité SERP | 2 | Haloscan | `keywords_serp_compare` | |
| Évolution page dans le temps | 2 | Haloscan | `keywords_serp_pageEvolution` | Optionnel — suivi position |
| Historique positions domaine | — | Haloscan | `domains_history_positions` | Optionnel — complément |
| Pages les plus fortes concurrent | — | Haloscan | `domains_top_pages` | Optionnel — priorisation concurrents |
| Keywords similaires (vérif. sémantique + enrichissement gap) | 1d, 2 | Haloscan | `keywords_highlights` (similarity_min=0.5, volume_min=100) | Score de similarité explicite — auto-skip + gap enrichment |
| Découverte keywords multi-source (fallback) | 1d | Haloscan | `find_keywords` (sources: serp, related, highlights, questions) | Si page_best_keywords + highlights insuffisants |
| Clustering topical (optionnel) | 1h | Haloscan | `keywords_site_structure` | Vérifier que les gap kw appartiennent au même cluster — 50 kw min |

---

## Notes de version

**v3.1 (25/02/2026)** — Fix extraction structure Hn
- **Bug critique identifié** : les htags retournés par DataForSEO (`on_page_instant_pages`) mélangent headings éditoriaux et headings template (footer, sidebar, CTA). La structure Hn détaillée stockée dans les briefs était reconstruite par inférence du texte brut — pas issue d'un parsing HTML fiable. Résultat : plans Hn dans les consignes rédacteur ne correspondant pas au vrai contenu de la page.
- **1a refondu** : `curl` + parsing HTML Python devient la source de vérité pour la structure Hn. Script inclus dans la SOP (extraction H1-H4, décodage entités HTML, filtre template par regex). DataForSEO reste pour les données techniques (mots, title, meta, liens, images).
- **Schéma JSON enrichi** : `htags_bruts` (texte exact de chaque heading), `htags_source` (méthode d'extraction), `htags_avertissement` — champs obligatoires dans `fiche_technique_page`. Le plan Hn est désormais traçable et vérifiable.
- **Fallback documenté** : si curl échoue → DataForSEO `on_page_instant_pages` → Chrome `javascript_tool`. Avertissement obligatoire si source dégradée. Interdit de reconstruire un plan par inférence du texte brut. Si Hn indisponible → STOP avant Phase 3 + validation humaine.
- **1g enrichi** : même avertissement pour les concurrents (htags potentiellement incomplets). Documenter la limitation, ne jamais compléter par inférence.
- **Règle ajoutée** : seuil minimum 3 H2 éditoriaux. Si < 3 → flag `⚠️ STRUCTURE HN SUSPECTE`.
- **Gate "séparation fait / interprétation"** ajoutée entre Phase 2 et Phase 3. Principe : ce qui est présenté comme un fait (état actuel de la page, données concurrents) doit être traçable à une donnée brute. Ce qui est une interprétation (recommandations, priorisation, angle éditorial) est libre. Si un fait n'est pas traçable → le signaler comme "donnée reconstituée".

**v3.0 (17/02/2026)** — Séquencement, checkpoint, Haloscan-first, top 10
- Graphe de dépendances explicite entre sous-étapes Phase 1 (3 vagues séquentielles + 1 vague cannibalisation)
- Règle absolue : ne jamais lancer vague N+1 avant fin de vague N (empêche l'hallucination de données)
- Checkpoint Phase 2 non court-circuitable (même si instruction "exécute tout")
- Gestion des erreurs MCP : retry 1x, puis "données indisponibles" (jamais inventer)
- **1c refondu** : appel unique `keywords_overview` avec `requested_data` complet (metrics, serp, related_question, keyword_match, similar_highlight, synonyms, volume_history, top_sites, categories). Fournit les URLs concurrentes pour toutes les phases suivantes.
- **1d rebalancé Haloscan-first** : `page_best_keywords` comme outil principal de gap analysis (par URL concurrente). `keywords_highlights` (avec score de similarité + filtre volume) pour enrichissement. `find_keywords` (multi-source) en fallback secondaire. DataForSEO `page_intersection` en dernier recours. Interdit d'utiliser `domains_competitors_keywords_diff` au niveau domaine (hors sujet garanti sur sites généralistes).
- **1e : intent sur top 10** (au lieu de top 5). Seuil "dominante" = 6+/10. Séparation claire : Haloscan pour positions organiques, DataForSEO pour features SERP uniquement. Règle d'override explicite quand label DataForSEO contredit le top 10 réel.
- **1g : sélection concurrents** depuis le top 10 filtré de 1c (même type de page, meilleure position, domaine comparable)
- **Auto-skip** : vérification similarité sémantique via `keywords_highlights` (score similarity > 0.5) au lieu de `keywords_similar`
- **Maillage interne retiré** du brief — fait manuellement par le consultant
- `keywords_site_structure` ajouté en optionnel (clustering topical des gap keywords, 50 kw min)
- Annexe outils/actions refondue avec nouveaux endpoints Haloscan
- **1b : découverte propriété GSC** — étape préalable `sites.list` si le format fourni échoue (sc-domain vs https://)
- **1b : garde-fou page récente** — si la page a < 90 jours dans l'index GSC, adapter la fenêtre de données. Si < 6 mois, saisonnalité non mesurable. Si < 30 jours, MC fragile.
- **1b : sélection MC** — regrouper les variantes proches (avec/sans accent, article) avant de sélectionner. Choisir la forme avec le plus de volume Haloscan pour les appels suivants.
- **Parallélisation** : clarification — "parallélisable" = pas de dépendance d'ordre. Si les sous-agents ont accès aux MCP, la parallélisation est autorisée. Si ça échoue, basculer sur séquentiel. Ne jamais boucler sur un mécanisme cassé.
- **Fallback scraping concurrents** : quand `on_page_content_parsing` échoue sur les URLs concurrentes (fréquent), cascade documentée : `on_page_instant_pages` → données SERP Haloscan → noter "scraping bloqué". Ne jamais inventer des signaux E-E-A-T ou des entités.
- **Principe 6 ajouté** : "Raisonner, documenter, dévier" — l'agent peut dévier de la méthode par défaut à condition de documenter la déviation et sa justification. Appliqué au choix du MC (1b) et applicable à toute décision de la SOP.

**v2.1 (17/02/2026)** — Corrections critiques post-revue
- Ajout Phase 1h : audit cannibalisation préventif (avant brief, plus en checklist post-publication)
- Détection saisonnalité intégrée en 1b (YoY 12 mois + ratio pic/creux)
- Analyse entités sémantiques ajoutée en 1d (gap au-delà des mots-clés)
- Volatilité SERP ajoutée au diagnostic Phase 2
- Colonne "Angle / intention" dans le tableau de recommandations (ne plus jamais donner un mot-clé seul)
- Section CWV recentrée sur les leviers éditoriaux uniquement (CLS images, LCP content-based)

**v2.0 (17/02/2026)** — Refonte complète
- Pipeline d'extraction unique (plus de scénarios rigides)
- Diagnostic synthétique avec auto-skip
- Gap analysis systématique avec double filtre (volume > 100 OU CPC > 2€)
- Séparation claire des rôles outils (GSC : clics/impressions, Haloscan : positions/keywords, DataForSEO : technique/SERP)
- Analyse CTR intégrée comme indicateur clé
- Format brief standardisé en 2 onglets
