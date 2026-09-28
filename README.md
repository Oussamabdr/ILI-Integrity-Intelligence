# ILI Integrity Intelligence

Application web full-stack pour la réception, le contrôle qualité, la validation et l'analyse des rapports d'inspection ILI des canalisations.

## Technologies retenues

- Frontend: JavaScript ES Modules, HTML5, CSS3 responsive, interface servie par Express.
- Backend: Node.js avec Express 5.
- Base de données: SQLite avec `node:sqlite`, clés étrangères, transactions et mode WAL.
- Sécurité: JWT, rôles, Helmet, rate limiting, validation Zod, limite d'upload, audit log et empreinte SHA-256 des fichiers.
- Import de rapports: CSV avec `@fast-csv/parse`, XLSX avec lecture ZIP/XML via `jszip` et `fast-xml-parser`.

Le choix frontend actuel est volontairement léger et maîtrisable pour un PFE: pas de framework imposant tant que les besoins sont surtout formulaires, tableaux, rapports et appels API. L'architecture reste compatible avec une migration future vers React ou Vue si nécessaire.

## Objectif 1: Gestion de la réception des rapports

Le module de réception couvre les exigences suivantes:

- Enregistrer les campagnes d'inspection avec code, pipeline, prestataire, date, outil ILI et version attendue.
- Identifier la canalisation inspectée à partir du référentiel `pipelines`.
- Gérer le prestataire et la date d'inspection dans la table `campaigns`.
- Réceptionner les fichiers de rapports au format CSV/XLSX.
- Gérer les versions successives des rapports dans `report_files`.
- Marquer le dernier rapport importé comme actif et conserver les anciennes versions avec le statut `SUPERSEDED`.
- Vérifier la présence des données requises par détection automatique du schéma.
- Assurer la traçabilité: nom original, nom stocké, taille, MIME type, SHA-256, utilisateur, date d'import, version, score qualité et audit log.

## Objectif 2: Contrôle automatique de qualité des données

Le système produit un Data Quality Report par campagne et par rapport actif. Les règles implémentées couvrent:

- Données manquantes: type d'anomalie, profondeur, localisation exploitable.
- Doublons: identifiant d'anomalie dupliqué.
- Valeurs hors domaine: profondeur hors 0-100% WT, confiance hors 0-1, position horaire invalide.
- Incohérences entre champs: incohérence PK/distance cumulée, type/profondeur incohérents.
- Erreurs de localisation: PK négatif, distance négative, PK supérieur à la longueur du pipeline.
- Incohérences de kilométrage: PK non croissant entre lignes successives.
- Anomalies géométriques ou spatiales: dimensions invalides, ratio largeur/longueur atypique, coordonnées incomplètes ou hors domaine.
- Écarts par rapport au modèle attendu: colonnes obligatoires et recommandées non détectées.

Chaque écart est stocké avec catégorie, règle, sévérité, ligne source, champ concerné et message. Le frontend affiche le score qualité, les écarts par catégorie, les règles les plus fréquentes et le détail des anomalies de qualité.

Deux fichiers d'essai sont fournis:

- `public/sample-report.csv`: rapport conforme pour tester la réception nominale.
- `public/sample-quality-issues.csv`: rapport avec erreurs volontaires pour tester le Data Quality Report.
- `public/sample-real-ili-report.csv`: exemple proche du format ILI réel avec les colonnes `Feature #`, `Abs. distance (m)`, `Joint number`, `Feature type`, `Depth (%)`, `ERF`, pressions B31G et commentaires.

## Objectif 3: Validation intelligente du rapport ILI

La phase de validation terrain est préparée par un mécanisme de sélection automatique des anomalies. La sélection prend déjà en compte:

- Profondeur de l'anomalie.
- Type d'anomalie.
- Criticité calculée.
- Niveau de confiance de l'inspection.
- Score de risque.

Les critères suivants sont prévus pour l'itération suivante:

- Localisation et distribution géographique.
- Densité d'anomalies par tronçon.
- Historique des inspections précédentes.
- Accessibilité du site.
- Importance stratégique de la canalisation.
- Représentativité statistique de l'échantillon.

## Lancement

```bash
npm install
npm run seed
npm start
```

Ouvrir ensuite:

```text
http://localhost:4200
```

Compte initial:

```text
Email: admin@exemple.dz
Mot de passe: Admin123!
```
