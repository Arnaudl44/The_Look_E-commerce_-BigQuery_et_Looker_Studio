# 📝 Analyse des performances de ventes – The Look E-commerce  

## 📌 Introduction  

**The Look E-commerce** est une base de données fournie par Google pour démontrer les capacités analytiques de **BigQuery**. Elle contient des informations détaillées sur les **commandes, produits, clients, stocks et interactions des utilisateurs** sur un site e-commerce.  

### 🗂️ Les tables utilisées :  

- **`orders`** : Détail des commandes passées par les clients, incluant le montant total et le statut de la commande.  
- **`order_items`** : Produits achetés dans chaque commande avec les prix et quantités.  
- **`products`** : Informations produits (catégorie, marque, prix catalogue, etc.).  
- **`users`** : Informations sur les clients (âge, localisation, historique d’achats).  
- **`inventory_items`** : Gestion des stocks des produits dans les centres de distribution.  
- **`distribution_centers`** : Localisation des entrepôts de stockage.  
- **`events`** : Enregistrement des interactions des visiteurs avec le site web (ajout au panier, consultation de pages, achats…).  

L’objectif de ce projet est d’explorer ces données pour en extraire des **insights exploitables**, en particulier sur la **performance des ventes et la rentabilité des produits**.  

---

## 🏗️ Création de la base dans BigQuery  

J’ai commencé par **structurer la base de données dans BigQuery** en important les différentes tables. Cette étape a permis de valider l'intégrité des données et de préparer le terrain pour l'analyse.  

- **Importation des fichiers sources** en tant que tables BigQuery.  
- **Vérification des relations entre les tables** (ex. `orders` et `order_items` via `order_id`).  
- **Contrôle de la granularité** des données pour s’assurer de leur cohérence (ex. `order_items` au niveau des produits, `orders` au niveau des transactions).  

---

## 🔍 Analyse exploratoire des données (EDA)  

Une fois les données disponibles dans BigQuery, j’ai effectué une **Exploratory Data Analysis (EDA)** afin de mieux comprendre leur structure et d’identifier les variables pertinentes.  

### 📌 Étapes réalisées lors de l’EDA  

1. **Exploration des volumes de données** : Nombre de lignes et vérification des valeurs nulles dans chaque table.  
2. **Vérification des valeurs manquantes et incohérences** : Examen des statuts de commande (`Completed`, `Cancelled`, `Shipped`…), identification des éventuelles anomalies.  
3. **Analyse de la distribution des données** : Répartition des commandes dans le temps, fréquence d’achat des clients, catégories de produits les plus populaires.  
4. **Exploration de la table `events`** : Étude du comportement des utilisateurs sur le site, avec analyse des interactions (`cart`, `purchase`, `product`, `home`…), et vérification de l’impact des `user_id` null sur la navigation.  

⚡ **Les requêtes SQL utilisées pour l’EDA sont disponibles dans un dossier dédié.**  

---

## 🔗 Jointures et préparation des données  

Suite à l’EDA, j’ai réalisé plusieurs jointures de tables afin de croiser les informations pertinentes et obtenir une vue plus complète des performances e-commerce.  

### 🔄 Jointure entre `order_items` et `products`

```sql
SELECT
  oi.*, -- Toutes les colonnes de la table order_items
  p.cost, -- Coût du produit
  p.category, -- Catégorie du produit
  p.name, -- Nom du produit
  p.brand, -- Marque du produit
  p.department, -- Département du produit
  p.distribution_center_id -- Identifiant du centre de distribution
FROM 
  `thelook_ecommerce.order_items` oi
LEFT JOIN 
  `thelook_ecommerce.products` p
ON 
  oi.product_id = p.id;
```

### 🔄 Jointure entre `order_items_x_products` et `users`

```sql
SELECT
  oip.*, -- Toutes les colonnes de la table order_items_x_products
  u.first_name, -- Prénom de l'utilisateur
  u.last_name, -- Nom de l'utilisateur
  u.age, -- Âge de l'utilisateur
  u.gender, -- Genre de l'utilisateur
  u.country, -- Pays de l'utilisateur
  u.latitude, -- Latitude de localisation
  u.longitude, -- Longitude de localisation
  u.created_at AS user_created_at -- Date de création de l'utilisateur
FROM 
  `thelook_ecommerce.order_items_x_products` oip
LEFT JOIN 
  `thelook_ecommerce.users` u 
ON 
  oip.user_id = u.id;
```

---

## 🎯 Analyse des performances des ventes  

Après avoir exploré les données, j’ai décidé de concentrer l’analyse sur la **performance des ventes**.  

### 🏆 Principaux KPI's  

