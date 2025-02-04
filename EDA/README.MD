# 📊 EDA The Look E-commerce

---

## 🛒 I - Table `orders`

### 🔢 1) Nombre de lignes dans la table `orders`
```sql
SELECT *
FROM `innate-path-415009.thelook_ecommerce.orders`;
```
**📌 Résultat** : 125 122 lignes

---

### 📦 2) Nombre de commandes et d'utilisateurs distincts
```sql
SELECT
  COUNT(DISTINCT order_id) AS number_of_orders,
  COUNT(DISTINCT user_id) AS number_of_customers,
  SUM(num_of_item) AS number_of_items_sold,
  SUM(num_of_item) / COUNT(DISTINCT order_id) AS average_items_per_order
FROM `innate-path-415009.thelook_ecommerce.orders`;
```
**📌 Résultat** :
- 🏷️ 125 122 commandes
- 👥 80 130 utilisateurs
- 📦 181 652 produits vendus
- 📊 1,45 produits par commande

---

### 👩‍💼 3) Nombre de commandes par genre
```sql
SELECT
  gender,
  COUNT(order_id) AS number_of_orders
FROM `innate-path-415009.thelook_ecommerce.orders`
GROUP BY gender;
```
**📌 Résultat** :
- 👩 Femmes : 62 700 commandes
- 👨 Hommes : 62 422 commandes

---

### 📊 4) Nombre de commandes par statut
```sql
SELECT
  status,
  COUNT(DISTINCT order_id) AS number_of_orders
FROM `innate-path-415009.thelook_ecommerce.orders`
GROUP BY status;
```
**📌 Résultat** :
- 🚚 **Shipped** : 37 490
- ✅ **Complete** : 31 342
- 🔄 **Processing** : 24 971
- ❌ **Cancelled** : 18 788
- 🔙 **Returned** : 12 531

---

### 📈 5) Répartition des utilisateurs selon le nombre de commandes passées
```sql
WITH total_orders AS (
  SELECT
    user_id,
    COUNT(DISTINCT order_id) AS number_of_orders
  FROM `innate-path-415009.thelook_ecommerce.orders`
  GROUP BY user_id
)
SELECT 
  number_of_orders,
  COUNT(user_id)
FROM total_orders
GROUP BY number_of_orders
ORDER BY 2 DESC;
```
**📌 Résultat** :
- 1 commande : 50 149 utilisateurs
- 2 commandes : 19 948
- 3 commandes : 5 055
- 4 commandes : 4 978

---

### 🛍️ 6) Répartition des commandes par nombre de produits
```sql
SELECT
  num_of_item,
  COUNT(order_id) AS number_of_orders
FROM `innate-path-415009.thelook_ecommerce.orders`
GROUP BY num_of_item
ORDER BY 2 DESC;
```
**📌 Résultat** :
- 1 produit : 87 944
- 2 produits : 25 018
- 3 produits : 6 318
- 4 produits : 6 292

---

### 📅 7) Nombre de commandes et d'utilisateurs par année
```sql
SELECT
  EXTRACT(YEAR FROM created_at) AS yr,
  COUNT(DISTINCT order_id) AS number_of_orders,
  COUNT(DISTINCT user_id) AS number_of_users
FROM `innate-path-415009.thelook_ecommerce.orders`
GROUP BY 1
ORDER BY 1;
```
**📌 Résultat** :
- 📈 2 193 commandes en 2019 (année incomplète) à 39 866 en 2024
- 👥 2 050 clients en 2019 et 30 793 en 2024

---

### 📊 8) Nombre de commandes et d'utilisateurs par mois (2019 - 2024)
```sql
SELECT
  EXTRACT(MONTH FROM created_at) AS mo,
  COUNT(DISTINCT order_id) AS number_of_orders,
  COUNT(DISTINCT user_id) AS number_of_users
FROM `innate-path-415009.thelook_ecommerce.orders`
GROUP BY 1
ORDER BY 1;
```
**📌 Observations** :
- 📆 Juin est le mois le plus actif
- 📉 L'activité augmente de mars à juin puis diminue après juillet

---

