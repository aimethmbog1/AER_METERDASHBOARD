# AER Meter Dashboard — Statistiques des compteurs

Application Streamlit indépendante pour visualiser les statistiques des **compteurs
(abonnés) installés**, par localité et par période d'installation, à partir des
exports du système de gestion des compteurs prépayés **Holley**.

Reprend la palette visuelle du tableau de bord AER (parc solaire / ressources
humaines) pour une cohérence entre outils AER, mais ne dépend d'aucun autre dépôt :
elle lit uniquement les fichiers Excel que tu lui déposes.

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

## Les cinq onglets

- **📊 Vue d'ensemble** : indicateurs clés (compteurs installés, localités et postes
  couverts, dernière période, part de compteurs solaires), top 10 des localités,
  rythme d'installation récent, et traçabilité par lot (un fichier déposé = un lot).
- **📍 Par localité** : classement des localités par nombre de compteurs installés,
  filtre par poste, répartition par type de compteur (réseau / solaire), tableau
  exportable.
- **📅 Par période** : nouvelles installations et cumul dans le temps, avec un choix
  de granularité (mois / trimestre / année) et un filtre de plage de dates.
- **🧭 Localité × période** : carte de chaleur et tableau croisé, avec un choix entre
  le Top 12 des localités les plus équipées et la totalité des localités, pour
  repérer où et quand l'effort d'installation s'est concentré.
- **🏗️ Par phase** : compteurs installés par phase du projet, évolution dans le temps
  (graphe empilé et cumul) et tableau croisé phase × période. Aucune colonne
  « phase » n'existe dans les exports Holley : les phases sont définies par des
  plages de dates **éditables dans la barre latérale** (« 🏗️ Phases du projet »),
  pré-remplies en 3 tiers égaux à corriger avec les vraies dates de chaque phase.
  Un compteur dont la période ne tombe dans aucune plage définie est compté à part,
  sous « (hors phase) », jamais rattaché silencieusement à une phase voisine.

Un export complet (toutes les lignes, colonnes nettoyées, y compris la phase) est
aussi disponible en bas de page une fois de vrais fichiers déposés.

## Déploiement

Le projet est prêt pour [Streamlit Community Cloud](https://streamlit.io/cloud) :
poussez ce dossier sur un dépôt Git et pointez le déploiement vers `streamlit_app.py`.
