## jointtsmodel

[![License](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat)](LICENSE.md)

This is a consolidated library for joint topic-sentiment (jst) models. 

### Description

Joint topic-sentiment models extract topical as well as sentiment information for each text. This library contains different jst models - JST, RJST, TSM, sLDA and TSWE.
    
### Installation

```
git clone https://github.com/victor7246/jointtsmodel.git
cd jointtsmodel
python setup.py install
```

Or from pip:

```
pip install jointtsmodel
```

### Usage

We can use vectorized texts to run joint topic-sentiment models.

```
from jointtsmodel.RJST import RJST
from jointtsmodel.JST import JST
from jointtsmodel.sLDA import sLDA
from jointtsmodel.TSM import TSM
from jointtsmodel.TSWE import TSWE

import pandas as pd
import numpy as np
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.datasets import fetch_20newsgroups
from jointtsmodel.utils import *

# This produces a feature matrix of token counts, similar to what
# CountVectorizer would produce on text.
data, _ = fetch_20newsgroups(shuffle=True, random_state=1,
                         remove=('headers', 'footers', 'quotes'),
                         return_X_y=True)
data = data[:1000]
vectorizer = CountVectorizer(max_df=0.7, min_df=10,
                            max_features=5000,
                            stop_words='english')
X = vectorizer.fit_transform(data)
vocabulary = vectorizer.get_feature_names()
inv_vocabulary = dict(zip(vocabulary,np.arange(len(vocabulary))))
lexicon_data = pd.read_excel('lexicon/prior_sentiment.xlsx')
lexicon_data = lexicon_data.dropna()
lexicon_dict = dict(zip(lexicon_data['Word'],lexicon_data['Sentiment']))
```

For JST model use
```
model = JST(n_topic_components=5,n_sentiment_components=5,random_state=123,evaluate_every=2)
model.fit(X.toarray(), lexicon_dict)

model.transform()[:2]

top_words = list(model.getTopKWords(vocabulary).values())
coherence_score_uci(X.toarray(),inv_vocabulary,top_words)
Hscore(model.transform())
```

For RJST model use
```
model = RJST(n_topic_components=5,n_sentiment_components=5,random_state=123,evaluate_every=2)
model.fit(X.toarray(), lexicon_dict)

model.transform()[:2]

top_words = list(model.getTopKWords(vocabulary).values())
coherence_score_uci(X.toarray(),inv_vocabulary,top_words)
Hscore(model.transform())
```

For TSM use
```
model = TSM(n_topic_components=5,n_sentiment_components=5,random_state=123,evaluate_every=2)
model.fit(X.toarray(), lexicon_dict)

model.transform()[:2]

top_words = list(model.getTopKWords(vocabulary).values())
coherence_score_uci(X.toarray(),inv_vocabulary,top_words)
Hscore(model.transform())
```

For sLDA model use
```
model = sLDA(n_topic_components=5,n_sentiment_components=5,random_state=123,evaluate_every=2)
model.fit(X.toarray(), vocabulary)

model.transform()[:2]

top_words = list(model.getTopKWords(vocabulary).values())
coherence_score_uci(X.toarray(),inv_vocabulary,top_words)
Hscore(model.transform())
```

For TSWE model we need word embedding matrix as an input. 
```
embeddings_index = {}
f = open('embeddings/glove.6B.100d.txt','r',encoding='utf8')

for i, line in enumerate(f):
    values = line.split()
    word = values[0]
    coefs = np.asarray(values[1:], dtype='float32')
    embeddings_index[word] = coefs
f.close()

print('Found %s word vectors.' % len(embeddings_index))

embedding_matrix = np.zeros((X.shape[1], 100))

for i, word in enumerate(vocabulary):
    if word in embeddings_index:
        embedding_matrix[i] = embeddings_index[word]
    else:
        embedding_matrix[i] = np.zeros(100)
```

Run TSWE model
```
model = TSWE(embedding_dim=100,n_topic_components=5,n_sentiment_components=5,random_state=123,evaluate_every=2)
model.fit(X.toarray(), lexicon_dict, embedding_matrix)

model.transform()[:2]

top_words = list(model.getTopKWords(vocabulary).values())
coherence_score_uci(X.toarray(),inv_vocabulary,top_words)
Hscore(model.transform())
```
### To do

* Add parallelization for faster execution
* Handle sparse matrix
* Add online JST models

### References -

[1] https://www.researchgate.net/figure/JST-and-Reverse-JST-sentiment-classification-results-with-multiple-topics_fig1_47454505

[2] https://www.aaai.org/ocs/index.php/AAAI/AAAI10/paper/viewFile/1913/2215

[3] https://hal.archives-ouvertes.fr/hal-02052354/document

[4] https://github.com/ayushjain91/Sentiment-LDA

[5] https://gist.github.com/mblondel/542786

[6] http://ceur-ws.org/Vol-1646/paper6.pdf
