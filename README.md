# AER Meter Dashboard — Statistiques des compteurs

Application Streamlit indépendante pour visualiser les statistiques des **compteurs
(abonnés) installés**, par localité et par période d'installation, à partir des
exports du système de gestion des compteurs prépayés **Holley**.

Elle lit uniquement les fichiers Excel que tu lui déposes, indépendamment de tout
autre dépôt.

## Identité visuelle — panneau de supervision (dark/néon)

Cette application a sa **propre identité visuelle**, volontairement différente du
reste de la suite AER Streamlit (qui garde la palette claire navy/bleu/or du
classeur Excel) : un thème sombre façon panneau de supervision industrielle,
avec un compromis assumé — *« néon réservé aux états »*.

- **Base très majoritairement neutre et sombre** ; la couleur saturée
  (émeraude/ambre/rouge/bleu) n'apparaît que sur ce qui a un état réel (actif,
  alerte, information) — jamais comme simple décor. Ce compromis s'inspire de la
  pratique professionnelle des interfaces de supervision (philosophie ISA-101
  « high-performance HMI »), qui réserve la couleur saturée à l'anormal ; aucune
  information n'est codée uniquement par la couleur (icône + libellé toujours
  présents).
- **Compteur total animé** au centre de gravité visuel de l'application (en
  haut de la page), avec un effet de compte-à-rebours progressif
  (`easeOutQuad`) et une respiration lente réservée au badge de statut
  « Installations actives ».
- **Flux animé** postes Holley → localités → total, juste en dessous.
- Toutes les animations respectent `prefers-reduced-motion` (désactivées si la
  personne a demandé moins de mouvement dans son système).
- Palette catégorielle des graphiques multi-séries (types de compteur, phases) :
  validée par la skill de data-visualisation (8 teintes, séparation
  daltonisme-compatible vérifiée), volontairement distincte des couleurs de
  statut.

## Installation

```bash
python -m venv .venv
source .venv/bin/activate          # Windows : .venv\Scripts\activate
pip install -r requirements.txt
```

## Lancement

```bash
streamlit run streamlit_app.py
```

L'application s'ouvre sur `http://localhost:8501`. Sans fichier déposé, elle tourne
sur un **jeu de données d'exemple** (localités et dates fictives) pour montrer le
fonctionnement.

## Format de fichier attendu

Dépose un ou plusieurs fichiers Excel dans la barre latérale (« 📥 Données
compteurs »). Deux formats sont pris en charge :

