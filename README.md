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
  permet de la vérifier.
- **Format générique** (si les colonnes ci-dessus sont absentes) : une
  correspondance manuelle des colonnes est proposée (quelle colonne est la
  localité, quelle colonne est la date d'installation...), pour un fichier à la
  fois.

Dans les deux cas, un encart « 🧹 Vérifier les localités » (barre latérale) propose
un regroupement automatique des variantes d'orthographe détectées dans les noms de
localités (coquilles, casse, espaces superflus, noms dupliqués — fréquents dans les
données réelles). La proposition est pré-remplie mais reste éditable : rien n'est
fusionné silencieusement.

## Les quatre onglets

- **📊 Vue d'ensemble** : indicateurs clés (compteurs installés, localités et sites
  couverts, dernière période, part de compteurs solaires), top 10 des localités,
  rythme d'installation récent, et traçabilité par lot (un fichier déposé = un lot).
- **📍 Par localité** : classement des localités par nombre de compteurs installés,
  filtre par site, répartition par type de compteur (réseau / solaire), tableau
  exportable.
- **📅 Par période** : nouvelles installations et cumul dans le temps, avec un choix
  de granularité (mois / trimestre / année) et un filtre de plage de dates.
- **🧭 Localité × période** : carte de chaleur croisant les 12 localités les plus
  équipées et les périodes d'installation, pour repérer où et quand l'effort
  d'installation s'est concentré — et le tableau croisé correspondant en CSV.

Un export complet (toutes les lignes, colonnes nettoyées) est aussi disponible en
bas de page une fois de vrais fichiers déposés.

## Déploiement

Le projet est prêt pour [Streamlit Community Cloud](https://streamlit.io/cloud) :
poussez ce dossier sur un dépôt Git et pointez le déploiement vers `streamlit_app.py`.
