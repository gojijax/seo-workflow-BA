# Redaction SEO — contenu publiable optimise

Redige du contenu SEO publiable pour une page existante (enrichissement) ou une nouvelle page (creation). Le contenu produit doit etre pret a copier-coller dans WordPress, avec un style humain, un angle unique, et une vraie valeur pour le lecteur.

## Arguments

$ARGUMENTS = [client] URL mot-cle-principal
- Ex: `/redaction mon-client https://www.mon-site.fr/ma-page "mot cle principal"`
- Si le client n'est pas precise, demander quel client.
- Si le mot-cle principal n'est pas precise, le deduire du brief existant ou demander.

## Pre-requis

1. **Lis `clients/{client}/PROJECT-CONTEXT.md`** — config client, phase, persona, ton, contraintes.
2. **Lis `clients/{client}/personas-parcours-client.md`** si disponible — questions/freins/besoins reels des prospects.
3. **Lis le brief existant** dans `clients/{client}/briefs/{slug}/` s'il existe — keywords, problemes identifies, donnees GSC.
4. **Lis `consignes/sop-brief-seo-v3.md`** pour les conventions d'outils et de non-redondance (Haloscan, DataForSEO, GSC).

## Le skill fonctionne en 3 ETAPES OBLIGATOIRES

**L'utilisateur DOIT valider l'etape 1 avant que l'etape 2 ne demarre.**
**L'etape 3 (relecture qualite) est executee automatiquement apres la redaction.**

---

## ETAPE 1 — Analyse & strategie (output de validation)

### 1.1 Crawl de la page existante

- Recuperer le contenu actuel via DataForSEO `on_page_content_parsing` OU curl
- Inventorier : title, meta, H1, structure Hn, nombre de mots, texte brut
- Si la page n'existe pas → mode creation

### 1.2 Objectif mesurable du contenu

Determiner le type de lecteur cible par cette page :

| Type de lecteur | Proximite achat | Implication | CTA principal |
|---|---|---|---|
| **Decisionnaire** | Tres elevee | Elevee | Acheter / Commander / Prendre RDV |
| **Convaincu en attente** | Elevee | Faible a moyen | Obtenir un devis, s'inscrire newsletter |
| **Evaluateur** | Moyenne | Elevee | Telecharger une ressource, demander un rappel |
| **Explorateur** | Faible | Faible | Cliquer pour en savoir plus |

### 1.3 Scraping SERP top 10-15

Pour le mot-cle principal et 2-3 variantes, recuperer via Haloscan et DataForSEO :
- Les 10-15 URLs qui rankent
- Leur structure de contenu (H1, angle, longueur estimee)
- Les PAA (People Also Ask)
- Les SERP features

Pour chaque resultat top 5, crawler le contenu pour :
- Identifier les angles couverts par tous (= le "corpus commun")
- Identifier les formats utilises (tableaux, FAQ, listes, calculateurs)
- Extraire les entites nommees recurrentes

#### Pourquoi recuperer les PAA

Les PAA sont de la data utilisateur fraiche. Google les calcule a partir des besoins informationnels les plus cliques. Pour ranker dans les PAA :
- Reponses precises et concises : **liste = 45 mots max, reponse simple = 30 mots max**
- Le passage doit integrer les termes cles utilises par les concurrents

Les LLMs (GPT-4, Gemini) utilisent aussi les PAA comme source d'entrainement — couvrir les PAA = couvrir aussi les LLMs.

### 1.4 Audit du format de page

**1. Observer les resultats enrichis** : featured snippets, images/videos, panels, cartes locales, calculateurs.

Si Google affiche un tableau en featured snippet → le contenu DOIT integrer un tableau.

**2. Choisir un framework editorial** adapte :

| Framework | Quand l'utiliser |
|---|---|
| **MECE** | Guides longs, piliers SEO — couvrir 100% du sujet sans chevauchement |
| **PAS** | Pages orientees "pain points", landing pages produit |
| **AIDA** | Pages hybrides article + conversion, comparatifs |
| **Pyramide inversee** | News, blog corporate, reponses rapides |

### 1.5 Identification du gap (Information Gain) — ~30% du plan

> **"Qu'est-ce que le top 10 ne couvre PAS, ou couvre mal, que notre page pourrait apporter ?"**

L'Information Gain est un concept brevete et integre dans les interfaces IA de Google. Google mesure ce qu'un article **ajoute** par rapport aux concurrents.

**Cible : environ 30% des sections du plan doivent apporter un angle unique.** Ces sections sont **reparties dans le plan global**, pas regroupees en fin d'article.

Sources d'Information Gain :
- Questions du parcours client non adressees dans le top 10
- Donnees locales absentes
- Angle emotionnel/vecu absent
- Expertise technique non vulgarisee
- Fonctionnalite manquante (tableau comparatif, calculateur, checklist)
- Retours d'experience concrets / cas clients
- Etudes proprietaires ou donnees internes

### 1.6 Structure Hn — plan et validation semantique

**Regles de construction :**
- Titres courts et descriptifs (pas de titres vagues/marketing)
- Structure logique et progressive
- Inclure les ~30% de sections [ANGLE UNIQUE] reparties dans le plan

**Titres descriptifs pour les LLMs — OBLIGATOIRE :**

| Mauvais titre | Bon titre |
|---|---|
| "Les problemes que ca pose" | "Les problemes causes par le sol argileux en nord Isere" |
| "Notre approche" | "Accompagnement personnalise du terrain a la remise des cles" |
| "Etape 1" | "1ere etape : choisir votre terrain et verifier sa constructibilite" |

Chaque titre doit permettre a un lecteur (ou un LLM) de comprendre le sujet de la section **sans lire le contenu**.

### 1.7 Output Etape 1

