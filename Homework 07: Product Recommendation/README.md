# Product Recommendation
We are provided with a list of multi items purchased by 44 classmates. So, we will take this chance to apply network graph for a product recommendation system.

**Google Colab:** [![Open In Collab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1xKXGlIYUFPAJKAHcTxsi8nO4WG5U79oR?usp=sharing)

### Step 1: Import Libraries
Below are libraries and modules we need for this task.
```javascript
import numpy as np
import pandas as pd
import seaborn as sns
import networkx as nx
import matplotlib.pyplot as plt
from mlxtend.frequent_patterns import apriori
from mlxtend.frequent_patterns import association_rules
```

### Step 2: Prepare Data
It requires to check how our data looks like first.
```javascript
df = pd.read_csv('/content/Customer Survey.csv')
df.head(5)
```
![Picture1](https://user-images.githubusercontent.com/59596996/122670867-4ed5fb00-d1ee-11eb-8cae-e47b37b01c60.jpg)

We disregard unnecessary columns and have to alter the qualitative response (yes/no) to a quantitative binary data (1/0).
![Picture2](https://user-images.githubusercontent.com/59596996/122671313-2ea73b80-d1f0-11eb-8b03-c2b8bab2348b.jpg)

### Step 3: Conduct EDA
We explore and analyze the data here.
```javascript
df.describe().T.sort_values('mean', ascending=False)
```
![Picture3](https://user-images.githubusercontent.com/59596996/122671659-facd1580-d1f1-11eb-9cf4-28666af37920.jpg)
```javascript
plt.figure(figsize=(10, 8))
df.sum(axis=0).sort_values(ascending=False).tail(15).plot(kind='barh').invert_yaxis()
```
![Picture4](https://user-images.githubusercontent.com/59596996/122671718-3a93fd00-d1f2-11eb-8f09-93fdd0045ddc.png)

Then, we perform an analysis to know which items are purchased simulatenously or, so-called, in the same basket. This is known as Market Basket Analysis.
### Step 4: Conduct Market Basket Analysis
Market Basket Analysis could be done by setting up Association Rule, in which you will two challenges; one is identifying "Frequent Itemsets", and another is constructing "Strong Rule". Apriori Algorithm is helpful for these purposes.
#### Step 4.1: Generating Frequent Itemsets
Frequent Itemsets are those sets of items which surpass the threshold you have set as criteria, or a minimum support.
```javascript
freq_itemsets = apriori(df, min_support=0.5, use_colnames=True)
freq_itemsets.head(10)
```
![Picture5](https://user-images.githubusercontent.com/59596996/122671943-2b617f00-d1f3-11eb-9bc4-a545d289e973.jpg)

#### Step 4.2: Generating Rules
The above-mentioned Strong Rule is the rule associating itemsets by a high degree of Confidence or Lift
```javascript
rule = association_rules(frequent_itemsets, metric='lift', min_threshold=1)
rule.sort_values('lift',ascending=False).head(5)
```
![Picture6](https://user-images.githubusercontent.com/59596996/122671996-82ffea80-d1f3-11eb-9839-b07bc274fb1a.jpg)

### Step 5: Visualize the Network
You will see that, for example, those who puchased grilled beef had a strong correlation in purchasing lego and authorized software.
So, we can use a network graph to generate a recommendation for other products as for an upselling technique when your client makes a purchase. 
```javascript
plt.figure(figsize=(15, 15))
G = nx.from_pandas_edgelist(simple_rules, source='antecedents', target='consequents', edge_attr='lift')
pos = nx.circular_layout(G)
nx.draw_networkx_nodes(G, pos, nodelist=dict(G.degree).keys(), node_size=[s*500 for s in dict(G.degree).values()], node_color='salmon')
nx.draw_networkx_labels(G, pos)
nx.draw_networkx_edges(G, pos, alpha=0.3, width=[w for w in dict(G.degree).values()], edge_color='slategray')
nx.draw_networkx_edge_labels(G, pos, nx.get_edge_attributes(G, 'lift'))
plt.show()
```
![Picture7](https://user-images.githubusercontent.com/59596996/122672061-d40fde80-d1f3-11eb-8ee5-e2482032e1fa.png)
