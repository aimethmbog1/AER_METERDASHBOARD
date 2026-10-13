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
- **Barres de saturation** (onglet « ⚡ Saturation des sites ») : une ligne = un
  site, échelle fixée à 150 % de la capacité théorique (repère vertical au palier
  des 100 %) pour que le dépassement reste visible plutôt que de saturer la barre
  à l'identique d'un site pile à l'objectif — seul le dépassement réel respire
  (même logique que le badge « Installations actives »).
- Les messages natifs Streamlit (`st.info`/`st.warning`/`st.success`/`st.error`)
  sont ramenés au même panneau sombre que le reste de l'interface (liseré de
  couleur plutôt que fond pastel clair) — un message natif reste un usage
  secondaire/explicatif ici ; le vrai statut passe toujours par le badge dédié.
- **Navigation par barre latérale** (et non plus par onglets en haut de page) :
  les huit sections apparaissent comme un menu de navigation dans la barre de
  gauche, avec surlignage de la section active — inspiré des tableaux de bord
  d'administration fournis en référence par l'utilisateur.
- **Cartes KPI colorées** : chaque indicateur clé (hors bandeau central, qui
  reste neutre) est présenté dans une carte avec un liseré de couleur
  d'identité — tirée de la **même palette catégorielle validée** que les
  graphiques multi-séries (8 teintes, jamais les couleurs de statut
  émeraude/ambre/rouge) : de la variété visuelle sans rouvrir le compromis
  « néon réservé aux états » déjà retenu pour cette application. Les
  indicateurs qui portent une variation réelle (ex. « Par période ») restent
  en revanche affichés en métriques natives Streamlit, pour garder la flèche
  de tendance colorée — une carte KPI ne la montre pas.
- **Graphiques et listes encadrés** dans des cartes à bordure, pour un effet
  de grille de widgets plus proche des tableaux de bord de référence.
- **Jauge (gauge)** dans « Vue d'ensemble » pour le taux de couverture du
  référentiel officiel — seul indicateur de l'application à la fois réel et
  naturellement adapté à ce type de widget ; les éléments décoratifs mais non
  représentatifs des maquettes de référence (carte du monde, graphique radar
  sans axes naturels dans ces données) n'ont volontairement pas été repris.

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
   `data/puissances_sites_aer.csv` embarque, du même classeur et de la même feuille,
   la puissance installée (kWc) de ces mêmes 499 sites — utilisée uniquement par
   l'onglet « ⚡ Saturation des sites » ci-dessous. Dans les deux cas, c'est un
   instantané statique de deux colonnes du classeur, pris une fois pour toutes :
   cette application ne lit jamais ce classeur à l'exécution. Une localité qui
   ressemble fortement à un de ces sites est corrigée vers son orthographe
   officielle (ex. `DOUKOUR0Y` → `Doukouroy`, `DJIGBAMAFOUS` / `NJIBAMAFOU` →
   `Njigbamafou`).
2. **Regroupement interne** pour les localités restantes (coquilles, casse, espaces
   superflus, noms dupliqués) — comme avant, regroupées sous la variante la plus
   fréquente.

La colonne « Origine de la correction » indique d'où vient chaque proposition. Dans
les deux cas, la proposition reste éditable : rien n'est fusionné silencieusement.
Beaucoup de localités des exports Holley ne correspondent à aucun site officiel
(sous-localités, quartiers) — c'est normal, elles passent simplement par le
regroupement interne.

## Filtre global

