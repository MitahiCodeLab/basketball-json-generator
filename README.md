# JSON Generator

Générateur de **documents JSON** conformes à un **JSON Schema** pour des matchs (métadonnées + joueurs + statistiques par skill).

## 📁 Arborescence

.
├── data/ # Sources (divisions, sections, zones, skills, effectifs...)
├── examples/ # Exemples JSON générés et validés
├── generator/ # App web Vanilla JS pour générer & télécharger
├── schema.json # Schéma JSON (Draft 2020-12)
└── README.md

## 🎯 Objectif

1. **Définir un schéma JSON strict** décrivant la structure d’un match :  
   `[{ metaMatch }, [ { player + stats par skill }... ]]`.
2. **Générer** des documents JSON aléatoires mais **cohérents** (zones/skills) et **reproductibles** via un `seed`.
3. **Valider** automatiquement les sorties contre `schema.json`.

---

## 🔧 Prérequis

Aucun package obligatoire pour la version navigateur.  
Pour servir localement (le navigateur bloque `fetch` sur le disque) :

- Node : `npx serve .` → ouvre `http://localhost:3000/generator/`

---

## 🗂️ Données sources (`/data`)

- `divisions.json` : liste d’objets `{ ID, division }`
- `sections.json` : tableau de strings (ex. `"U18M"`, `"SéniorsF"`, …)
- `skills.json` : 13 compétences (clés obligatoires dans la sortie)
- `zones.json` : liste blanche des zones de tir autorisées
- `players-team-*.json` : effectifs avec colonnes `ID`, `NOM`, `PRENOM`, `NUMERO`, `POSITION`

**Mapping joueurs → sortie** :

- `firstname` ← `NOM`
- `lastname` ← `PRENOM`
- `number` ← `NUMERO`

---

## 🧩 Schéma (`schema.json`)

- **Draft** : 2020-12
- **Structure** : tableau à 2 éléments

  1. `meta` : `{ team, squad?, division(enum), section(enum), game{host, opponent}, date(YYYY-MM-DD) }`
  2. `players[]` : chaque joueur possède **toutes** les 13 `skills` (même avec `{ "count": 0 }`)

- **Événements** : indexés `"1".."N"` via `patternProperties` ; chaque événement = `{ time, zone(enum) }`.

> ℹ️ On peut renforcer des règles métier dans le schéma (ex. `freethrow` → zone `freethrows`) avec des `if/then`, mais la cohérence est déjà assurée côté générateur.

---

## 🚀 Génération (Vanilla JS)

Ouvre `generator/index.html` via un serveur local, puis :

1. Choisis **équipe hôte** et **adverse** (fichiers d’effectifs).
2. Sélectionne **division**, **section**, **date**.
3. Renseigne **seed** (entier) pour un résultat **déterministe**.
4. Paramètre **Nombre de matchs** et **Max actions par skill/joueur**.
5. Clique **Générer** → aperçu en JSON → **Télécharger**.

### Champs générés

- `meta` : `team`, `squad` (par défaut `2`), `division`, `section`, `game{host,opponent}`, `date`
- `players[]` : `id`, `firstname`, `lastname`, `number` + 13 `skills`  
  Chaque `skill` contient `count` et, si `count > 0`, des entrées `"1".."N"` avec `{ time, zone }`

### Cohérence zones/skills

- `freethrow` → zone uniquement `"freethrows"`
- `shootwiderange` → zones 3 points + arc + (tolérance) `freethrows`
- `shootmidrange` → zones `mid-range-*`
- `Autres skill` → zones libres (ajustable)

---

## ✅ Validation

### Option 1 — Depuis le navigateur

Un bouton **Valider** peut charger `schema.json` et vérifier la sortie avec **AJV** (via CDN).

### Option 2 — En lot (Node CLI) _(facultatif)_

Si besoin de 1 000+ documents, prévoir :

- `generate.js` : génère en JSON/JSONL, paramétrable (`--matches`, `--seed`, `--max-per-skill`…)
- `validate.js` : valide chaque fichier contre `schema.json` (lib **ajv**)

---

## 🧪 Tests rapides

1. **Déterminisme** : mets `seed=42` → Générer → Télécharge → régénère avec le même seed → fichiers identiques.
2. **Exhaustivité skills** : chaque joueur possède les 13 skills.
3. **Bornes** : `maxPerSkill=0` → aucun événement (`count` uniquement).
4. **Enums** : `division`, `section`, `zone` font partie des listes autorisées.
5. **Cohérence** : aucun `freethrow` avec une zone ≠ `freethrows`.

---

## 📦 Dossier `examples/`

Contient des exemples validés :

- `match-small.json` (peu d’actions)
- `match-zero.json` (`maxPerSkill=0`)
- `matches-10.json` (lot)

---

## ❓ Dépannage

- **fetch bloque en local** → Utilise un **serveur** (voir _Prérequis_).
- **Erreur enum** → Ajoute/édite la valeur dans `/data` puis régénère.
- **Structure divergente** → Vérifie le mapping joueurs (`NOM`/`PRENOM`/`NUMERO`) et la présence des 13 skills.
- **JSON invalide** → Lance la **validation AJV** pour voir l’erreur exacte.

---

## 📄 Licence

Projet interne / client — vérifier la politique de diffusion avant publication.
