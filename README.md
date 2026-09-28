# EV Battery Health — Projet Business Intelligence

Projet réalisé dans le cadre du module **Business Intelligence** (4ᵉ année, semestre 8) de l'**EPF Fondation**, encadré par A. Charlemagne.

L'objectif : construire de bout en bout une solution BI complète — du cadrage métier jusqu'au dashboard Power BI — en s'appuyant sur une architecture cloud moderne (**Microsoft Fabric**, Lakehouse, PySpark) et une modélisation en étoile.

**Auteurs :** Demerdjiev, Gharbi, Eymard

---

## 🎯 Cas d'usage métier

**Client fictif :** un constructeur automobile qui se lance dans l'électrique et cherche à comprendre quels facteurs accélèrent la dégradation des batteries de ses véhicules.

**Utilisateur final :** le département d'ingénierie du constructeur (expertise data limitée → besoin d'une vue d'ensemble simple et actionnable).

**Problème résolu :** relier des dizaines de variables d'usage (température, cycles de charge, style de conduite, charge rapide…) pour identifier les leviers qui préservent l'état de santé (*State of Health*, SoH) des batteries, et permettre :
- l'optimisation de la conception des futures batteries (refroidissement vs. chimie des cellules),
- la planification proactive de la maintenance / du remplacement,
- la définition de profils de référence par type de batterie et style de conduite.

Le projet a été mené selon une démarche agile : recueil du besoin → user stories → backlog → architecture data → dashboard.

---

## 📊 Dataset

