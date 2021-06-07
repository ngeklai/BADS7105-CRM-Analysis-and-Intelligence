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
## Step 2: Text Preprocessing
Handling a task related to text requires a text preparation, in which a text is cleansed by removing meaningless word and then tokenized.
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