### 📊 9) Nombre de commandes par statut, année et mois
```sql
SELECT
  EXTRACT(YEAR FROM created_at) AS yr,
  COUNT(order_id) AS total_orders,
  COUNT(CASE WHEN status = 'Processing' THEN order_id ELSE NULL END) AS Processing,
  ROUND(COUNT(CASE WHEN status = 'Processing' THEN order_id ELSE NULL END) / COUNT(order_id) * 100, 2) AS pct_Processing,
  COUNT(CASE WHEN status = 'Complete' THEN order_id ELSE NULL END) AS Complete,
  ROUND(COUNT(CASE WHEN status = 'Complete' THEN order_id ELSE NULL END) / COUNT(order_id) * 100, 2) AS pct_Complete,
  COUNT(CASE WHEN status = 'Shipped' THEN order_id ELSE NULL END) AS Shipped,
  ROUND(COUNT(CASE WHEN status = 'Shipped' THEN order_id ELSE NULL END) / COUNT(order_id) * 100, 2) AS pct_Shipped,
  COUNT(CASE WHEN status = 'Returned' THEN order_id ELSE NULL END) AS Returned,
  ROUND(COUNT(CASE WHEN status = 'Returned' THEN order_id ELSE NULL END) / COUNT(order_id) * 100, 2) AS pct_Returned,
  COUNT(CASE WHEN status = 'Cancelled' THEN order_id ELSE NULL END) AS Cancelled,
  ROUND(COUNT(CASE WHEN status = 'Cancelled' THEN order_id ELSE NULL END) / COUNT(order_id) * 100, 2) AS pct_Cancelled
FROM `innate-path-415009.thelook_ecommerce.orders`
GROUP BY 1
ORDER BY 1;
```
**📌 Observations** :
- 🔄 Le taux de retour est stable (~10%)
- ❌ Le taux d'abandon est autour de 15%

---

### 🚚 10) Délais moyens de livraison
```sql
WITH CTE AS (
  SELECT
    order_id,
    EXTRACT(YEAR FROM created_at) AS yr,
    TIMESTAMP_DIFF(delivered_at, shipped_at, DAY) AS days_diff
  FROM `innate-path-415009.thelook_ecommerce.orders`
)
SELECT 
  yr,
  ROUND(AVG(days_diff), 2) AS Average_delivery_time
FROM CTE
GROUP BY 1
ORDER BY 1;
```
**📌 Résultat** : Délais de livraison stables (~2 jours)

---

### 📦 11) Délais moyens d'envoi
```sql
WITH CTE AS (
  SELECT
    order_id,
    EXTRACT(YEAR FROM created_at) AS yr,
    TIMESTAMP_DIFF(shipped_at, created_at, DAY) AS days_diff
  FROM `innate-path-415009.thelook_ecommerce.orders`
)
SELECT 
  yr,
  ROUND(AVG(days_diff), 2) AS Average_shipping_time
FROM CTE
GROUP BY 1
ORDER BY 1;
```
**📌 Résultat** : Délais d'envoi stables (~1 jour)

---

## 🛍️ II - Table `order_items`

### 🔢 1) Nombre de lignes dans la table `order_items`
```sql
SELECT *
FROM `innate-path-415009.thelook_ecommerce.order_items`;
```
**📌 Résultat** : 181 652 lignes (niveau de granularité au niveau du produit)

---

### 💰 2) Total des ventes, nombre de produits par commande et montant moyen par commande
```sql
SELECT
  ROUND(SUM(sale_price), 2) AS total_sales,
  ROUND(COUNT(*) / COUNT(DISTINCT order_id), 2) AS average_items_per_order,
  ROUND(SUM(sale_price) / COUNT(DISTINCT order_id), 2) AS AOV 
FROM `innate-path-415009.thelook_ecommerce.order_items`;
```
**📌 Résultat** :
- 💶 Total des ventes sur la période : 10 784 634€
- 📦 1,45 produits par commande
- 💳 Montant moyen par commande : 86€

---

### 📆 3) Total des ventes, nombre de produits par commande et montant moyen par commande par année
```sql
SELECT
  EXTRACT(YEAR FROM created_at) AS yr,
  ROUND(SUM(sale_price), 2) AS total_sales,
  ROUND(COUNT(*) / COUNT(DISTINCT order_id), 2) AS average_items_per_order,
  ROUND(SUM(sale_price) / COUNT(DISTINCT order_id), 2) AS AOV 
FROM `innate-path-415009.thelook_ecommerce.order_items`
GROUP BY 1
ORDER BY 1;
```
**📌 Observations** :
- 📈 L'activité a fortement augmenté jusqu'à 2023 avant de se stabiliser en 2024
- 📊 Stabilité observée sur le nombre de produits par commande et le montant moyen

---

## 🏢 III - Table `distribution_centers`

### 🔢 1) Nombre de lignes dans la table `distribution_centers`
```sql
SELECT * 
FROM `innate-path-415009.thelook_ecommerce.distribution_centers`;
```
**📌 Résultat** : 10 centres de distribution, tous situés aux USA

---

## 📦 IV - Table `inventory_items`

