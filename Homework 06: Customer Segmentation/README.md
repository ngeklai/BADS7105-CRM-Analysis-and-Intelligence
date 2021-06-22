# Customer Segmentation
I will use Supermarket dataset to segment customers by querying data from Google BigQuery and creating K-Mean clustering model by Python in Google Colab.

**Google Colab:**[![Open In Collab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1qGr2a_C0wnt3Na2g1bH8emHEDCN0rYSk?usp=sharing)
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
