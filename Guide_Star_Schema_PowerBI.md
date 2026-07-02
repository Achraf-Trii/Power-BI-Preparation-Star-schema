# Guide détaillé — Construction du Star Schema RetailData dans Power BI

Ce guide explique, étape par étape, comment construire le modèle en étoile complet (fact_sales + dimensions) à partir du fichier source unique **RetailData.xlsx**, en utilisant Power Query dans Power BI Desktop.

---

## 0. Import de la source

```
Home → Get Data → Text/CSV (ou Excel si tu pars de RetailData.xlsx)
↓
Sélectionner RetailData
↓
Transform Data
```

⚠️ Ne clique pas sur **Load** directement — on veut d'abord tout préparer dans Power Query.

Cette requête initiale s'appelle **retailData**. Toutes les tables du modèle (dimensions + fait) seront créées à partir d'elle via **Reference**, jamais via **Duplicate**.

> **Pourquoi Reference et pas Duplicate ?**
> Une référence reste connectée à la requête d'origine. Si `retailData` est modifiée ou rafraîchie, toutes les tables qui en dépendent (dim_dates, dim_item_produit, fact_sales...) se mettent à jour automatiquement. Un Duplicate, lui, copie les données de façon indépendante et casse ce lien.

---

## 1. DIM_DATES

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_dates
```

**Étape 2 — Garder uniquement Order_Date**
```
Sélectionner toutes les colonnes sauf Order_Date
↓
Home → Remove Columns
```

**Étape 3 — Supprimer les doublons**
```
Sélectionner Order_Date
↓
Home → Remove Rows → Remove Duplicates
```

**Étape 4 — Ajouter Date_Key**
```
Add Column → Custom Column
Nom : Date_Key
Formule :
Date.Year([Order_Date])*10000
+ Date.Month([Order_Date])*100
+ Date.Day([Order_Date])
```

**Étape 5 — Ajouter Day**
```
Add Column → Date → Day → Name of Day
```
Résultat : `Tuesday`, `Monday`, `Saturday`...

**Étape 6 — Ajouter Month**
```
Add Column → Date → Month → Name of Month
```
Résultat : `December`, `February`, `July`...

**Étape 7 — Ajouter Quarter**
```
Add Column → Date → Quarter → Quarter of Year
```
Résultat : `4`, `1`, `3`...
(Tu peux transformer en texte `Q1`, `Q2`... si tu veux, ce n'est pas obligatoire.)

**Étape 8 — Ajouter Year**
```
Add Column → Date → Year → Year
```

**Étape 9 — Réorganiser les colonnes**

| Date_Key | Order_Date | Day | Month | Quarter | Year |
|---|---|---|---|---|---|
| 20131231 | 2013-12-31 | Tuesday | December | 4 | 2013 |
| 20130225 | 2013-02-25 | Monday | February | 1 | 2013 |
| 20130727 | 2013-07-27 | Saturday | July | 3 | 2013 |

`dim_dates` est prête.

---

## 2. DIM_ITEM_PRODUIT

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_item_produit
```

**Étape 2 — Garder les colonnes produit**
```
Sélectionner : item_id, Category, Sub_Category, Sub_Sub_Category, Brand
↓
Home → Remove Other Columns (ou Choose Columns)
```

**Étape 3 — Supprimer les doublons**
```
Sélectionner item_id
↓
Home → Remove Rows → Remove Duplicates
```

Résultat :

| item_id | Category | Sub_Category | Sub_Sub_Category | Brand |
|---|---|---|---|---|
| 860 | Mobile Phones | Smartphones | Android Phones | iBall |
| 1620 | Footwear | Women Footwear | Sports Sandals | Adidas |

`dim_item_produit` est prête.

---

## 3. DIM_LOCALISATION

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_localisation
```

**Étape 2 — Garder les colonnes localisation**
```
Sélectionner : loc_id, City, State
↓
Home → Choose Columns → Choose Columns
```
⚠️ Ne pas inclure **Region** : l'analyse de qualité de données a montré qu'un même `loc_id` (ex. Riverside, 104) est associé à plusieurs régions différentes (North, South, East, West). Region n'est donc pas un attribut fiable de la localisation — elle est traitée séparément dans `dim_region`.

**Étape 3 — Supprimer les doublons**
```
Sélectionner loc_id
↓
Home → Remove Rows → Remove Duplicates
```

Résultat :

| loc_id | City | State |
|---|---|---|
| 29 | San Francisco | California |
| 40 | San Jose | California |

`dim_localisation` est prête.

---

## 4. DIM_CUSTOMERS

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_customers
```

