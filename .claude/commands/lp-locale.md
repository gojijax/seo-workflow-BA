# LP Locale — Landing page SEO local programmatique

Cree une landing page SEO locale unique pour un couple {service x ville/zone}. Chaque page est enrichie de donnees locales reelles (INSEE, DVF, cadastre via data.gouv.fr), scrapee SERP, et redigee avec le skill `/redaction` augmente d'une couche locale. Un verificateur anti-doorway garantit l'unicite inter-pages.

## Arguments

$ARGUMENTS = [client] mot-cle-principal ville/zone [--batch fichier.md]
- Ex: `/lp-locale mon-client "plombier" "Marseille"`
- Ex: `/lp-locale mon-client "constructeur maison" "Lyon" --batch prompts-lp-locales.md`
- Le mot-cle principal et la ville sont OBLIGATOIRES.

## Pre-requis

1. **Lis `clients/{client}/PROJECT-CONTEXT.md`** — config client, zones couvertes.
2. **Lis `clients/{client}/local-config.json`** s'il existe — config locale deja cadree.
3. **Lis `clients/{client}/personas-parcours-client.md`** si disponible.
4. **Lis `consignes/sop-brief-seo-v3.md`** pour les conventions d'outils.

## Le skill fonctionne en 4 PHASES OBLIGATOIRES

**Phase 0** : Cadrage client (une seule fois, pas a chaque LP)
**Phase 1** : Enrichissement donnees locales (par LP)
**Phase 2** : Analyse & redaction (couplage /redaction)
**Phase 3** : Verification anti-doorway (par LP, AVANT livraison)

**L'utilisateur DOIT valider la Phase 1 avant que la Phase 2 ne demarre.**
**La Phase 3 est automatique mais peut STOPPER la livraison.**

---

## PHASE 0 — Cadrage client (une fois par client)

Si `clients/{client}/local-config.json` existe deja → SKIP cette phase.

Sinon, poser ces questions et sauvegarder les reponses :

**Business :** activite, USPs, certifications, anciennete, nombre d'agences et adresses
**Zones :** zone de chalandise par agence, communes prioritaires, zones exclues
**Assets locaux :** realisations geolocalisees, temoignages, photos, donnees proprietaires
**Concurrence locale :** concurrents par zone, differenciation
**Technique :** CMS, template LP existant, domaine unique ou multi-domaines

Sauvegarder dans `clients/{client}/local-config.json`.

---

## PHASE 1 — Enrichissement donnees locales (par LP)

### 1.1 Donnees geographiques de base
- Commune : nom officiel, code INSEE, code postal, departement, intercommunalite
- Population, communes limitrophes, distance depuis l'agence la plus proche

### 1.2 Donnees INSEE (via data.gouv.fr)
- Demographie : population, evolution sur 5-10 ans, tranches d'age
- Revenus : revenu median, % CSP+
- Logement : % proprietaires vs locataires, % maisons individuelles
- Rechercher via `search_datasets` les jeux de donnees pertinents

### 1.3 Donnees immobilieres DVF
- Prix moyen du m² terrain dans la commune et limitrophes
- Evolution des prix sur 2-3 ans
- Nombre de transactions terrain

### 1.4 Donnees urbanisme / constructibilite
- PLU : zones constructibles, contraintes
- Zones de risques : inondation, argile, sismique (georisques.gouv.fr)
- Projets d'amenagement : ZAC, lotissements en cours

### 1.5 SERP locale
- Scraper via Haloscan pour "[mot-cle] [ville]", "[mot-cle] [departement]"
- Top 10-15 URLs, PAA locales, SERP features (map pack, featured snippet)
- Crawler les top 3-5 pour analyser le contenu local

### 1.6 Assets client dans la zone
- Realisations livrees dans la commune ou a proximite
- Note et avis Google Business Profile
- Temoignages geolocalises

### Output Phase 1
Presenter la fiche donnees locales complete a l'utilisateur.
**→ STOP. Attendre validation.**

---