```sql
SELECT 
  SUM(sale_price) AS total_revenue,
  COUNT(DISTINCT order_id) AS total_orders,
  COUNT(id) AS total_quantity,
  CAST(COUNT(id) AS FLOAT64) / COUNT(DISTINCT order_id) AS avg_products_per_order,
  COUNT(DISTINCT user_id) AS total_customers,
  SUM(cost) AS total_cost,
  SUM(sale_price) - SUM(cost) AS profit_margin,
  SUM(sale_price) / COUNT(DISTINCT order_id) AS avg_order_value,
  COUNT(DISTINCT CASE WHEN status = 'Cancelled' THEN order_id ELSE NULL END) AS total_orders_cancelled,
  COUNT(returned_at) AS total_products_returned
FROM 
  `thelook_ecommerce.order_items_x_products`;

📌 **Visualisation des KPI's** :

![KPI Dashboard](https://raw.githubusercontent.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/main/images/KPI.png)

---

### 📅 Principaux KPI's par année et mois  

```sql
SELECT 
  EXTRACT(YEAR FROM created_at) AS yr, -- Année
  EXTRACT(MONTH FROM created_at) AS mo, -- Mois
  SUM(sale_price) AS total_revenue, -- Chiffre d'affaires total
  COUNT(DISTINCT order_id) AS total_orders, -- Nombre total de commandes
  COUNT(id) AS total_quantity, -- Nombre total de produits vendus
  CAST(COUNT(id) AS FLOAT64) / COUNT(DISTINCT order_id) AS avg_products_per_order, -- Nombre moyen de produits par commande
  COUNT(DISTINCT user_id) AS total_customers, -- Nombre total de clients actifs
  SUM(cost) AS total_cost, -- Coût total
  SUM(sale_price) - SUM(cost) AS profit_margin, -- Marge totale
  SUM(sale_price) / COUNT(DISTINCT order_id) AS avg_order_value, -- Valeur moyenne par commande (AOV)
  COUNT(DISTINCT CASE WHEN status = 'Cancelled' THEN order_id ELSE NULL END) AS total_orders_cancelled, -- Nombre total de commandes annulées
  COUNT(returned_at) AS total_products_returned -- Nombre total de produits retournés
FROM 
  `thelook_ecommerce.order_items_x_products`
GROUP BY 
  1, 2
ORDER BY 
  1, 2;

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165623_Ventes_globales.png)


### 📊 Répartition des utilisateurs selon le nombre de commandes passées  

```sql
WITH total_orders_cte AS (
  SELECT
    user_id, -- Identifiant de l'utilisateur
    COUNT(DISTINCT order_id) AS total_orders -- Nombre total de commandes par utilisateur
  FROM 
    `innate-path-415009.thelook_ecommerce.orders`
  GROUP BY 
    user_id
)

SELECT 
  total_orders, -- Nombre de commandes passées
  COUNT(user_id) AS total_users -- Nombre d'utilisateurs ayant passé ce nombre de commandes
FROM 
  total_orders_cte
GROUP BY 
  total_orders
ORDER BY 
  total_users DESC; -- Trier par nombre d'utilisateurs en ordre décroissant

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165853_Commandes_par_client.png)

### 📊 Répartition des commandes par nombre de produits  

```sql
SELECT
  num_of_item, -- Nombre de produits dans une commande
  COUNT(order_id) AS number_of_orders -- Nombre de commandes correspondantes
FROM 
  `innate-path-415009.thelook_ecommerce.orders`
GROUP BY 
  num_of_item
ORDER BY 
  number_of_orders DESC; -- Trier par nombre de commandes en ordre décroissant

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165806_Nombre_produits_par%20commande.png)

### 📊 Saisonnalité des ventes  

```sql
SELECT
  EXTRACT(MONTH FROM created_at) AS mo, -- Mois
  SUM(sale_price) AS total_revenue, -- Chiffre d'affaires total
  COUNT(DISTINCT order_id) AS total_orders, -- Nombre total de commandes
  COUNT(id) AS total_quantity, -- Nombre total de produits vendus
  CAST(COUNT(id) AS FLOAT64) / COUNT(DISTINCT order_id) AS avg_products_per_order, -- Nombre moyen de produits par commande
  COUNT(DISTINCT user_id) AS total_customers, -- Nombre total de clients actifs
  SUM(cost) AS total_cost, -- Coût total
  SUM(sale_price) - SUM(cost) AS profit_margin, -- Marge totale
  SUM(sale_price) / COUNT(DISTINCT order_id) AS avg_order_value, -- Valeur moyenne par commande (AOV)
  COUNT(DISTINCT CASE WHEN status = 'Cancelled' THEN order_id ELSE NULL END) AS total_orders_cancelled, -- Nombre total de commandes annulées
  COUNT(returned_at) AS total_products_returned -- Nombre total de produits retournés
FROM 
  `thelook_ecommerce.order_items_x_products`
GROUP BY 
  mo
ORDER BY 
  mo; -- Trier par mois

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165527_Saisonnalit%C3%A9.png)

### 📊 Produits les plus commandés et retournés par catégorie  

```sql
SELECT
  category, -- Catégorie de produit
  COUNT(order_id) AS total_orders, -- Nombre total de commandes
  SUM(CASE WHEN returned_at IS NOT NULL THEN 1 ELSE 0 END) AS total_returns -- Nombre total de produits retournés
