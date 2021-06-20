# Product Recommendations
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
