# Who Is Our Competitor? Cosine Similarity Approach Based on Customer Reviews.
Suppose that you were a business owner. By which means would you use to identify who your competitors are in the market?

As the market has been developing, commercial landscape might be changed and it turns to be different from what you expected since the beginning. 
Hence, there is a need to verify/ benchmark your market position.
Actually, there are several ways to identify your competitors. But, a method I am proposing here is quite simple, fast, and at cheap price.
However, the following assumption needs to be agreed upon;

## Concept
* Assumption: a competitor shares a common trait of your customers.
* Dataset: a list of customer reviews
* Method: NLP Document Similarity
* Brief Steps: Customer Reviews -> Documents -> TF-IDF Vectorization -> Cosine Similarity

## Step 1: Data Preparation
    import pandas as pd
    documents = pd.read_csv('/content/CustomerReviews(R01).csv')
    documents['Restaurant']
