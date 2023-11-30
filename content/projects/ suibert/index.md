+++
title = 'Analyzing Suicide-Related Discourse on X: A BERT-Based Exploration of Emotion Dimensions and Engagement'
date = 2023-11-25T17:36:18+01:00
draft = true
slug = 'BERT-suicide-analysis'
tags = ['machine learning', 'social media analysis', 'mental health', 'user engagement', 'project']
+++

{{< lead >}}
Suicide stands as a significant global issue, claiming more than 700,000 lives each year. However, there exists a potential for effective prevention through the implementation of evidence-based interventions targeting at-risk individuals. Therefore, the accurate identification of signs indicating suicidal tendencies and a comprehensive understanding of suicidal behavior are pivotal for the development of proactive and effective prevention strategies. This project focuses on fine-tuning the state-of-the-art deep learning model BERT for the purpose of detecting tweets associated with suicide. Subsequently, the negative binomial regression model is applied to delve into the intricate relationships among suicidal tendencies, emotional dimensions, and engagement levels.

{{< /lead >}}

Suicidal thoughts and behaviors (STBs) have become urgent public health concerns. However, obtaining sufficient information through traditional methods, such as self-reported questionnaire surveys, is challenging due to the sensitive and complex nature of suicide. With the development of the Internet, social media platforms are considered powerful tools for identifying and understanding individuals at risk. In this project, 120,000 suicide-related tweets are collected along with author details and engagement data. Following this, SuiBERT, a specialized BERT model for detecting suicidal content, is developed. Afterward, dimensional emotional scores (valence and arousal) are calculated using a lexicon-based approach. Last, the regression analysis is conducted to explore the relationships between emotional dimensions, engagement, and suicidality. 

Notably, SuiBERT demonstrated an impressive accuracy of 90.92% and a recall of 90.40%. Moreover, the regression analysis unveiled a positive correlation between emotional scores and engagement size, while revealing a negative correlation with conversion rates.

## 1. Basic Concepts, Variables, and Theoretical Framework


## 2. Collecting Suicide-related Disclose on X

### 2.1 Data Collection
To perform empirical analysis, publicly posted tweets on X related to suicide are collected using Python language. The keywords utilized for data collection are \textquote{suicide} and \textquote{suicidal}. The dataset covers the period from December 2022 to May 2023, with a monthly target of 20,000 tweets. Consequently, a comprehensive dataset comprising 120,000 samples was amassed, spanning 1 to 3 days for each month.

There are 2 main purposes for the selection of this specific timeframe for data collection. Firstly, it effectively captures the metric *view*, a crucial component of the dependent variable in this study, that was introduced only on December 22, 2022. Additionally, by strategically spanning the entire day and encompassing multiple months, the dataset can capture fluctuating suicide-related tendencies across different time segments. This approach also mitigates potential biases related to seasons or festivals, ensuring the extraction of the most authentic and nuanced information from users.

The complete dataset includes the following attributes: *UserID, Tweet, Timestamp, Retweets, Likes, Replies, Views, Bookmarks, Location, Followers, Followings, TotalTweets, Hashtags, HasURL*.

### 2.2 Data Cleaning
The collected raw data from X is further cleaned to ensure the authenticity of the data for future analysis. This involves filtering out low-activity account samples with a total tweet count of less than 30 and a follower count of less than 10, and then missing values are checked and removed. The samples of the filtered database were reduced from 120,000 to 105,315. The data cleansing process ensures data accuracy, consistency, and relevance in subsequent analyses, and therefore facilitates accurate and consistent analysis results.

***Code for Data Cleaning***

```python
import pandas as pd
import re
import string
import nltk
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer, WordNetLemmatizer
from nltk.tokenize import word_tokenize

nltk.download('punkt')
nltk.download('stopwords')
nltk.download('wordnet')

# Load the data from collected csv file
df = pd.read_csv("tweets_20000_per_month_halfyear2.csv",low_memory=True)

# Account selection: drop rows with TotalTweets < 30 and Followers < 10
df = df[(df['TotalTweets'] >= 30) & (df['Followers'] >= 10)]

# Preprocessing
stop_words = set(stopwords.words('english'))
ps = PorterStemmer()
lemmatizer = WordNetLemmatizer()

def preprocess_text(text):
    # Replace links with the keyword "LINK"
    text = re.sub(r'http\S+|www\S+', 'LINK', text)
    
    # Lowercase
    text = text.lower()
    
    # Tokenization
    tokens = word_tokenize(text)
    
    # Removing punctuation, numbers, and stop words
    tokens = [token for token in tokens if token.isalpha() and token not in string.punctuation and token not in stop_words]
    
    # Stemming
    stemmed_tokens = [ps.stem(token) for token in tokens]
    
    # Lemmatization
    lemmatized_tokens = [lemmatizer.lemmatize(token) for token in stemmed_tokens]
    
    # Join tokens into a single string
    preprocessed_text = ' '.join(lemmatized_tokens)
    
    return preprocessed_text

# Apply text preprocessing to the 'Tweet' column
df['CleanedTweet'] = df['Tweet'].apply(preprocess_text)

# Export the cleaned data to tweets_100_cleaned.csv
df.to_csv('tweets_20000_per_month_halfyear2.csv-Clean.csv', index=False)
```

## 3. Classifying Suicidal Post Using Fine-tined BERT Model
In this section, SuiBERT, a BERT-based fine-tuned model, is developed. Its purpose is to classify the acquired suicide-related data into two categories: *suicidal* and *non-suicidal*, providing corresponding probability scores for each classification.

To accomplish this, the *bert-base-uncased*[^1] model (hidden vectors: 768, self-attention heads: 12), which does not possess case sensitivity during tokenization, is selected as the baseline model. The fact that BERT was pre-trained on large yet general datasets necessitates a subsequent fine-tuning stage with a downstream-specific dataset to achieve optimal application performance \parencite{Devlin2018}. Consequently, a semi-supervised machine learning approach is utilized to fine-tune the pre-trained \textit{bert-base-uncased} model and establish a connection to its application in suicidality detection. This approach involves a stepwise process of iteratively fine-tuning the model using a limited number of labelled samples, followed by its deployment on a larger volume of unlabelled data.


### 3.1. Initialization of Fine-Tuning Process of SuiBERT



## 4. Calculating Dimensional Emotion Scores






<div style="display:flex; gap:6px">

{{< badge >}} machine learning {{< /badge >}}

{{< badge >}} social media analysis {{< /badge >}}

{{< badge >}} mental health {{< /badge >}}

{{< badge >}} user engagement {{< /badge >}}

{{< badge >}} project {{< /badge >}}
</div>



