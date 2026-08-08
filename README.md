# Grilles indiciaires — Fonction Publique Hospitalière (FPH)

Base de données SQL open-source des grilles indiciaires de la Fonction Publique Hospitalière française : corps, grades, échelons, indices bruts/majorés et durées d'échelon.

## Pourquoi ce projet

Les grilles indiciaires de la FPH sont publiées corps par corps (arrêtés, décrets, tableaux PDF), sans jeu de données structuré et réutilisable centralisé. Ce projet vise à combler ce manque en proposant une base SQL simple, versionnée et complétable par la communauté.

⚠️ **Avertissement** : ces données sont saisies manuellement à partir de sources publiques (arrêtés, décrets, sites RH tels que vocationservicepublic.fr ou emploi-collectivites.fr) et n'ont pas de valeur officielle. Elles peuvent contenir des erreurs, des estimations, ou ne pas refléter la version la plus récente d'un texte réglementaire. **Toujours vérifier auprès d'une source officielle (Légifrance, décret concerné) avant tout usage RH, paie ou juridique.**

## Fichiers du projet

| Fichier       | Rôle                                                                                   |
| ------------- | --------------------------------------------------------------------------------------- |
| `grilles.sql` | Création de la table `OUTILS_1_GRILLE` + données indiciaires (échelons, indices, durées) |
| `CORPS.sql`   | Nomenclature de référence `OUTILS_1_CORPS` (corps/grades/codes), utile comme feuille de route pour les contributions à venir |
| `README.md`   | Ce fichier                                                                               |

## Installation rapide

```bash
mysql -u user -p ma_base < grilles.sql
mysql -u user -p ma_base < CORPS.sql   # optionnel : nomenclature de référence corps/grades
```

(Adapter la commande selon votre SGBD : MySQL, PostgreSQL, SQLite...)

## Structure de la table `OUTILS_1_GRILLE`

```sql
CREATE TABLE `OUTILS_1_GRILLE` (
    `Corps`       varchar(100) DEFAULT NULL,
    `Grade`       varchar(100) DEFAULT NULL,
    `Categorie`   varchar(2)   DEFAULT NULL,  -- A, A+, B, C
    `Filiere`     varchar(75)  DEFAULT NULL,  -- ex: Filière hospitalière Administrative
    `CAT`         varchar(3)   DEFAULT NULL,  -- PNM (non médical) ou PM (personnel médical)
    `echelon`     varchar(10)  DEFAULT NULL,  -- numérique (1, 2, 3...) ou hors échelle (HEA, HEA2...)
    `indice_brut` int          DEFAULT NULL,
    `indice_maj`  int          DEFAULT NULL,
    `duree`       varchar(50)  DEFAULT NULL,  -- durée de l'échelon en MOIS, NULL = dernier échelon
    `code`        varchar(30)  NOT NULL       -- clé primaire, identifiant unique lisible
);
```