FROM 
  `thelook_ecommerce.order_items_x_products_x_users`
GROUP BY 
  category
ORDER BY 
  total_returns DESC; -- Trier par nombre de retours décroissant

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165729_Retours.png)

### 🏅 Top 10 clients  

```sql
WITH customer_summary AS (
  SELECT
    user_id, -- Identifiant de l'utilisateur
    first_name, -- Prénom
    last_name, -- Nom de famille
    age, -- Âge
    country, -- Pays
    COUNT(DISTINCT order_id) AS total_orders, -- Nombre total de commandes
    SUM(sale_price) AS total_spent, -- Total des dépenses
    AVG(sale_price) AS avg_order_value -- Valeur moyenne des commandes
  FROM 
    `thelook_ecommerce.order_items_x_products_x_users`
  GROUP BY 
    user_id, first_name, last_name, age, country
)
SELECT 
  user_id, -- Identifiant de l'utilisateur
  CONCAT(first_name, ' ', last_name) AS full_name, -- Nom complet
  age, -- Âge
  country, -- Pays
  total_orders, -- Nombre total de commandes
  total_spent, -- Total des dépenses
  avg_order_value -- Valeur moyenne des commandes
FROM 
  customer_summary
ORDER BY 
  total_spent DESC -- Trier par dépenses totales décroissant
LIMIT 10; -- Top 10 clients

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165413_Top_10_clients.png)

### 📊 Segmentation par âge et chiffre d'affaires total  

```sql
SELECT 
  CASE 
    WHEN age < 18 THEN 'Under 18'
    WHEN age BETWEEN 18 AND 24 THEN '18-24'
    WHEN age BETWEEN 25 AND 34 THEN '25-34'
    WHEN age BETWEEN 35 AND 44 THEN '35-44'
    WHEN age BETWEEN 45 AND 54 THEN '45-54'
    WHEN age >= 55 THEN '55+'
    ELSE 'Unknown'
  END AS age_group, -- Catégorie d'âge
  COUNT(DISTINCT user_id) AS total_customers, -- Nombre total de clients
  SUM(sale_price) AS total_revenue, -- Chiffre d'affaires total
  AVG(sale_price) AS avg_order_value -- Valeur moyenne des commandes
FROM 
  `thelook_ecommerce.order_items_x_products_x_users`
GROUP BY 
  age_group
ORDER BY 
  total_revenue DESC; -- Trier par chiffre d'affaires décroissant

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20165956_Clients_%C3%A2ge.png)

### 📊 Segmentation par comportement d'achat (petits vs gros clients)  

```sql
WITH customer_spending AS (
  SELECT 
    user_id, -- Identifiant de l'utilisateur
    SUM(sale_price) AS total_spent -- Total des dépenses
  FROM 
    `thelook_ecommerce.order_items_x_products_x_users`
  GROUP BY 
    user_id
)
SELECT 
  CASE 
    WHEN total_spent < 100 THEN 'Low Spenders (< 100)'
    WHEN total_spent BETWEEN 100 AND 500 THEN 'Medium Spenders (100-500)'
    ELSE 'High Spenders (> 500)'
  END AS spending_category, -- Catégorie de dépense
  COUNT(user_id) AS total_customers, -- Nombre total de clients
  SUM(total_spent) AS total_revenue, -- Chiffre d'affaires total
  AVG(total_spent) AS avg_spent_per_customer -- Dépense moyenne par client
FROM 
  customer_spending
GROUP BY 
  spending_category
ORDER BY 
  total_revenue DESC; -- Trier par chiffre d'affaires décroissant

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20170135_Cat%C3%A9gorie_d%C3%A9penses.png)

### 🔗 Nombre de commandes par source de trafic  

```sql
WITH CTE AS (
  SELECT
    u.id AS user_id, -- Identifiant de l'utilisateur
    u.traffic_source AS traffic_source, -- Source de trafic
    COUNT(o.order_id) AS number_of_orders -- Nombre de commandes passées
  FROM 
    `innate-path-415009.thelook_ecommerce.users` u
  LEFT JOIN 
    `innate-path-415009.thelook_ecommerce.orders` o
  ON 
    u.id = o.user_id 
  GROUP BY 
    u.id, u.traffic_source
  HAVING 
    COUNT(o.order_id) >= 1 -- Inclure uniquement les utilisateurs ayant passé au moins une commande
)
SELECT 
  traffic_source, -- Source de trafic
  SUM(number_of_orders) AS total_orders_by_source -- Total des commandes par source
FROM 
  CTE
GROUP BY 
  traffic_source
ORDER BY 
  total_orders_by_source DESC; -- Trier par total des commandes décroissant

![Evolution des ventes](https://github.com/Arnaudl44/The_Look_E-commerce_-BigQuery_et_Looker_Studio/blob/main/images/Capture%20d%E2%80%99%C3%A9cran%202025-02-03%20170038_Trafic_source.png)
