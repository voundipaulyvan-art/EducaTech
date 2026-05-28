# EducaTech
# 🎓 EduTech — Entrepôt de Données Décisionnel

> **Projet d'Informatique Décisionnelle (BI) — EPSI 2026**  
> Analyse des performances et de l'abandon étudiant sur une plateforme universitaire en ligne

---

## 📊 Problématique

La plateforme EduTech fait face à des indicateurs préoccupants :

| Indicateur | Valeur |
|---|---|
| Taux d'abandon | **31,16 %** |
| Taux d'échec | **21,64 %** |
| Taux de réussite | **37,93 %** |
| Distinction | **9,28 %** |

**Objectif :** Comprendre les facteurs d'abandon et identifier les étudiants à risque pour intervenir en amont.

---

## Architecture de la Solution

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   ZONE BRONZE   │    │   ZONE ARGENT   │    │    ZONE OR      │
│   (Input)       │───▶│  (Traitement)   │───▶│   (Output)      │
│                 │    │                 │    │                 │
│ 7 CSV bruts     │    │ Talend +        │    │ 4 CSV propres   │
│ OULAD Dataset   │    │ Python (pandas) │    │ + MySQL DW      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
                                              ┌─────────────────┐
                                              │   Power BI      │
                                              │   Dashboard     │
                                              │   (3 pages)     │
                                              └─────────────────┘
```

---

## Structure du Projet

```
EduTech_Project/
├── 📂 input/                          # Zone Bronze — Données brutes OULAD
│   ├── studentInfo.csv                # Profil démographique (3 381 Ko)
│   ├── studentAssessment.csv          # Scores aux évaluations (5 557 Ko)
│   ├── studentVle.csv                 # Clics plateforme (443 200 Ko — 10,6M lignes)
│   ├── courses.csv                    # Informations modules
│   ├── assessments.csv                # Types et poids des évaluations
│   ├── studentRegistration.csv        # Inscriptions/désinscriptions
│   └── vle.csv                        # Ressources pédagogiques
│
├── 📂 output/                         # Zone Or — Données nettoyées
│   ├── studentInfo_clean.csv          # 32 593 lignes
│   ├── studentAssessment_clean.csv    # 173 912 lignes
│   ├── studentVle_agrege.csv          # 28 237 lignes
│   └── FAIT_PERFORMANCE_ETUDIANT.csv  # 32 593 lignes (table de faits)
│
├── 📂 scripts py/                     # Scripts ETL Python
│   ├── clean_studentinfo.py           # Nettoyage studentInfo
│   ├── clean_studentassessment.py     # Nettoyage studentAssessment
│   ├── aggregate_vle.py               # Agrégation clics VLE
│   ├── build_fact.py                  # Construction table de faits
│   └── import_mysql.py                # Import CSV → MySQL
│
├── 📊 EduTech.pbix                    # Dashboard Power BI
├── 🗄️ edutech_dw.sql                  # Structure base de données MySQL
└── 📄 EduTech_CodeSource.txt          # Code source complet (Python + SQL + DAX)
```

---

## Stack Technique

| Outil | Version | Rôle |
|---|---|---|
| **Talend Open Studio** | 8.0.1 | ETL visuel — nettoyage des données |
| **Python** | 3.12.3 | ETL complémentaire — traitement gros volumes |
| **pandas** | Latest | Manipulation et transformation des données |
| **MySQL** | 8 (WAMP) | Stockage — Data Warehouse |
| **phpMyAdmin** | Latest | Administration de la base de données |
| **Power BI Desktop** | Latest | Dashboard interactif — restitution BI |
| **Java** | JDK 11 (Temurin) | Requis pour Talend Open Studio |

---

## Modélisation — Schéma en Étoile

```
                    ┌─────────────────┐
                    │   dim_temps     │
                    │─────────────────│
                    │ code_presentation│
                    │ annee           │
                    │ semestre        │
                    └────────┬────────┘
                             │
┌──────────────┐    ┌────────▼──────────────────┐    ┌─────────────────┐
│  dim_module  │    │  fait_performance_etudiant │    │  dim_etudiant   │
│──────────────│    │───────────────────────────│    │─────────────────│
│ code_module  │◄───│ id_student (FK)            │───►│ id_student      │
│ code_present.│    │ code_module (FK)           │    │ gender          │
│ length       │    │ code_presentation (FK)     │    │ region          │
└──────────────┘    │ id_assessment (FK)         │    │ highest_educ.   │
                    │ final_result               │    │ imd_band        │
                    │ score_moyen ← MESURE       │    │ age_band        │
                    │ total_clics ← MESURE       │    │ disability      │
                    │ flag_a_risque ← MESURE     │    └─────────────────┘
                    └────────┬──────────────────-┘
                             │
                    ┌────────▼────────┐
                    │  dim_evaluation │
                    │─────────────────│
                    │ id_assessment   │
                    │ code_module     │
                    │ assessment_type │
                    │ date_assessment │
                    │ weight          │
                    └─────────────────┘
