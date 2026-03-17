# Brief SEO — lancement complet

Lance le process complet de brief SEO pour une URL.

## Arguments

$ARGUMENTS = [client] URL de la page a briefer
- Ex: `/brief mon-client https://www.mon-site.fr/ma-page`
- Si le client n'est pas precise, demander quel client.

## Instructions

1. **Identifie le client** a partir du premier argument. Determine le dossier `clients/{client}/`.

2. **Lis `clients/{client}/PROJECT-CONTEXT.md`** pour charger la config client (domaine, GSC, Drive, persona, ton, contraintes).

3. **Lis les fichiers consignes/** — tous les `.md` et `.json` presents. Ces fichiers contiennent la procedure complete, le format de rendu, le schema JSON, et les regles d'execution. Applique-les a la lettre.

4. **Si aucun fichier consignes/** trouve : STOP. Afficher :
   > Aucun fichier de consignes trouve dans `consignes/`. Le process de brief necessite tes SOPs. Ajoute-les et relance.

5. **Sinon**, deroule le process selon les consignes :
   - Phase 1 : extraction brute (vagues sequentielles selon la SOP)
   - Phase 2 : diagnostic + checkpoint
   - Phase 3 : JSON + brief + consigne redacteur + export Drive

6. **Export Drive** :
   - Recupere l'ID du dossier Drive parent depuis PROJECT-CONTEXT.md du client
   - Cree un sous-dossier nomme par le slug de l'URL
   - Cree les Google Docs : brief SEO, consigne redacteur, JSON data

7. **Sauvegarde locale** :
   - Cree le dossier `clients/{client}/briefs/{slug}/`
   - Ecris les fichiers `data.json`, `brief.md`, `consigne.md`

8. **Log** : ajoute une entree au journal `clients/{client}/journal/JOURNAL.md` "Brief {slug} livre sur Drive".

## Regles cardinales (rappel)

- Jamais de donnees inventees
- Checkpoint Phase 2 non court-circuitable
- Intent match alerte → STOP + validation humaine
- Vagues strictement sequentielles