**Étape 2 — Garder Customer_id**
```
Sélectionner : Customer_id
↓
Home → Choose Columns → Choose Columns
```

**Étape 3 — Supprimer les doublons**
```
Sélectionner Customer_id
↓
Home → Remove Rows → Remove Duplicates
```

`dim_customers` est prête (une ligne par client).

---

## 5. DIM_PAYMENT

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_payment
```

**Étape 2 — Garder Payment_Method**
```
Sélectionner : Payment_Method
↓
Home → Choose Columns → Choose Columns
```

**Étape 3 — Supprimer les doublons**
```
Sélectionner Payment_Method
↓
Home → Remove Rows → Remove Duplicates
```

**Étape 4 — Ajouter payment_id (clé surrogate)**
```
Add Column → Index Column → From 1
Renommer la colonne Index en : payment_id
```

**Étape 5 — Réorganiser les colonnes**

| payment_id | Payment_Method |
|---|---|
| 1 | Credit Card EMI |
| 2 | Cash on Delivery |
| 3 | Debit Card |
| 4 | Credit Card |
| 5 | Net Banking |
| 6 | Gift Coupons |

`dim_payment` est prête.

---

## 6. DIM_ORDER_TYPE

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_order_type
```

**Étape 2 — Garder Order_Type**
```
Sélectionner : Order_Type
↓
Home → Choose Columns → Choose Columns
```

**Étape 3 — Supprimer les doublons**
```
Sélectionner Order_Type
↓
Home → Remove Rows → Remove Duplicates
```

**Étape 4 — Ajouter order_type_id (clé surrogate)**
```
Add Column → Index Column → From 1
Renommer la colonne Index en : order_type_id
```

Résultat :

| order_type_id | Order_Type |
|---|---|
| 1 | Contact Center |
| 2 | Online |
| 3 | In-Store |

`dim_order_type` est prête.

---

## 7. DIM_REGION

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : dim_region
```

**Étape 2 — Garder Region**
```
Sélectionner : Region
↓
Home → Choose Columns → Choose Columns
```

**Étape 3 — Supprimer les doublons**
```
Sélectionner Region
↓
Home → Remove Rows → Remove Duplicates
```

**Étape 4 — Ajouter Region_id (clé surrogate)**
```
Add Column → Index Column → From 1
Renommer la colonne Index en : Region_id
```

Résultat :

| Region_id | Region |
|---|---|
| 1 | North |
| 2 | East |
| 3 | South |
| 4 | West |

`dim_region` est prête.

---

## 8. FACT_SALES

**Étape 1 — Créer la référence**
```
Clic droit sur retailData → Reference
Renommer : fact_sales
```

**Étape 2 — Fusionner (Merge) avec chaque dimension pour récupérer les clés**

Pour chaque dimension créée à partir d'un attribut descriptif (Payment_Method, Order_Type, Region), on doit ramener la clé surrogate dans `fact_sales` :

```
Home → Merge Queries
Table 1 : fact_sales, colonne Payment_Method
Table 2 : dim_payment, colonne Payment_Method
Join Kind : Left Outer
↓
Développer (Expand) uniquement payment_id
```

Répéter la même opération :
- `Merge` avec **dim_order_type** sur `Order_Type` → récupérer `order_type_id`
- `Merge` avec **dim_region** sur `Region` → récupérer `Region_id`
- `Merge` avec **dim_dates** sur `Order_Date` → récupérer `Date_Key`

**Étape 3 — Supprimer les colonnes descriptives devenues redondantes**
```
Supprimer : Payment_Method, Order_Type, Region, Order_Date,
            Category, Sub_Category, Sub_Sub_Category, Brand,
            City, State