### 🔢 1) Nombre de lignes dans la table `inventory_items`
```sql
SELECT * 
FROM `innate-path-415009.thelook_ecommerce.inventory_items`;
```
**📌 Résultat** : 490 590 lignes

---

### 🏷️ 2) Nombre de catégories, produits, marques
```sql
SELECT 
  COUNT(*) AS total_rows,
  COUNT(DISTINCT product_id) AS unique_products,
  COUNT(DISTINCT product_category) AS unique_categories,
  COUNT(DISTINCT product_brand) AS unique_brands
FROM `innate-path-415009.thelook_ecommerce.inventory_items`;
```
**📌 Résultat** :
- 🛍️ 29 060 produits uniques
- 📂 26 catégories de produits
- 🏷️ 2 754 marques

---

### 📊 3) Répartition par catégorie et marque
```sql
SELECT 
  product_category, 
  COUNT(product_id) AS number_of_products
FROM `innate-path-415009.thelook_ecommerce.inventory_items`
GROUP BY product_category
ORDER BY number_of_products DESC
LIMIT 10;
```
**📌 Résultat** : La catégorie **Intimates** contient le plus de produits (36 516)

```sql
SELECT 
  product_brand, 
  COUNT(product_id) AS number_of_products
FROM `innate-path-415009.thelook_ecommerce.inventory_items`
GROUP BY product_brand
ORDER BY number_of_products DESC
LIMIT 10;
```
**📌 Résultat** : La marque **Allegra K** possède le plus de produits (16 742)

---

### 💰 4) Répartition des prix
```sql
SELECT 
  MIN(product_retail_price) AS min_price,
  MAX(product_retail_price) AS max_price,
  AVG(product_retail_price) AS avg_price,
  COUNT(CASE WHEN product_retail_price < 10 THEN 1 ELSE NULL END) AS low_price_products
FROM `innate-path-415009.thelook_ecommerce.inventory_items`;
```
**📌 Résultat** :
- 📉 Prix minimum : 0.019€
- 📈 Prix maximum : 999€
- 📊 Prix moyen : 59,37€
- 💰 Produits < 10€ : 21 563

---

### 🏢 5) Vérification des distributions géographiques
```sql
SELECT 
  product_distribution_center_id, 
  COUNT(product_id) AS number_of_products
FROM `innate-path-415009.thelook_ecommerce.inventory_items`
GROUP BY product_distribution_center_id
ORDER BY number_of_products DESC;
```
**📌 Résultat** : Les centres de distribution **1, 2 et 3** ont le plus de stock (> 60 000)

---

### 📅 6) Dates importantes
```sql
SELECT 
  MIN(created_at) AS oldest_product_sold,
  MAX(created_at) AS most_recent_product_sold,
  COUNT(CASE WHEN sold_at IS NOT NULL THEN 1 ELSE NULL END) AS products_sold
FROM `innate-path-415009.thelook_ecommerce.inventory_items`;
```
**📌 Résultat** :
- 🛍️ Première vente : 26/11/2018
- 📆 Vente la plus récente : 30/06/2024

---

### ❌ 7) Produits invendus
```sql
SELECT 
  COUNT(*) AS unsold_products
FROM `innate-path-415009.thelook_ecommerce.inventory_items`
WHERE sold_at IS NULL;
```
**📌 Résultat** : 308 938 produits invendus

---

## 🏷️ V - Table `products`

### 🔢 1) Nombre de lignes dans la table `products`
```sql
SELECT *
FROM `innate-path-415009.thelook_ecommerce.products`;
```

---

### ❌ 2) Calcul des invendus
```sql
SELECT 
  p.category AS product_category,
  p.brand AS product_brand,
  COUNT(i.id) AS total_stock,
  COUNT(CASE WHEN i.sold_at IS NULL THEN 1 ELSE NULL END) AS unsold_items,
  CASE 
    WHEN COUNT(i.id) = 0 THEN 0 
    ELSE ROUND(
      COUNT(CASE WHEN i.sold_at IS NULL THEN 1 ELSE NULL END) * 100.0 / COUNT(i.id), 
      2
    )
  END AS shortage_rate
FROM `innate-path-415009.thelook_ecommerce.products` p
LEFT JOIN `innate-path-415009.thelook_ecommerce.inventory_items` i
ON p.id = i.product_id
GROUP BY p.category, p.brand
ORDER BY 3 DESC, 5 DESC;
```
**📌 Résultat** : Analyse du taux d'invendus par catégorie et marque

---

## 👥 VI - Table `users`