Un panneau de filtres (« 🔎 Filtres »), juste sous l'expander d'introduction, en
haut de page : **localité, poste, région, phase, type de compteur, plage de
dates**. Il s'applique à **toute l'application** (bandeau central, les huit
sections, l'export CSV et le rapport PDF) dès qu'une valeur est choisie — pas
seulement à la section affichée. Remplace les filtres qui n'existaient
auparavant que dans certaines sections (l'ancien filtre par poste de « Par
localité », l'ancienne plage de dates de « Par période »), pour n'avoir qu'un
seul endroit où filtrer. Le nombre de compteurs affichés par rapport au total
chargé est indiqué dès qu'un filtre réduit les données ; un bouton réinitialise
tous les filtres d'un coup.

## Les huit sections (barre de navigation latérale)

- **📊 Vue d'ensemble** : indicateurs clés (compteurs installés, localités et postes
  couverts, dernière période, part de compteurs solaires), top 10 des localités,
  jauge de couverture du référentiel officiel, rythme d'installation récent, et
  traçabilité par lot (un fichier déposé = un lot).
- **📍 Par localité** : classement des localités par nombre de compteurs installés,
  répartition par type de compteur (réseau / solaire), tableau exportable (filtre
  par poste désormais dans le panneau de filtres global, en haut de page).
- **📅 Par période** : nouvelles installations et cumul dans le temps, avec un choix
  de granularité (mois / trimestre / année) et la **variation vs la période
  précédente** (delta natif Streamlit) — la plage de dates se règle dans le panneau
  de filtres global, en haut de page.
- **🧭 Localité × période** : carte de chaleur et tableau croisé, avec un choix entre
  le Top 12 des localités les plus équipées et la totalité des localités, pour
  repérer où et quand l'effort d'installation s'est concentré.
- **🏗️ Par phase** : compteurs installés par phase du projet, **objectifs vs réalisé**
  (cibles éditables dans la barre latérale « 🎯 Objectifs du projet » — aucune cible
  chiffrée n'existe dans les exports Holley, à saisir manuellement ; statut vert/
  ambre/rouge à 100 % / 80 % / en dessous de l'objectif, seuils assumés par défaut),
  évolution dans le temps (graphe empilé et cumul) et tableau croisé phase × période.
  Aucune colonne « phase » n'existe dans les exports Holley : deux méthodes de
  classement, dans cet ordre de priorité — (1) le **préfixe du numéro de compteur**
  (`numero_compteur`, colonne « Old Consumer No. ») : **251** → 1re phase définie
  ci-dessous, **241** → 2e phase, **5** → 3e phase (association par ordre des lignes,
  pas par nom ; convention communiquée par l'utilisateur, pas une règle officielle
  documentée — à ajuster dans `utils/compteurs.py`, `PREFIXES_COMPTEUR_PHASE`, si une
  autre convention s'applique) ; (2) à défaut, pour un numéro ne correspondant à
  aucun de ces préfixes, la **plage de dates** définie et modifiable dans la barre
  latérale (« 🏗️ Phases du projet »), pré-remplie en 3 tiers égaux à corriger avec
  les vraies dates de chaque phase. Un compteur classé par aucune des deux méthodes
  est compté à part, sous « (hors phase) », jamais rattaché silencieusement à une
  phase voisine. Les cas où les deux méthodes se contredisent (préfixe connu, mais
  date hors de la plage de la phase correspondante) sont comptés et détaillés dans
  l'onglet « 🔍 Qualité des données », jamais résolus en silence.
- **🌍 Par région** *(nouveau — pas une carte géographique)* : répartition des
  compteurs par région et par antenne, à partir du rattachement **exact** entre la
  localité (déjà corrigée) et le référentiel des 499 sites officiels. `data/
  sites_officiels_aer.csv` ne contient aucune coordonnée GPS (contrairement au
  classeur Excel du tableau de bord principal, que cette application ne lit
  volontairement pas) : c'est un regroupement par région, pas une carte. Une
  localité sans correspondance dans le référentiel apparaît sous
  « (hors référentiel) ».
- **⚡ Saturation des sites** *(nouveau)* : taux de saturation de chaque site officiel —
  compteurs installés rapportés à sa **capacité théorique**, définie comme la
  puissance du site (kWc) × 1 000 ÷ **400 W par compteur**, arrondie à l'entier
  inférieur. Convention de dimensionnement communiquée par l'utilisateur, pas une
  valeur officielle retrouvée dans les fichiers de l'application — à ajuster dans
  `utils/compteurs.py` (`WATTS_PAR_COMPTEUR`) si une autre règle s'applique. Seuls les
  sites du référentiel officiel apparaissent (puissance connue) ; une localité
  installée hors référentiel reste signalée dans l'onglet « 🌍 Par région », jamais
  ici avec une capacité devinée. Un site jamais encore équipé apparaît avec un taux de
  0 % (jamais omis). Seuils (assumés, mêmes paliers que les objectifs par phase) :
  vert en dessous de 80 % de la capacité, ambre de 80 % à 100 %, rouge au-delà —
  un dépassement réel (compteurs installés > capacité théorique) est signalé
  explicitement, avec le détail des sites concernés, pour vérification manuelle
  (extension de puissance non répercutée ici, compteurs partagés avec un poste
  voisin, ou dimensionnement initial sous-estimé — cette application ne tranche pas
  laquelle).
- **🔍 Qualité des données** : anomalies à vérifier manuellement —
  compteurs en doublon (même numéro dans plusieurs lignes, risque de double
  comptage), postes sans nouvelle installation depuis plus de 90 jours (seuil
  assumé par défaut), **désaccords entre le préfixe du numéro de compteur et la
  plage de dates de la phase** (tableau détaillé, les deux classements côte à
  côte), part de lignes sans localité ou poste lisible. Rien n'est corrigé ni
  retiré automatiquement des statistiques des autres onglets — ces indicateurs
  (doublons, postes inactifs) sont aussi résumés en continu juste sous le
  bandeau central, avant même d'ouvrir cet onglet.

Un **taux de couverture des 499 localités officielles** (combien ont au moins un
compteur installé) ainsi qu'un badge **sites en sursaturation** sont affichés en
continu sous le bandeau central. Un export complet (toutes les lignes, colonnes
nettoyées, y compris la phase) et un **rapport de synthèse PDF** (KPI, top
localités, objectifs vs réalisé, répartition régionale, sites en dépassement de
capacité, alertes qualité — généré à la demande, pratique pour un reporting
institutionnel) sont disponibles en bas de page une fois de vrais fichiers déposés.
Si la génération des images de graphiques échoue sur l'hébergement choisi (la
librairie `kaleido` a besoin d'un Chrome headless, pas toujours disponible), le
PDF se génère quand même, sans les graphiques, avec une note explicite plutôt
qu'un plantage.

## Déploiement

Le projet est prêt pour [Streamlit Community Cloud](https://streamlit.io/cloud) :
poussez ce dossier sur un dépôt Git et pointez le déploiement vers `streamlit_app.py`.
