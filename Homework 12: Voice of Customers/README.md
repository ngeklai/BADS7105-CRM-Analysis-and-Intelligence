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
* Steps in Brief: Customer Reviews -> Documents -> TF-IDF Vectorization -> Cosine Similarity

By this assignment, we are provided with list of customer reviews about three shabu restaurants operating mainly in BKK, Thailand.
Such list is comprised of eight columns; Review ID, Restaurant ID, Restaurant, User, Headline, Review, and Rating.

NOTE:
Referring to NLP Document Similarity, we then concatenate customer reviews of each restaurant, generating three different texts (documents).

After combining customer reviews of each restaurant completely, we have in our hand now three documents. 
We import them by Python and sequent procedures.

## Step 1: Install & Import Libraries
We have to install an upgraded version of PyThaiNLP as customer reviews are done in Thai language.
Also, we have to use other necessary libraries and modules.
```javascript
!pip install --upgrade pythainlp
import pythainlp
import warnings
warnings.filterwarnings("ignore", category=DeprecationWarning)
from sklearn.feature_extraction.text import TfidfVectorizer 
import numpy as np 
```
## Step 2: Preprocess Texts
Handling a text task requires a text preparation, in which a text is cleansed by removing meaningless word and then tokenized.
```javascript
stopwords = list(pythainlp.corpus.thai_stopwords())
removed_words = [' ', '  ', '\n', 'ร้าน', '(', ')' , '           ']
screening_words = stopwords + removed_words

def tokenize_with_space(sentence):
  merged = ''
  words = pythainlp.word_tokenize(str(sentence), engine='newmm')
  for word in words:
    if word not in screening_words:
      merged = merged + ',' + word
  return merged[1:]
  ```
## Step 3: Transform Text to TF-IDF and Vector
Now, all documents are set ready. We then convert words into TF-IDF first and vectorize it.
```javascript
tfidfvectoriser=TfidfVectorizer()
tfidfvectoriser.fit(documents['Review_tokenized'] )

tfidf_vectors=tfidfvectoriser.transform(documents['Review_tokenized'] )
```
## Step 4: Calculate Cosine Similarity
Upon putting TF-IDF into a vector space, we deploy dot product for cosine similarity calculation.
The calculation is conducted by a pairwise of document indices.
```javascript
pairwise_similarities=np.dot(tfidf_vectors,tfidf_vectors.T).toarray()
pairwise_similarities
```
We will develop a function to support an identification of the most similar document.
```javascript
def most_similar(doc_id,similarity_matrix,matrix):
    print (f'Document: {documents.iloc[doc_id]["Review"]}')
    print ('\n')
    print ('Similar Documents:')
    if matrix=='Cosine Similarity':
        similar_ix=np.argsort(similarity_matrix[doc_id])[::-1]
    for ix in similar_ix:
        if ix==doc_id:
            continue
        print('\n')
        print (f'Document: {documents.iloc[ix]["Review"]}')
        print (f'{matrix} : {similarity_matrix[doc_id][ix]}')
```
The below function is the product I have proposed for this assignment.
Applicably, you can change an index in the function argument. The index variance represents documents (customer reviews) of each shabu restaurant.
```javascript
most_similar(0,pairwise_similarities,'Cosine Similarity')
```