```

### Tables & Lignes

| Table | Lignes | Description |
|---|---|---|
| `dim_etudiant` | 28 785 | Profil démographique des étudiants |
| `dim_module` | 22 | Modules de cours (AAA→GGG) |
| `dim_temps` | 4 | Sessions académiques |
| `dim_evaluation` | 206 | Évaluations (TMA, CMA, Examen) |
| `fait_performance_etudiant` | 32 593 | Table de faits centrale |

---

## ⚙️ Processus ETL

### Jobs Talend

| Job | Transformation | Résultat |
|---|---|---|
| `Clean_StudentInfo` | `imd_band` null → `"Unknown"` | 32 593 lignes |
| `Clean_StudentAssessment` | `score` null → `0` | 173 912 lignes |

> **Note Talend :** Problème de compatibilité avec Java 21 résolu en installant **Java 11 (Eclipse Temurin JDK 11.0.31.11)** et en modifiant `TOS_BD-win-x86_64.ini`.

### Scripts Python

| Script | Rôle | Résultat |
|---|---|---|
| `clean_studentinfo.py` | Nettoyage `imd_band` | 32 593 lignes |
| `clean_studentassessment.py` | Nettoyage `score` | 173 912 lignes |
| `aggregate_vle.py` | SUM clics par étudiant × module | 28 237 lignes |
| `build_fact.py` | Construction table de faits + `flag_a_risque` | 32 593 lignes |
| `import_mysql.py` | Import CSV → MySQL `edutech_dw` | Toutes tables |

### Règles de Transformation

```python
# imd_band vide → "Unknown"
df['imd_band'] = df['imd_band'].fillna('Unknown').replace('', 'Unknown')

# score null → 0
df['score'] = df['score'].fillna(0)

# Agrégation clics
result = df.groupby(['id_student', 'code_module'])['sum_click'].sum()

# Flag à risque : score < 40 ET clics < 200
df['flag_a_risque'] = ((df['score_moyen'] < 40) & (df['total_clics'] < 200)).astype(int)
```

---

## 📈 KPIs & Mesures DAX

| KPI | Valeur | Mesure DAX |
|---|---|---|
| Score moyen | **59,65 / 100** | `AVERAGE(fait[score_moyen])` |
| Taux d'abandon | **31,16 %** | `DIVIDE(COUNTROWS(FILTER(...,"Withdrawn")), COUNTROWS(...))` |
| Taux d'échec | **21,64 %** | `DIVIDE(COUNTROWS(FILTER(...,"Fail")), COUNTROWS(...))` |
| Étudiants à risque | **~6 000** | `SUM(fait[flag_a_risque])` |
| Engagement moyen | **1 270 clics** | `AVERAGE(fait[total_clics])` |

---

## Dashboard Power BI

Le dashboard est structuré en **3 pages** :

### Page 1 — Vue Générale
- 6 KPI Cards (score, abandon, échec, risque, clics, nb étudiants)
- Graphique à barres : score moyen par module
- Camembert : répartition Pass / Withdrawn / Fail / Distinction
- Treemap : étudiants à risque par région
- 2 Slicers : `code_module` et `final_result`

### Page 2 — Détail Module (Drill-through)
- Tableau détaillé filtré au clic droit sur un module

### Page 3 — Tooltip
- Info-bulle personnalisée au survol des visuels

### Conclusions
- Module **FFF** : meilleur score moyen
- Module **AAA** : score le plus faible
- **Londres** : plus forte concentration d'étudiants à risque

---

## Installation & Utilisation

### Prérequis
```
- Python 3.12+
- WAMP Server (MySQL 8 + phpMyAdmin)
- Power BI Desktop
- Talend Open Studio 8 + Java JDK 11
```

### 1. Cloner le projet
```bash
git clone https://github.com/[votre-username]/EduTech-BI.git
cd EduTech-BI
```

### 2. Installer les dépendances Python
```bash
pip install pandas mysql-connector-python
```

### 3. Créer la base de données MySQL
```sql
-- Dans phpMyAdmin ou MySQL Workbench
SOURCE edutech_dw.sql;
```

### 4. Lancer l'ETL Python
```bash
cd "scripts py"
python clean_studentinfo.py
python clean_studentassessment.py
python aggregate_vle.py
python build_fact.py
python import_mysql.py
```

### 5. Ouvrir le Dashboard
```
Ouvrir EduTech.pbix dans Power BI Desktop
```

---

## Data Marts

| Vue SQL | Destinataire | Objectif |
|---|---|---|
| `datamart_performance` | CEO | Performances académiques par module et session |
| `datamart_risque` | Dir. Opérations | Étudiants à risque avec profil démographique |

---

## Dataset

**OULAD** — Open University Learning Analytics Dataset  
Source : [Open University UK](https://analyse.kmi.open.ac.uk/open_dataset)

| Fichier | Taille | Lignes |
|---|---|---|
| studentInfo.csv | 3 381 Ko | 32 593 |
| studentAssessment.csv | 5 557 Ko | 173 912 |
| studentVle.csv | 443 200 Ko | ~10,6M |
| courses.csv | 1 Ko | 22 |
| assessments.csv | 9 Ko | 206 |
| studentRegistration.csv | 1 084 Ko | ~32K |
| vle.csv | 255 Ko | ~6K |

---

## Décideurs Cibles

| Persona | Rôle | Besoins |
|---|---|---|
| **CEO** | Vision stratégique | Performance globale, tendances par module |
| **Dir. Opérations** | Vision opérationnelle | Étudiants à risque, alertes précoces |

---

## Licence

Projet académique — EPSI 2026 — Informatique Décisionnelle  
Dataset OULAD sous licence [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/)

---

<div align="center">
  <strong>🎓 EduTech BI Project — EPSI 2026</strong><br>
  <em>Talend • Python • MySQL • Power BI</em>
</div>