Presenter a l'utilisateur : objectif du contenu, format & framework, corpus SERP, gap identifie, entites a couvrir, structure Hn proposee, fonctionnalites recommandees.

**→ STOP. Attendre validation utilisateur. Ne PAS passer a l'etape 2 sans accord explicite.**

---

## ETAPE 2 — Redaction

### 2.1 Principes redactionnels NON NEGOCIABLES

#### Chaque phrase = valeur
- Avant d'ecrire une phrase, se demander : "Si je la supprime, est-ce que le lecteur perd quelque chose ?"
- Pas de remplissage, pas de transitions creuses

#### Pas de cible de longueur
- Le contenu fait la taille qu'il doit faire
- La valeur prime TOUJOURS sur le volume

#### Un paragraphe = une information
- Paragraphes courts : 3-5 lignes max (lecture mobile)

#### Angle unique, pas copie SERP
- Le contenu apporte l'angle identifie dans l'etape 1 (Information Gain)

### 2.2 Redaction section par section

Pour les contenus > 1200 mots prevus, rediger section par section. Ne PAS tenter un one-shot.

### 2.3 Introduction — TL;DR + PATT

#### 1. Bloc TL;DR
- 2 a 5 points maximum, 20 mots max par phrase
- Place AVANT l'introduction textuelle

#### 2. Introduction PATT (Probleme – Apercu – Teasing – Transition)
- 50 a 100 mots
- Pas de buildup generique

#### 3. Optimisation semantique des 200 premiers mots
- Les 200 premiers mots ont un poids disproportionne (biais de primaute, zone la plus vue, IS Predictor)
- Integrer les termes semantiques cles naturellement

### 2.4 Anti-patterns IA — LISTE NOIRE

**Phrases creuses interdites :**
- "Dans un monde ou..."
- "Il est important de noter que..."
- "N'hesitez pas a..."
- "Force est de constater que..."
- "Vous l'aurez compris, ..."

**Patterns structurels interdits :**
- Conclusion qui repete l'introduction
- Toutes les sections de meme longueur
- Questions rhetoriques en ouverture de chaque section

**Patterns de vocabulaire interdits :**
- Surutilisation de "permettre"
- "Au coeur de" / "au sein de" en surabondance
- Adverbes inutiles : "veritablement", "absolument", "indeniablement"

**HTML propre — OBLIGATOIRE :**
- Ne JAMAIS laisser de data-attributes IA dans le HTML (ex: `data-start="0"`, `data-end="9"`)

### 2.5 Style et burstiness (variation humaine)

- **Varier la longueur des phrases** : alterner phrases de 5 mots et phrases de 25 mots
- **Varier la longueur des paragraphes**
- **Commencer les phrases differemment**
- **Oser les phrases nominales**
- **Oser les opinions**
- **Injecter du specifique** : le generique = IA, le specifique = humain

### 2.6 Title tag — bonnes pratiques

**Methode :**
1. Lister les titles des concurrents top 10
2. Categoriser les mots
3. Proposer 3-5 options
4. Choisir celui qui correspond au persona ET au contenu

**Patterns performants (etudes SearchPilot) :**

| Pattern | Impact |
|---|---|
| Nombres dans le titre | +16% trafic |
| Titre plus general | +24% trafic |
| Signal de fraicheur ("2026") | +11% trafic |
| Marque en premier (si reconnue) | +15% trafic |

**Meta description : NE PAS en rediger.**
Etude SearchPilot : retirer les meta descriptions = +4.2% de sessions organiques.

### 2.7 Tone of Voice

Le ton est defini dans `PROJECT-CONTEXT.md` du client. A defaut :
- Vouvoiement
- Registre professionnel mais accessible
- Posture : accompagnateur expert, pas vendeur

### 2.8 Maillage interne

Ne PAS se limiter a "site:mot-cle" sur Google (biaise, incomplet).

**Criteres :**
1. Penser utilisateur : ou l'emmener ensuite ?
2. Ne pas oublier les vieux articles
3. Proximite semantique

---

## ETAPE 3 — Relecture qualite (auto-executee)

### 3.1 Verification anti-patterns IA
Scanner les phrases creuses, verifier la variation, compter les occurrences de "permettre".

### 3.2 Fact-checking
Identifier toutes les allegations, verifier chaque point, convertir en [PLACEHOLDER] si non verifiable.

### 3.3 Metriques de lisibilite
- Diversite lexicale : cible 60-80%
- Mots par phrase : 15-20 en moyenne
- Voix passive : limiter
- Adverbes lourds : max 5-8 pour le texte complet

### 3.4 Checklist finale

**Contenu :** valeur par phrase, burstiness, pas de liste noire, HTML propre
**Structure :** TL;DR, intro PATT, 200 premiers mots, titres Hn descriptifs, ~30% angle unique
**SEO :** title optimise, PAS de meta description, entites, maillage interne
**Engagement :** CTA contextuels, bloc auteur EEAT, recommandations visuelles

---

## Sauvegarde

- Sauvegarder le contenu dans `clients/{client}/briefs/{slug}/contenu-redige.md`
- Mettre a jour `clients/{client}/PROJECT-CONTEXT.md`

## Regles cardinales (rappel)

1. **Jamais de donnees inventees** — [PLACEHOLDER] plutot que mentir
2. **3 etapes obligatoires** — Analyse → Redaction → Relecture qualite
3. **Validation etape 1** — NE PAS rediger sans accord explicite
4. **Information Gain ~30%** — si le contenu ne dit rien de nouveau, il est rate
5. **Anti-patterns IA** — relire et verifier la liste noire
6. **Titres Hn descriptifs** — comprehensibles isolement, pour humains ET LLMs
7. **Pas de meta description** — laisser Google la generer
8. **HTML propre** — aucun artefact IA dans le code livre
