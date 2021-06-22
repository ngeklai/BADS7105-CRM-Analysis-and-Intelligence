# Customer Segmentation
I will use Supermarket dataset to segment customers by querying data from Google BigQuery and creating K-Mean clustering model by Python in Google Colab.

**Google Colab:** [![Open In Collab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1qGr2a_C0wnt3Na2g1bH8emHEDCN0rYSk?usp=sharing)
### 1. Google BigQuery
Below SQL script is employed to query data from the dataset.
```sql
SELECT
    S.CUST_CODE,
    SUM(S.SPEND) AS TOTAL_SPEND,
    AVG(M.MONTHLY_SPEND) AS AVG_MONTHLY_SPEND,
    STDDEV_POP(M.MONTHLY_SPEND) AS STD_MONTHLY_SPEND,
    COUNT(DISTINCT S.BASKET_ID) AS TOTAL_VISIT,
    AVG(M.MONTHLY_VISIT) AS AVG_MONTHLY_VISIT,
    STDDEV_POP(M.MONTHLY_VISIT) AS STD_MONTHLY_VISIT,
    CASE
    WHEN SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END)
     AND SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END) THEN 3
    WHEN SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END)
     AND SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) THEN 2
    WHEN SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='M' THEN 1 ELSE 0 END)
     AND SUM(CASE WHEN S.BASKET_SIZE='S' THEN 1 ELSE 0 END) >= SUM(CASE WHEN S.BASKET_SIZE='L' THEN 1 ELSE 0 END) THEN 1
    END AS MODE_BASKET_SIZE,
    DATE_DIFF(PARSE_DATE('%Y%m%d', CAST(MAX(S.SHOP_DATE) AS STRING)), PARSE_DATE('%Y%m%d', CAST(MIN(S.SHOP_DATE) AS STRING)), DAY) AS CUST_LIFETIME,
    DATE_DIFF(DATE'2008-07-06', PARSE_DATE('%Y%m%d', CAST(MIN(S.SHOP_DATE) AS STRING)), DAY) AS DURATION_FROM_FIRST_PURCHASE,
    DATE_DIFF(DATE'2008-07-06', PARSE_DATE('%Y%m%d', CAST(MAX(S.SHOP_DATE) AS STRING)), DAY) AS DURATION_FROM_LAST_PURCHASE
FROM
    `secure-stone-272416.BADS7105.bads7105` AS S
INNER JOIN (
    SELECT
        CUST_CODE,
        LEFT(CAST(SHOP_DATE AS STRING), 6) AS SHOP_MONTH,
        SUM(SPEND) AS MONTHLY_SPEND,
        COUNT(DISTINCT BASKET_ID) AS MONTHLY_VISIT
    FROM
        `secure-stone-272416.BADS7105.bads7105`
    GROUP BY
        CUST_CODE, SHOP_MONTH
) AS M ON S.CUST_CODE = M.CUST_CODE
WHERE
    S.CUST_CODE IS NOT NULL
GROUP BY
    S.CUST_CODE
ORDER BY
    S.CUST_CODE
```
### 2. Import libraries and modules
All libraries and modules necessitated for this task are imported.
```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import silhouette_score
from sklearn.preprocessing import scale, StandardScaler
```
Here how the data queries looks like.
```python
df = pd.io.gbq.read_gbq(SQL, project_id=project_id)
df.head(10)
```
![Picture9](https://user-images.githubusercontent.com/59596996/122940653-db272000-d39e-11eb-8246-84cbd2ee3156.jpg)
The followings are 7 features I will use for modelling the K-mean.
* AVG_MONTHLY_SPEND
* STD_MONTLY_SPEN
* AVG_MONTHLY_VISIT
* STD_MONTLY_VISIT
* MODE_BASKET_SIZE
* DURATION_FROM_LAST_PURCHASE
* CUST_LIFETIME
### 3. Number of Clusters
To decide about an appropriate number of clusters, I deploy the Elbow Method and Silhouette. The result of both methods is shown below. It seems K=3 should be selected.

![Picture10](https://user-images.githubusercontent.com/59596996/122943624-5e497580-d3a1-11eb-9a74-ef521c345aec.png)
### 4. Feature Importance
I use Random Forest to help determine which feature are important in clustering process.
```python
x = df_cluster[Features]
y = df_cluster['CLUSTER']
clf = RandomForestClassifier(criterion = 'entropy').fit(x, y)
df_importance = pd.DataFrame({'Feature': Features, 'Importance' : clf.feature_importances_}).set_index('Feature')
df_importance.sort_values('Importance').plot.barh(title='Repeat-Purchase Customers - Feature Importance')
```
![Picture11](https://user-images.githubusercontent.com/59596996/122944458-0b23f280-d3a2-11eb-8ddd-1cc139d875b5.png)

As the result, this K-mean model listed 5 important features; CUST_LIFETIME, DURATION_FROM_LAST_PURCHASE, STD_MONTHLY_SPEND, STD_MONTHLY_VISIT, and AVG_MONTHLY_VISIT.
### 5. Result Analysis
As K=3, I name these 3 clusters as Churn, Active, and Premium. By the density plots, I could explore the characteristics of each cluster the density plots as follow;
* #Churn: One time purchase and not active for over 1 year.
* 
```python
df_eda = df_cluster[['CLUSTER','TOTAL_SPEND','TOTAL_VISIT','AVG_MONTHLY_SPEND','STD_MONTHLY_SPEND','AVG_MONTHLY_VISIT',
                     'STD_MONTHLY_VISIT','MODE_BASKET_SIZE','CUST_LIFETIME','DURATION_FROM_FIRST_PURCHASE','DURATION_FROM_LAST_PURCHASE']]
df_eda.sort_values(by=['CLUSTER'], inplace=True)
df_eda['CLUSTER'].replace({0:'Churned',1:'Active',2:'Premium'}, inplace=True)
fig, axes = plt.subplots(5, 2, figsize=(12, 20), tight_layout=True)
axes = axes.ravel()
for col, ax in zip(df_eda.iloc[:,1:].columns, axes):
  sns.kdeplot(data=df_eda, x=col, hue='CLUSTER', palette='tab10', ax=ax)
plt.show()
```
![Picture12](https://user-images.githubusercontent.com/59596996/122954284-7fae5f80-d3a9-11eb-8006-c920ad1340d1.png)

From the density plots, it can be concluded as below.

CHURNED
One time purchase
Not active for over 1 year
ACTIVE
Still active within 3 months
Visit not frequently (< 4 times/month)
Spend less money per visit (< 300 USD/time)
PREMIUM
Still active within 3 months
Visit frequently (>= 4 times/month)
Spend much money per visit (>= 300 USD/time)
