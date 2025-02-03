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