| Colonne       | Description                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------------- |
| `Corps`       | Corps de la fonction publique hospitalière                                                          |
| `Grade`       | Grade au sein du corps (classe normale, supérieure, exceptionnelle, hors classe...)                 |
| `Categorie`   | Catégorie statutaire : `A`, `A+`, `B`, `C`                                                          |
| `Filiere`     | Filière hospitalière (Administrative, Soignante, Technique, Socio-éducative, Sages-femmes, Enseignement, Ouvriers hospitaliers, Services de soins/rééducation/médico-technique...) |
| `CAT`         | `PNM` (personnel non médical) ou `PM` (personnel médical)                                           |
| `echelon`     | Numéro d'échelon (numérique), ou libellé d'échelon hors échelle (`HEA`, `HEA2`, `HEB3`...)          |
| `indice_brut` | Indice brut (IB). `NULL` pour les échelons hors échelle (pas d'IB) et pour certains corps médicaux (PU-PH, MCU-PH) dont seul l'indice majoré est disponible |
| `indice_maj`  | Indice majoré (IM), sert de base au calcul du traitement : `brut = IM × valeur du point d'indice`   |
| `duree`       | Durée de l'échelon en **mois** (`NULL` si dernier échelon du grade, ou si passage "au choix" pour un échelon hors échelle) |
| `code`        | Clé primaire, identifiant unique lisible (voir convention ci-dessous)                               |

### Table complémentaire `OUTILS_1_CORPS`

`CORPS.sql` crée et alimente une table séparée, `OUTILS_1_CORPS` (`Corps`, `Grade`, `code`) : une nomenclature de référence bien plus large que ce qui est actuellement chiffré dans `OUTILS_1_GRILLE`. Elle sert de **feuille de route** : chaque ligne y est un corps/grade qui existe dans la FPH mais n'a pas forcément (encore) sa grille indiciaire complète dans `grilles.sql`. Utile pour choisir un code cohérent avant de contribuer une nouvelle grille.

### Convention de code

Format général : `{CORPS}_{GRADE_ABREGE}_{ECHELON}`

Abréviations de grade déjà utilisées dans `grilles.sql` (à adapter selon le corps) :

- `CN` = classe normale
- `CS` = classe supérieure
- `CE` = classe exceptionnelle
- `HC` = hors classe
- `PR` = principal
- `P0` / `P1` / `P2` = grade de base / principal 1re classe / principal 2e classe (corps de catégorie C notamment)
- `G1` / `G2` = 1er grade / 2ème grade (ex : sages-femmes)
- `CP` = classe provisoire (en voie d'extinction)
- `0` = grade unique quand le corps n'a qu'un seul grade (ex : `PUPH_0_1`, `MCUPH_0_1`, `CM_0_1`)

Pour les échelons hors échelle, l'échelon lui-même porte le libellé (`HEA`, `HEA2`, `HEA3`, `HEB`, `HEB2`, `HEB3`, `HEBbis`, `HEC`...) au lieu d'un numéro — voir « Points de vigilance » plus bas.

Exemples :

- `AAH_CN_1` → Attaché d'administration hospitalière, classe normale, échelon 1
- `ACH_CS_5` → Adjoint des cadres hospitaliers, classe supérieure, échelon 5
- `DH_HC_HEA2` → Directeur d'hôpital, hors classe, échelon hors échelle HEA2
- `SFH_G2_3` → Sage-femme des hôpitaux, 2ème grade, échelon 3
- `PUPH_0_1` → Professeur des universités-praticien hospitalier (grade unique), échelon 1

Pour un nouveau corps, choisissez un préfixe court et unique (2 à 5 lettres, cohérent avec l'usage RH courant si possible — s'inspirer de `CORPS.sql` quand une abréviation existe déjà) et documentez-le dans ce tableau.

## Corps déjà renseignés (grilles indiciaires complètes)

| Préfixe code | Corps                                                          | Filière                                                      | CAT | Grades disponibles  |
| ------------ | ----------------------------------------------------------------- | --------------------------------------------------------- | --- | ------------------- |
| AAH          | Attaché d'administration hospitalière                             | Administrative                                            | PNM | Classe normale, hors-classe, principal |
| AA           | Adjoint administratif hospitalier                                 | Administrative                                            | PNM | Grade de base, principal 1re/2e classe |
| ACH          | Adjoint des cadres hospitaliers                                   | Administrative                                            | PNM | Classe normale, supérieure, exceptionnelle |
| AMA          | Assistant médico-administratif                                    | Administrative                                            | PNM | Classe normale, supérieure, exceptionnelle |
| AMB          | Ambulancier de la fonction publique hospitalière                  | Services de soins, de rééducation et médico-technique     | PNM | Grade de base, principal |
| ASE          | Assistant territorial socio-éducatif                              | Socio-éducative                                           | PNM | Grade de base, classe exceptionnelle |
| ASHQ         | Agent de service hospitalier qualifié                             | Services de soins, de rééducation et médico-technique     | PNM | Classe normale, supérieure |
| ASH          | Aide-soignant hospitalier                                         | Soignante                                                 | PNM | Classe normale, classe supérieure |
| CESF         | Conseiller en économie sociale et familiale                       | Socio-éducative                                           | PNM | 1er grade classe supérieure, 2nd grade |
| CM           | Coordonnateur en maïeutique (emploi fonctionnel)                  | Sages-femmes                                              | PM  | Grade unique (+ échelons hors échelle HEA-HEA3) |
| DESSMS       | Directeur d'établissement sanitaire, social et médico-social      | Administrative                                            | PNM | Classe normale, hors classe (+ hors échelle) |
| DH           | Directeur d'hôpital                                               | Administrative                                            | PNM | Classe provisoire, classe normale, hors classe, classe exceptionnelle (+ hors échelle) |
| IDE          | Infirmier en soins généraux                                       | Services de soins, de rééducation et médico-technique     | PNM | Grade 1 ISGS, grade 2 ISGS |
| IHC          | Ingénieur en chef hospitalier                                     | Technique                                                 | PNM | Grade de base, hors classe, classe exceptionnelle (+ hors échelle) |
| IH           | Ingénieur hospitalier                                             | Technique                                                 | PNM | Grade de base, principal, hors classe (+ hors échelle) |
| MCUPH        | Maître de conférences des universités-praticien hospitalier       | Enseignement                                              | PM  | Grade unique |
| MO           | Maitrise ouvrière                                                 | Ouvriers hospitaliers                                     | PNM | Agent de maîtrise, agent de maîtrise principal |
| PO           | Personnels ouvrier                                                | Ouvriers hospitaliers                                     | PNM | Agent d'entretien qualifié, ouvrier principal 1re/2e classe |
| PUPH         | Professeur des universités-praticien hospitalier                  | Enseignement                                              | PM  | Grade unique |
| SFH          | Sage-femme des hôpitaux                                           | Sages-femmes                                              | PM  | 1er grade, 2ème grade |

## Corps à compléter (contributions bienvenues)

`CORPS.sql` référence de nombreux autres corps/grades sans grille chiffrée dans `grilles.sql`, notamment :

- Filière soignante : infirmiers spécialisés (IADE, IBODE), puéricultrices, cadres de santé, cadres supérieurs de santé, auxiliaires de puériculture, auxiliaires médicaux en pratique avancée
- Filière médico-technique : manipulateurs d'électroradiologie, techniciens de laboratoire médical, préparateurs en pharmacie hospitalière
- Filière rééducation : masseurs-kinésithérapeutes, ergothérapeutes, orthophonistes, orthoptistes, psychomotriciens, pédicures-podologues
- Filière socio-éducative : éducateurs spécialisés, éducateurs de jeunes enfants, assistants de service social, cadres socio-éducatifs
- Filière technique : techniciens hospitaliers, techniciens supérieurs hospitaliers, dessinateurs
- Praticiens hospitaliers (PH temps plein/partiel), praticiens hospitaliers universitaires (PHU)
- Personnels de recherche clinique (ARC, coordinateurs/chefs de projet recherche clinique, biostatisticiens)
- Psychologues, physiciens médicaux, directeurs des soins, directeurs généraux/adjoints
- ... et tout autre corps manquant

## Comment contribuer

1. Récupérez la grille indiciaire du corps souhaité depuis une source fiable (Légifrance, arrêté publié au JO, ou un site RH vérifié).
2. Vérifiez dans `CORPS.sql` si un code de référence existe déjà pour ce corps/grade, pour rester cohérent.
3. Ajoutez les lignes correspondantes dans `grilles.sql` (ou un fichier `insert_<corps>.sql` séparé), en respectant le schéma de la table et la convention de code.
4. Convertissez les durées d'échelon en **mois** (pas de texte libre type "2 ans 6 mois").
5. Pour un échelon hors échelle (HEA, HEB...), laissez `indice_brut` à `NULL` et utilisez le libellé de l'échelon (`HEA`, `HEA2`...) dans la colonne `echelon`.
6. Indiquez la source et la date de vérification dans un commentaire SQL en tête de votre ajout.
7. Ouvrez une pull request.

### Exemple de format attendu pour une contribution

```sql
-- Source : [nom de la source], vérifié le JJ/MM/AAAA
INSERT INTO `OUTILS_1_GRILLE` (`Corps`, `Grade`, `Categorie`, `Filiere`, `CAT`, `echelon`, `indice_brut`, `indice_maj`, `duree`, `code`) VALUES
('Nom du corps', 'Classe normale', 'A', 'Filière hospitalière ...', 'PNM', '1', 000, 000, '12', 'XXX_CN_1');
```

## Points de vigilance connus

- **Directeur d'hôpital (DH), classe normale, échelon 1** : durée de 6 mois (au lieu de 12 pour les échelons suivants) — à confirmer par rapport au décret n°2005-921 si besoin d'un usage officiel.
- **Échelons hors échelle (`HEA`, `HEA2`, `HEB3`...)** : la source d'origine utilisait la colonne "Indice Brut" pour indiquer le libellé de l'échelon plutôt qu'un indice brut réel — `indice_brut` est donc systématiquement `NULL` pour ces lignes, seul l'indice majoré fait foi. Le passage d'un échelon hors échelle à l'autre se fait "au choix" ou "selon les fonctions exercées", sans durée statutaire garantie ; les durées renseignées (`12` mois, etc.) sont indicatives.
- **PU-PH et MCU-PH** (`PUPH_0_*`, `MCUPH_0_*`) : statut bi-appartenant (rémunération partagée université/hôpital), sans grille officielle facilement accessible en ligne au moment de la saisie. Les indices majorés ont été **reconstitués par calcul inverse** (`IM = salaire brut ÷ valeur du point d'indice`) à partir de salaires bruts publiés, et non lus directement sur un décret — écart de rounding possible (~1 à 3 €/mois). `indice_brut` est laissé à `NULL` faute de donnée fiable. **À vérifier en priorité avant tout usage officiel.**
- Les échelons `Eleve dire` (DESSMS, DH) et `Echelon fo` (DH, classe provisoire) sont des libellés tronqués repris tels quels de la source d'origine (probablement "Élève directeur" et "Échelon fonctionnel") — à clarifier si une contribution ultérieure a la source complète.
- Les grilles évoluent régulièrement (revalorisations, points d'indice, réformes catégorielles) : la date de dernière vérification doit toujours être documentée par contribution.

## Licence

À définir selon votre choix (MIT, CC0, ou autre licence ouverte recommandée pour un jeu de données destiné à un usage communautaire).