### 🔢 1) Nombre de lignes dans la table `users`
```sql
SELECT * 
FROM `innate-path-415009.thelook_ecommerce.users`;
```
**📌 Résultat** : 100 000 utilisateurs

---

### 🔑 2) Vérification unicité clé primaire
```sql
SELECT id, COUNT(*)
FROM `innate-path-415009.thelook_ecommerce.users`
GROUP BY id
HAVING COUNT(*) > 1;
```
**📌 Résultat** : La colonne `id` est confirmée comme clé primaire

---

### 🛍️ 3) Nombre d'utilisateurs actifs (avec au moins une commande)
```sql
SELECT u.id, COUNT(o.order_id)
FROM `innate-path-415009.thelook_ecommerce.users` u
LEFT JOIN `innate-path-415009.thelook_ecommerce.orders` o
ON u.id = o.user_id 
GROUP BY u.id
HAVING COUNT(o.order_id) >= 1;
```
**📌 Résultat** : 80 130 utilisateurs ont passé au moins une commande

---

### 🌍 4) Nombre de commandes par pays
```sql
WITH CTE AS (
  SELECT u.id AS user_id, u.country, COUNT(o.order_id) AS number_of_orders
  FROM `innate-path-415009.thelook_ecommerce.users` u
  LEFT JOIN `innate-path-415009.thelook_ecommerce.orders` o
  ON u.id = o.user_id 
  GROUP BY u.id, u.country
  HAVING COUNT(o.order_id) >= 1
)
SELECT country, SUM(number_of_orders) AS total_orders_by_country
FROM CTE
GROUP BY country
ORDER BY total_orders_by_country DESC;
```
**📌 Résultat** :
- 🇨🇳 1/3 des commandes proviennent de Chine
- 🇺🇸 🇧🇷 Environ 70% des commandes proviennent de Chine, USA et Brésil

---

### 📊 5) Exploration des utilisateurs n'ayant jamais passé de commande par pays
```sql
WITH CTE1 AS ( 
  SELECT u.id AS user_id, u.gender, u.age, u.country
  FROM `innate-path-415009.thelook_ecommerce.users` u
  LEFT JOIN `innate-path-415009.thelook_ecommerce.orders` o
  ON u.id = o.user_id
  WHERE o.user_id IS NULL
),
CTE2 AS (
  SELECT country, COUNT(id) AS number_of_users_by_country
  FROM `innate-path-415009.thelook_ecommerce.users`
  GROUP BY country
)
SELECT CTE1.country, COUNT(CTE1.user_id) AS user_with_no_order,
  CTE2.number_of_users_by_country AS number_of_total_registered_users,
  ROUND(COUNT(CTE1.user_id) / CTE2.number_of_users_by_country * 100, 2) AS no_order_ratio
FROM CTE1
LEFT JOIN CTE2 ON CTE1.country = CTE2.country
GROUP BY CTE1.country, CTE2.number_of_users_by_country
ORDER BY user_with_no_order DESC;
```
**📌 Résultat** :
- 🇨🇴 🇯🇵 🇧🇪 Ratio élevé d'utilisateurs enregistrés n'ayant jamais commandé
- 🇨🇳 🇺🇸 Malgré un ratio plus faible (19,61 % et 19,95 %), la base d’utilisateurs est importante

---

### 🌐 6) Nombre de commandes par source de trafic
```sql
WITH CTE AS (
  SELECT u.id AS user_id, u.traffic_source, COUNT(o.order_id) AS number_of_orders
  FROM `innate-path-415009.thelook_ecommerce.users` u
  LEFT JOIN `innate-path-415009.thelook_ecommerce.orders` o
  ON u.id = o.user_id 
  GROUP BY u.id, u.traffic_source
  HAVING COUNT(o.order_id) >= 1
)
SELECT traffic_source, SUM(number_of_orders) AS total_orders_by_source
FROM CTE
GROUP BY traffic_source
ORDER BY total_orders_by_source DESC;
```

---

## 🎯 VII - Table `events`

### 🔢 1) Nombre de lignes dans la table `events`
```sql
SELECT *
FROM `innate-path-415009.thelook_ecommerce.events`;
```
**📌 Résultat** : 2 428 740 lignes

---

### 🎟️ 2) Nombre d'évènements
```sql
SELECT COUNT(*) AS number_of_events
FROM `innate-path-415009.thelook_ecommerce.events`;
```
**📌 Résultat** : 2 428 740 évènements

---

### 📊 3) Nombre de sessions
```sql
SELECT COUNT(DISTINCT session_id) AS number_of_sessions
FROM `innate-path-415009.thelook_ecommerce.events`;
```
**📌 Résultat** : 681 652 sessions