- **Export Holley** (le format réel des fichiers AER, reconnu automatiquement grâce
  aux colonnes `Full Address` / `Template Code`) : chaque fichier représente un
  **lot d'abonnés installés**, et c'est la **date présente dans le nom du fichier**
  (ex. `01-10-2024.xls`) qui sert de période d'installation pour toutes ses lignes
  — la colonne `Opening Date` de ces exports est un champ système figé à
  01/01/2018 et n'est donc pas utilisée. La date détectée dans chaque nom de
  fichier est affichée et modifiable dans la barre latérale avant de lancer les
  statistiques, et un tableau de traçabilité par lot (onglet « Vue d'ensemble »)
  permet de la vérifier. La colonne `Holley System ID` (ex. « 31K433 ») est un
  **poste** — l'équipement/poste de configuration Holley ayant servi à
  enregistrer les compteurs — et non un site géographique : le vrai équivalent
  « site » de cette application, c'est la **localité** (voir ci-dessous).
- **Format générique** (si les colonnes ci-dessus sont absentes) : une
  correspondance manuelle des colonnes est proposée (quelle colonne est la
  localité, quelle colonne est la date d'installation...), pour un fichier à la
  fois.

Dans les deux cas, un encart « 🧹 Vérifier les localités » (barre latérale) propose
une correction pour chaque localité détectée, en deux passes :

1. **Rapprochement d'un site officiel AER** — `data/sites_officiels_aer.csv` embarque
   les 499 sites du parc solaire (nom, région, département, antenne), extraits du
   classeur `Gestion_Parc_Solaire_AER2027_DASHBOARD.xlsx` (feuille « Données sites »).
   Une localité qui ressemble fortement à un de ces sites est corrigée vers son
   orthographe officielle (ex. `DOUKOUR0Y` → `Doukouroy`, `DJIGBAMAFOUS` /
   `NJIBAMAFOU` → `Njigbamafou`).
2. **Regroupement interne** pour les localités restantes (coquilles, casse, espaces
   superflus, noms dupliqués) — comme avant, regroupées sous la variante la plus
   fréquente.

La colonne « Origine de la correction » indique d'où vient chaque proposition. Dans
les deux cas, la proposition reste éditable : rien n'est fusionné silencieusement.
Beaucoup de localités des exports Holley ne correspondent à aucun site officiel
(sous-localités, quartiers) — c'est normal, elles passent simplement par le
regroupement interne.

## Les sept onglets

- **📊 Vue d'ensemble** : indicateurs clés (compteurs installés, localités et postes
  couverts, dernière période, part de compteurs solaires), top 10 des localités,
  rythme d'installation récent, et traçabilité par lot (un fichier déposé = un lot).
- **📍 Par localité** : classement des localités par nombre de compteurs installés,
  filtre par poste, répartition par type de compteur (réseau / solaire), tableau
  exportable.
- **📅 Par période** : nouvelles installations et cumul dans le temps, avec un choix
  de granularité (mois / trimestre / année), un filtre de plage de dates, et la
  **variation vs la période précédente** (delta natif Streamlit).
- **🧭 Localité × période** : carte de chaleur et tableau croisé, avec un choix entre
  le Top 12 des localités les plus équipées et la totalité des localités, pour
  repérer où et quand l'effort d'installation s'est concentré.
- **🏗️ Par phase** : compteurs installés par phase du projet, **objectifs vs réalisé**
  (cibles éditables dans la barre latérale « 🎯 Objectifs du projet » — aucune cible
  chiffrée n'existe dans les exports Holley, à saisir manuellement ; statut vert/
  ambre/rouge à 100 % / 80 % / en dessous de l'objectif, seuils assumés par défaut),
  évolution dans le temps (graphe empilé et cumul) et tableau croisé phase × période.
  Aucune colonne « phase » n'existe dans les exports Holley : les phases sont
  définies par des plages de dates **éditables dans la barre latérale**
  (« 🏗️ Phases du projet »), pré-remplies en 3 tiers égaux à corriger avec les
  vraies dates de chaque phase. Un compteur dont la période ne tombe dans aucune
  plage définie est compté à part, sous « (hors phase) », jamais rattaché
  silencieusement à une phase voisine.
- **🌍 Par région** *(nouveau — pas une carte géographique)* : répartition des
  compteurs par région et par antenne, à partir du rattachement **exact** entre la
  localité (déjà corrigée) et le référentiel des 499 sites officiels. `data/
  sites_officiels_aer.csv` ne contient aucune coordonnée GPS (contrairement au
  classeur Excel du tableau de bord principal, que cette application ne lit
  volontairement pas) : c'est un regroupement par région, pas une carte. Une
  localité sans correspondance dans le référentiel apparaît sous
  « (hors référentiel) ».
- **🔍 Qualité des données** *(nouveau)* : anomalies à vérifier manuellement —
  compteurs en doublon (même numéro dans plusieurs lignes, risque de double
  comptage), postes sans nouvelle installation depuis plus de 90 jours (seuil
  assumé par défaut), part de lignes sans localité ou poste lisible. Rien n'est
  corrigé ni retiré automatiquement des statistiques des autres onglets — ces
  deux indicateurs (doublons, postes inactifs) sont aussi résumés en continu
  juste sous le bandeau central, avant même d'ouvrir cet onglet.

Un **taux de couverture des 499 localités officielles** (combien ont au moins un
compteur installé) est affiché en continu sous le bandeau central. Un export
complet (toutes les lignes, colonnes nettoyées, y compris la phase) et un
**rapport de synthèse PDF** (KPI, top localités, objectifs vs réalisé, répartition
régionale, alertes qualité — généré à la demande, pratique pour un reporting
institutionnel) sont disponibles en bas de page une fois de vrais fichiers déposés.
Si la génération des images de graphiques échoue sur l'hébergement choisi (la
librairie `kaleido` a besoin d'un Chrome headless, pas toujours disponible), le
PDF se génère quand même, sans les graphiques, avec une note explicite plutôt
qu'un plantage.

## Déploiement

Le projet est prêt pour [Streamlit Community Cloud](https://streamlit.io/cloud) :
poussez ce dossier sur un dépôt Git et pointez le déploiement vers `streamlit_app.py`.