## PHASE 2 — Analyse & redaction (couplage /redaction)

Applique le skill `/redaction` avec les **ajouts locaux suivants** :

### Structure obligatoire d'une LP locale

1. **Accroche locale** (H1) — "[Service] a [Ville] : [proposition de valeur unique locale]"
2. **Contexte local** — donnees demographiques, qualite de vie (alimente par INSEE)
3. **Marche immobilier local** — prix, tendances, comparaison communes voisines (alimente par DVF) — section [ANGLE UNIQUE]
4. **Services adaptes au contexte local** — pas une copie de la page service generique
5. **Realisations dans la zone** — projets livres avec details
6. **Urbanisme et constructibilite** — PLU, zones, risques — section [ANGLE UNIQUE]
7. **Temoignages / avis locaux**
8. **FAQ locale** — questions specifiques a la zone
9. **CTA localise** — contact agence la plus proche

### Injection donnees locales
- Chaque donnee locale DOIT etre sourcee : "(source : INSEE 2023)" / "(source : DVF 2024)"
- Pas de paraphrase vide — des FAITS

### Anti-generique — controle obligatoire
- Chaque paragraphe DOIT contenir au moins un element specifique a la ville
- Test mental : "Si je remplace [Ville] par [Autre Ville], est-ce que la phrase est toujours vraie ?" → Si oui, reecrire

### Schema markup obligatoire
- Schema `LocalBusiness` avec `areaServed`
- Schema `FAQ` pour la section FAQ locale

### Maillage interne hub & spoke
- Mailler vers : homepage agence, pages service, autres LP locales du meme hub, page realisations
- Recevoir un lien depuis : homepage agence, pages service

---

## PHASE 3 — Verification anti-doorway

### 3.1 Test d'unicite inter-pages
- Comparer avec chaque LP existante du client
- **Seuil** : max 15% de contenu partage entre 2 LP locales

### 3.2 Test anti-generique
- Compter les paragraphes "generiques" (qui passent le test de substitution ville)
- **Seuil** : max 20% de paragraphes generiques

### 3.3 Verification des sources
- Chaque chiffre cite DOIT avoir une source identifiable (INSEE, DVF, PLU)
- Aucun chiffre invente

### 3.4 Verification signaux locaux (minimum 5/8)
- [ ] Nom de la commune dans le H1
- [ ] Communes limitrophes mentionnees
- [ ] Donnees INSEE/demographiques
- [ ] Prix immobilier local (DVF)
- [ ] Reference au PLU ou urbanisme local
- [ ] Realisation ou temoignage geolocalise
- [ ] Reference a un point geographique specifique
- [ ] Contact de l'agence locale

### 3.5 Verification cannibalisation
- Comparer les mots-cles cibles avec ceux des pages existantes
- Si chevauchement > 3 mots-cles principaux → ALERTE

### Verdict : LIVRABLE ou A CORRIGER
Si a corriger → corriger et relancer la Phase 3.

---

## Mode batch (4-5 LP max)

1. Le cadrage (Phase 0) est fait UNE SEULE FOIS
2. La Phase 1 est faite pour TOUTES les LP avant de commencer la redaction
3. La redaction (Phase 2) est faite LP par LP, sequentiellement
4. La verification (Phase 3) compare chaque nouvelle LP avec TOUTES les precedentes

---

## Regles cardinales

1. **Chaque LP doit etre unique** — si le test d'unicite echoue, la LP n'est pas livree
2. **Chaque chiffre a une source** — INSEE, DVF, PLU. Jamais d'invention.
3. **Test de substitution** — si remplacer le nom de ville ne change rien, le paragraphe est rate
4. **5/8 signaux locaux minimum** — une LP sans signaux locaux est une doorway page
5. **Hub & spoke obligatoire** — chaque LP maille vers la pilier et les pages service
6. **Schema LocalBusiness obligatoire** — avec areaServed
7. **Anti-doorway = non negociable** — Google penalise. Mieux vaut 3 LP excellentes que 10 LP mediocres.
