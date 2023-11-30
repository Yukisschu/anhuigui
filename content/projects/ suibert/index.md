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

## 1. Basic Concepts, Variables, and Theoretical Framework {#1.}


## 2. Collecting Suicide-related Disclose on X {#2.}

### 2.1 Data Collection {#2.1.}
To start, public posts on X related to suicide are collected using Python. The keywords utilized for data collection are ***suicide*** and ***suicidal***. The timeframe covers the period from December 2022 to May 2023, with a monthly target of 20,000 tweets. Consequently, a comprehensive dataset comprising 120,000 samples was amassed, spanning 1 to 3 days for each month. 

There are 2 main purposes for the selection of this specific timeframe for data collection. Firstly, it effectively captures the metric ***view***, a crucial component of the dependent variable in this study, that was introduced only on December 22, 2022. Additionally, by strategically spanning the entire day and encompassing multiple months, the dataset can capture fluctuating suicide-related tendencies across different time segments. This approach also mitigates potential biases related to seasons or festivals, ensuring the extraction of the most authentic and nuanced information from users.

Apart from the post content, the complete dataset also includes the following attributes: ***UserID, Tweet, Timestamp, Retweets, Likes, Replies, Views, Bookmarks, Location, Followers, Followings, TotalTweets, Hashtags, HasURL***.

### 2.2. Data Cleaning {#2.1.}

The collected raw data from X is further cleaned to ensure the authenticity of the data for future analysis. This involves filtering out low-activity account samples with a total tweet count of less than 30 and a follower count of less than 10, and then missing values are checked and removed. The samples of the filtered database are reduced from 120,000 to 105,315. The data cleansing process ensures data accuracy, consistency, and relevance in subsequent analyses, and therefore facilitates accurate and consistent analysis results.

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

## 3. Classifying Suicidal Post Using Fine-tined BERT Model {#3.}

In this section, SuiBERT, a BERT-based fine-tuned model, is developed. Its purpose is to classify the acquired suicide-related data into two categories: ***suicidal*** and ***non-suicidal***, providing corresponding probability scores for each classification.

To achieve this objective, the ***bert-base-uncased*** [^1] model, featuring 768 hidden vectors and 12 self-attention heads, is chosen as the baseline model. The selection of BERT, pre-trained on expansive yet general datasets, mandates a subsequent fine-tuning stage using a downstream-specific dataset. This fine-tuning process is essential to attain optimal performance in the specific application context[^1].

{{< github repo="google-research/bert" >}}

As a result, a semi-supervised machine learning approach is employed to refine the pre-trained ***bert-base-uncased*** model and bridge its application to suicidality detection. This method entails a stepwise process of iteratively fine-tuning the model with a restricted number of labeled samples, subsequently deploying it on a more extensive pool of unlabeled data.

### 3.1. Initialization of Fine-Tuning Process of SuiBERT {#3.1.}

To initiate the fine-tuning process, the PyTorch framework (*version: 2.0.1+cu118*) and transformers library (*version: 4.30.2*) are deployed. And AdamW ({{< katex >}}\\(beta_1 = 0.9, beta_2 = 0.999\\)) is selected as the optimizer. Given the scarcity of positively labeled suicidal instances, the cross-entropy loss function is employed to impose stricter penalties for misclassifying suicidal cases.

Furthermore, the recommended range of hyperparameters are[^1]:
- Batch size: 16, 32;
- Learning rate (Adam): 5 × 10<sup>-5</sup>, 3 × 10<sup>-5</sup>, 2 × 10<sup>-5</sup>;
- item Number of epochs: 2, 3, 4.

Considering both hardware capabilities and database scale, the foundational learning rate is configured at 2 × 10<sup>-5</sup>. The training process spans 5 epochs, with a training batch size of 32 and an evaluation batch size of 8. All computations and tasks are efficiently executed on the GPU A100 for enhanced performance.

### 3.2. Dataset Distribution and Stratified Split for Fine-Tuning SuiBERT {#3.2.}