```
(Ces informations restent disponibles via les tables de dimension liées par les clés.)

**Étape 4 — Ajouter les mesures calculées (colonnes)**
```
Add Column → Custom Column
Montant_Brut = [Unit_Price] * [Qty]
Montant_Net  = [Montant_Brut] * (1 - [Discount_Percent])
Marge        = ([Unit_Price] - [Unit_Cost]) * [Qty]
```

**Étape 5 — Colonnes finales de fact_sales**

| Colonne | Origine |
|---|---|
| Order_Id | retailData |
| Date_Key | FK → dim_dates |
| item_id | FK → dim_item_produit |
| loc_id | FK → dim_localisation |
| Customer_id | FK → dim_customers |
| Region_id | FK → dim_region |
| order_type_id | FK → dim_order_type |
| payment_id | FK → dim_payment |
| Unit_Price | mesure |
| Qty | mesure |
| Discount_Percent | mesure |
| Unit_Cost | mesure |
| Montant_Brut | mesure calculée |
| Montant_Net | mesure calculée |
| Marge | mesure calculée |

`fact_sales` est prête. Clique maintenant sur **Close & Apply**.

---

## 9. Créer les relations dans le modèle (Model View)

Dans **Model view** de Power BI :

```
Model → glisser une clé de dim_XXX vers la clé correspondante de fact_sales
```

Relations à créer (toutes en 1 → *) :

| Dimension | Clé | → | fact_sales |
|---|---|---|---|
| dim_dates | Date_Key | → | Date_Key |
| dim_item_produit | item_id | → | item_id |
| dim_localisation | loc_id | → | loc_id |
| dim_customers | Customer_id | → | Customer_id |
| dim_payment | payment_id | → | payment_id |
| dim_order_type | order_type_id | → | order_type_id |
| dim_region | Region_id | → | Region_id |

Vérifie que chaque relation est bien **Single direction** et **1 (dimension) → * (fact)**.

---

## 10. Ajouter les hiérarchies dans Power BI

Une hiérarchie permet de faire du drill-down/drill-up directement dans les visuels (ex: passer de l'année au trimestre au mois au jour d'un simple clic).

### Hiérarchie Date (dans dim_dates)

```
Dans le volet Fields (Report view) :
Clic droit sur Year (dans dim_dates)
↓
Create Hierarchy
Renommer : Hiérarchie Date
↓
Glisser-déposer Quarter dans la hiérarchie (sous Year)
↓
Glisser-déposer Month dans la hiérarchie (sous Quarter)
↓
Glisser-déposer Day dans la hiérarchie (sous Month)
```

Résultat : `Hiérarchie Date` = Year → Quarter → Month → Day

### Hiérarchie Produit (dans dim_item_produit)

```
Clic droit sur Category
↓
Create Hierarchy
Renommer : Hiérarchie Produit
↓
Ajouter Sub_Category (sous Category)
↓
Ajouter Sub_Sub_Category (sous Sub_Category)
↓
Ajouter Brand (sous Sub_Sub_Category) — optionnel selon le niveau de détail souhaité
```

Résultat : `Hiérarchie Produit` = Category → Sub_Category → Sub_Sub_Category → (Brand)

### Hiérarchie Localisation (dans dim_localisation)

```
Clic droit sur State
↓
Create Hierarchy
Renommer : Hiérarchie Localisation
↓
Ajouter City (sous State)
```

Résultat : `Hiérarchie Localisation` = State → City

### Utilisation

Une fois créées, ces hiérarchies apparaissent comme un seul champ pliable dans le volet Fields. Glisse-les directement dans un visuel (Matrix, Bar Chart, Treemap...) : les flèches de drill-down/drill-up apparaissent automatiquement dans le coin du visuel.

---

## 11. Résultat final — schéma du modèle

```
                dim_payment          dim_item_produit
                     |                      |
                     |                      |
dim_customers ---- fact_sales ---- dim_localisation
                     |     |
                     |     |
              dim_order_type   dim_region
                     |
                  dim_dates
```

Le modèle est maintenant un star schema complet : 1 table de faits centrale, 7 tables de dimensions, relations 1-à-plusieurs, et 3 hiérarchies prêtes pour le drill-down dans les rapports.