- **Source :** [Kaggle – EV Battery Degradation and Charge](https://www.kaggle.com/datasets/bertnardomariouskono/electric-vehicle-ev-battery-degradation-and-charge/data)
- **Volume :** 10 000 véhicules, granularité 1 ligne = 1 véhicule
- **Qualité :** aucune valeur manquante, aucun doublon
- **Variables clés :** modèle de véhicule, type/capacité de batterie, âge du véhicule, cycles de charge, température moyenne, taux de charge rapide, taux de décharge, style de conduite, résistance interne, SoH (%), statut de la batterie (bon état / à remplacer)

---

## 🏗️ Architecture technique — Microsoft Fabric

Pipeline complet construit sur **Microsoft Fabric**, structuré selon l'architecture **Medallion** :

```
CSV source (Kaggle)
      │
      ▼
 ┌─────────────┐   ┌─────────────┐   ┌──────────────────────┐
 │   BRONZE    │──▶│   SILVER    │──▶│         GOLD          │
 │ copie brute │   │ typage,     │   │ tables métier :       │
 │ sans        │   │ nettoyage,  │   │ gold_fact_ev_charging │
 │ transfor-   │   │ NULL, noms  │   │ gold_dim_vehicle      │
 │ mation      │   │ de colonnes │   │ gold_dim_battery      │
 └─────────────┘   └─────────────┘   │ gold_dim_driving_style│
                                      │ gold_dim_battery_status│
                                      └──────────────────────┘
                                                │
                                                ▼
                                      Power BI (import / DirectLake)
```

- **Lakehouse** : stockage central du dataset sous forme de tables Delta (Bronze / Silver / Gold), exposé à Power BI via OneLake.
- **Notebook (PySpark/Python)** : moteur de transformation — un notebook par table, de l'ingestion brute jusqu'aux tables Gold prêtes pour le reporting.
- Choix **ELT** justifié (chargement dans le Lakehouse puis transformation en place, plutôt que transformation avant chargement) et argumenté à l'écrit face aux alternatives (Dataflow Gen2, Warehouse, Data Pipeline) non retenues pour ce périmètre.

---

## ⭐ Modèle sémantique (schéma en étoile)

| Table | Rôle | Contenu |
|---|---|---|
| `gold_fact_ev_charging` | **Fait** | Métriques quantitatives : capacité batterie, température moy., taux de décharge, taux de charge rapide, résistance interne |
| `gold_dim_vehicle` | Dimension | Modèle de véhicule |
| `gold_dim_battery` | Dimension | Type et capacité de batterie |
| `gold_dim_driving_style` | Dimension | Style de conduite (agressif / modéré / prudent) |
| `gold_dim_battery_status` | Dimension | Statut de la batterie (sain / à remplacer) |

Relations en cardinalité `1:*` (dimension → fait), filtre à sens unique, cohérentes avec les bonnes pratiques de modélisation Power BI.

---

## ✅ User stories & KPIs livrés

| Profil | Besoin | KPI / visuel associé |
|---|---|---|
| Ingénieur | Suivre la proportion de batteries avec un SoH > 95 % après 5 ans | Courbe d'évolution + carte KPI |
| Responsable marketing | Identifier le style de conduite le plus favorable à la durabilité | Comparatif SoH par style de conduite |
| Chef de projet | Comparer les types de batteries en charge rapide (> 50 %) | Comparatif par type de batterie / cycles |
| Ingénieur | Estimer le SoH courant à partir de l'historique d'usage | Carte KPI de santé prédictive |

---

## 🛠️ Stack technique

`Microsoft Fabric` · `Lakehouse` · `Notebook (PySpark / Python)` · `Delta Tables` · `Power BI Desktop` · `DAX` · `Modélisation en étoile` · `Méthodologie Agile (User Stories, Backlog)`

---

## 📁 Contenu du dépôt

```
├── LAB1/
│   ├── Business_Intelligence_Lab1.pdf         # Énoncé : cadrage du besoin, choix du dataset
│   └── ev_battery_degradation_v1.csv          # Dataset source (10 000 lignes)
├── LAB2/
│   └── Business_Intelligence_Lab2.pdf         # Énoncé : user stories, backlog, mock-up
├── LAB3/
│   ├── Business_Intelligence_Lab3.pdf         # Énoncé : pipeline Fabric, Medallion, modèle sémantique
│   ├── LAB3_EYMARD_DEMERDJIEV_GHARBI.pbix     # Rapport Power BI (dashboard final)
│   └── template.png                           # Gabarit du mock-up de dashboard
├── LAB3_EYMARD_DEMERDJIEV_GHARBI.pbix         # Copie du rapport Power BI final
├── DEMERDJIEV_GHARBINHO_EYMARD_TP1_TP2_BI.docx  # Livrable écrit : cadrage, user stories, backlog
└── DEMERDJIEV_GHARBINHO_EYMARD_Projet_Power_BI.pdf # Livrable écrit consolidé (Lab 1 à 3)
```

Les fichiers `Business_Intelligence_Lab*.pdf` sont les **énoncés fournis par l'enseignant** ; les fichiers `DEMERDJIEV_GHARBINHO_EYMARD_*` et `LAB3_EYMARD_DEMERDJIEV_GHARBI.pbix` sont les **livrables produits par l'équipe**.

---

## 🧠 Compétences mises en œuvre

- Cadrage d'un besoin métier et rédaction d'un cahier des charges (contexte, scope, KPIs, contraintes)
- Rédaction de user stories et d'un backlog agile orienté pipeline data (ingestion → transformation → modélisation → reporting)
- Modélisation dimensionnelle (schéma en étoile, tables de faits/dimensions, cardinalités, sens de filtrage)
- Mise en œuvre d'une architecture Medallion (Bronze/Silver/Gold) sur Microsoft Fabric
- Transformation de données en PySpark/Python dans des notebooks
- Conception d'un dashboard Power BI orienté décision (KPIs, mesures DAX, filtres)
- Justification argumentée des choix d'architecture (ELT vs ETL, Lakehouse vs Warehouse, etc.)

## ▶️ Consulter le rapport

Ouvrir `LAB3/LAB3_EYMARD_DEMERDJIEV_GHARBI.pbix` avec **Power BI Desktop** pour explorer le modèle de données et le dashboard interactif.