First, a randomly selected subset (*N = 2,500*) undergoes human labeling, where each tweet is annotated with a status label from the tag set {Suicidal, Non-suicidal}. As depicted in [Figure 1.](#fig1), the dataset displays a significantly imbalanced distribution, with only 2.88\% (*N = 72*) of tweets categorized as suicidal and 97.12\% (*N = 2,428*) as non-suicidal.


<a name="fig1"></a>
{{< chart >}}

type: 'bar',
data: {
  labels: ['Suicidal', 'Non-suicidal'],
  datasets: [{
    backgroundColor: ['rgba(255, 99, 132, 0.2)', 'rgba(75, 192, 192, 0.2)'],
    borderColor: ['rgba(255, 99, 132, 1)', 'rgba(75, 192, 192, 1)'],
    borderWidth: 1,
    barThickness: 100,
    data: [72, 2428],
  }]
},
options: {
  plugins: {
    title: {
      display: true,
      text: 'Figure 1. Suicidal and Non-Suicidal Tweet Distribution in Subset (N = 2500)',
    },
    legend: {
      display: false
    }
  }
}
{{< /chart >}}


To ensure an adequate representation of suicidal instances during the process of model training, the dataset undergoes a series of splits, with stratification based on the label attribute, to create different subsets for training, validation, and testing. 

Initially, the dataset is divided into a training set and a test set, with a split ratio of 85\% for training and 15\% for testing. Subsequently, the training set is further divided into an 85\% training subset and a 15\% validation subset. To this end, the outcome is a 72.25\% training subset (\textit{N} = 1,806), a 12.75\% validation subset (\textit{N} = 319), and a 15\% test subset (\textit{N} = 375). Specifically, the training dataset will serve as the foundation for fine-tuning the model's parameters. The validation set will be used to track the model's performance. Lastly, the test dataset will provide an unbiased estimation of the model's capability to generalize to unseen data.


### 3.3. Pseudolabelling for Hybrid-Augmenting Scare Labels {#3.3.}
Notably, when confronting extreme label imbalances, models often prioritize predicting the majority class to achieve better performance metrics. Given that the primary goal is to identify a higher proportion of suicidal tweets, the significance of minimizing instances of missed suicidal samples (false negatives) outweighs the consequences of incorrectly categorizing non-suicidal samples (false positives). Therefore, such an unbalanced dataset will have an extra negative impact on performance within the specific scope of this study.

In response to the challenge posed by class imbalance, two methods are conducted during the fine-tuning process. The first method involves adjusting class weights within the loss function. In the initial batch, the loss function is modified to assign a weight of 0.1 to the non-suicidal class and a weight of 1.0 to the suicidal class. This adjustment results in misclassifying a suicidal sample holding tenfold more significance in the model's training compared to misclassifying a non-suicidal sample.

After implementing the weight adjustment, the model undergoes validation using a designated validation set and is subsequently saved for the evaluation phase on the test set. Furthermore, according to \textcite{Devlin2018}, larger datasets tend to have reduced sensitivity to hyperparameter choices in comparison to smaller datasets. Therefore, this study refrains from introducing additional adjustments to the hyperparameter configurations. Instead, the focus remains on enhancing the scarce suicidal label with a pseudo-labelling approach that combines prediction outcomes with human interpretation. This approach effectively addresses dataset imbalance and significantly enhances the ability of detecting suicidal tweets. A conceptual algorithmic overview of this process is illustrated in Figure \ref{fig:suibert-pseudolabeling}.


## 4. Calculating Dimensional Emotion Scores
The focus is to analyze two specific emotion dimensions: valence and arousal. By studying these two emotional dimensions, a nuanced insight into the \textit{valence} (positive or negative sentiment) and \textit{arousal} (active or passive stimulation) of emotions expressed in the textual content can be discovered. As stated by \textcite{taboada2011lexicon}, lexicon-based methods for sentiment analysis demonstrate strong robustness, yielding favourable performance across various domains. Based on similar research design principles, this study employs a lexicon-based methodology to delve into the emotional nuances of each tweet.




<div style="display:flex; gap:6px">

{{< badge >}} machine learning {{< /badge >}}

{{< badge >}} social media analysis {{< /badge >}}

{{< badge >}} mental health {{< /badge >}}

{{< badge >}} user engagement {{< /badge >}}

{{< badge >}} project {{< /badge >}}
</div>



[^1]: Devlin, J., Chang, M. W., Lee, K., & Toutanova, K. (2018). Bert: Pre-training of deep bidirectional transformers for language understanding. arXiv preprint arXiv:1810.04805.
