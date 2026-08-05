# Grilles indiciaires — Fonction Publique Hospitalière (FPH)

Base de données SQL open-source des grilles indiciaires de la Fonction Publique Hospitalière française : corps, grades, échelons, indices bruts/majorés et durées d'échelon.

## Pourquoi ce projet

Les grilles indiciaires de la FPH sont publiées corps par corps (arrêtés, décrets, tableaux PDF), sans jeu de données structuré et réutilisable centralisé. Ce projet vise à combler ce manque en proposant une base SQL simple, versionnée et complétable par la communauté.

⚠️ **Avertissement** : ces données sont saisies manuellement à partir de sources publiques et n'ont pas de valeur officielle. Elles peuvent contenir des erreurs ou ne pas refléter la version la plus récente d'un texte réglementaire. **Toujours vérifier auprès d'une source officielle (Légifrance, décret concerné) avant tout usage RH, paie ou juridique.**

## Structure de la table

```sql
CREATE TABLE OUTILS_1_GRILLE (
    code        VARCHAR(30),   -- identifiant unique lisible, ex: AAH_CN_1
    Corps       VARCHAR(100),  -- ex: Attaché d'administration hospitalière (AAH)
    Grade       VARCHAR(100),  -- ex: Classe normale
    echelon     VARCHAR(10),
    indice_brut INT,
    indice_maj  INT,
    duree       INT            -- durée de l'échelon en MOIS, NULL = dernier échelon (pas de durée)
);
```

| Colonne       | Type          | Description                                                            |
|---------------|---------------|--------------------------------------------------------------------------|
| `code`        | VARCHAR(30)   | Clé unique lisible (voir convention ci-dessous)                         |
| `Corps`       | VARCHAR(100)  | Corps de la fonction publique hospitalière                              |
| `Grade`       | VARCHAR(100)  | Grade au sein du corps (classe normale, supérieure, exceptionnelle...)  |
| `echelon`     | VARCHAR(10)   | Numéro d'échelon                                                        |
| `indice_brut` | INT           | Indice brut (IB)                                                        |
| `indice_maj`  | INT           | Indice majoré (IM), sert de base au calcul du traitement                |
| `duree`       | INT           | Durée de l'échelon en **mois** (`NULL` si dernier échelon du grade)     |

### Convention de code

Format : `{CORPS}_{GRADE_ABREGE}_{ECHELON}`

Abréviations de grade utilisées :
- `CN` = classe normale (ou grade unique quand il n'y a qu'un seul grade)
- `CS` = classe supérieure
- `CE` = classe exceptionnelle

Exemples :
- `AAH_CN_1` → Attaché d'administration hospitalière, classe normale, échelon 1
- `ACH_CS_5` → Adjoint des cadres hospitaliers, classe supérieure, échelon 5
- `DH_CN_10` → Directeur d'hôpital, classe normale, échelon 10

Pour un nouveau corps, choisissez un préfixe court et unique (2 à 4 lettres, cohérent avec l'usage RH courant si possible) et documentez-le dans ce tableau.

## Fichiers du projet

| Fichier                      | Rôle                                                                  |
|------------------------------|-----------------------------------------------------------------------|
| `grille.sql`                 | Grilles et création de la table `OUTILS_1_GRILLE`                     |


> Si vous partez d'une base neuve, exécutez les scripts dans l'ordre ci-dessus.

## Installation rapide

```bash
mysql -u user -p ma_base < create_table.sql
mysql -u user -p ma_base < insert_grilles_fph.sql
mysql -u user -p ma_base < ajout_colonne_code.sql
mysql -u user -p ma_base < index_code.sql
```

(Adapter la commande selon votre SGBD : MySQL, PostgreSQL, SQLite...)

## Corps déjà renseignés

| Code corps | Corps                                              | Grades disponibles                                  |
|------------|-----------------------------------------------------|------------------------------------------------------|
| AAH        | Attaché d'administration hospitalière               | Classe normale                                        |
| ACH        | Adjoint des cadres hospitaliers                      | Classe normale, classe supérieure, classe exceptionnelle |
| AMA        | Assistant médico-administratif                       | Classe normale, classe supérieure, classe exceptionnelle |
| DH         | Directeur d'hôpital                                  | Classe normale                                         |

## Corps à compléter (contributions bienvenues)

- Adjoint administratif hospitalier (catégorie C)
- Secrétaire médicale (ancien corps)
- Filière soignante (infirmiers, aides-soignants, cadres de santé...)
- Filière technique
- Filière socio-éducative
- Filière médico-technique
- Praticiens hospitaliers
- ... et tout autre corps manquant

## Comment contribuer

1. Récupérez la grille indiciaire du corps souhaité depuis une source fiable (Légifrance, arrêté publié au JO, ou un site RH vérifié).
2. Ajoutez les lignes correspondantes dans un fichier `insert_<corps>.sql`, en respectant le schéma de la table et la convention de code.
3. Convertissez les durées d'échelon en **mois** (pas de texte libre type "2 ans 6 mois").
4. Indiquez la source et la date de vérification dans un commentaire SQL en tête du fichier.
5. Ouvrez une pull request.

### Exemple de format attendu pour une contribution

```sql
-- Source : [nom de la source], vérifié le JJ/MM/AAAA
INSERT INTO OUTILS_1_GRILLE (code, Corps, Grade, echelon, indice_brut, indice_maj, duree) VALUES
('XXX_CN_1', 'Nom du corps', 'Classe normale', 1, 000, 000, 12);
```

## Points de vigilance connus

- **Directeur d'hôpital (DH), échelon 9** : la durée d'échelon était illisible sur la source d'origine et a été estimée par cohérence de progression. À vérifier avant usage officiel (décret n°2005-921).
- Les grilles évoluent régulièrement (revalorisations, points d'indice, réformes catégorielles) : la date de dernière vérification doit toujours être documentée par contribution.

## Licence

À définir selon votre choix (MIT, CC0, ou autre licence ouverte recommandée pour un jeu de données destiné à un usage communautaire).