---

### 🛍️ 4) Nombre de sessions avec commandes
```sql
SELECT COUNT(DISTINCT session_id) AS number_of_sessions
FROM `innate-path-415009.thelook_ecommerce.events` 
WHERE user_id IS NOT NULL;
```
**📌 Résultat** : 181 652 sessions avec commande

---

### 📊 5) Analyse des taux de conversion (ajouts au panier et achats) par session
```sql
WITH CTE AS (
  SELECT
    session_id,
    COUNT(CASE WHEN event_type = 'home' THEN 1 ELSE NULL END) AS home_visits,
    COUNT(CASE WHEN event_type = 'department' THEN 1 ELSE NULL END) AS department_visits,
    COUNT(CASE WHEN event_type = 'product' THEN 1 ELSE NULL END) AS product_views,
    COUNT(CASE WHEN event_type = 'cart' THEN 1 ELSE NULL END) AS cart_additions,
    COUNT(CASE WHEN event_type = 'purchase' THEN 1 ELSE NULL END) AS purchases,
    COUNT(CASE WHEN event_type = 'cancel' THEN 1 ELSE NULL END) AS cancellations
  FROM `innate-path-415009.thelook_ecommerce.events`
  GROUP BY session_id
)
SELECT
  SUM(product_views) AS total_product_views,
  SUM(cart_additions) AS total_cart_additions,
  SUM(purchases) AS total_purchases,
  ROUND(SUM(cart_additions) / SUM(product_views) * 100, 2) AS cart_conversion_rate,
  ROUND(SUM(purchases) / SUM(cart_additions) * 100, 2) AS purchase_conversion_rate
FROM CTE;
```
**📌 Résultat** :
- 🛒 70,35% des vues produits → ajout au panier
- 🛍️ 30,55% des ajouts → achat

---

### ❌ 6) Volume d'abandons de paniers
```sql
SELECT COUNT(DISTINCT session_id) AS abandoned_sessions
FROM `innate-path-415009.thelook_ecommerce.events`
WHERE user_id IS NULL AND event_type = 'cart';
```
**📌 Résultat** : 250 000 sessions avec abandon de panier

---

### 📊 7) Vue agrégée des abandons de panier par source de trafic
```sql
WITH AbandonedCartSessions AS (
  SELECT session_id, traffic_source
  FROM `innate-path-415009.thelook_ecommerce.events`
  WHERE user_id IS NULL AND event_type = 'cart'
)
SELECT traffic_source, COUNT(session_id) AS abandoned_sessions
FROM AbandonedCartSessions
GROUP BY traffic_source
ORDER BY abandoned_sessions DESC;
```
**📌 Résultat** :
- 📧 Emails : 112 449 abandons
- 🔍 Adwords : 74 719 abandons
- 🌱 Trafic organique : 12 613 abandons

---

### 📉 8) Analyse des abandons de panier par source de trafic avec taux d'abandon
```sql
WITH AbandonedCartSessions AS (
  SELECT session_id, traffic_source
  FROM `innate-path-415009.thelook_ecommerce.events`
  WHERE user_id IS NULL AND event_type = 'cart'
),
TotalSessions AS (
  SELECT traffic_source, COUNT(DISTINCT session_id) AS number_of_total_sessions
  FROM `innate-path-415009.thelook_ecommerce.events`
  GROUP BY traffic_source
)
SELECT 
  acs.traffic_source,
  COUNT(acs.session_id) AS abandoned_sessions,
  ts.number_of_total_sessions AS total_sessions_final,
  ROUND(COUNT(acs.session_id) / ts.number_of_total_sessions * 100, 2) AS abandon_ratio
FROM AbandonedCartSessions acs
LEFT JOIN TotalSessions ts
ON acs.traffic_source = ts.traffic_source
GROUP BY acs.traffic_source, ts.number_of_total_sessions
ORDER BY abandoned_sessions DESC;
```
**📌 Résultat** :
- 📊 36% des sessions avec ajout au panier aboutissent à un abandon
- 📧 Les emails génèrent le plus grand nombre d’abandons

---

### 📈 9) Taux de conversion par mois
```sql
SELECT
  EXTRACT(MONTH FROM created_at) AS month,
  SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS total_purchases,
  ROUND(
    SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) / COUNT(DISTINCT session_id) * 100, 2
  ) AS purchase_conversion_rate
FROM `innate-path-415009.thelook_ecommerce.events`
GROUP BY month
ORDER BY month;
```
**📌 Résultat** :
- 📆 Juin : taux de conversion maximum (37,13%)

---
