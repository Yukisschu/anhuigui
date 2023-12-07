+++
title = 'SuiBERT: A BERT-Based Model for Suicide Identification'
date = 2023-11-25T17:36:18+01:00
draft = false
slug = 'BERT-suicide-analysis'
tags = ['SuiBERT', 'machine learning', 'social media analysis', 'mental health', 'user engagement', 'project']
+++

{{< lead >}}
Suicide stands as a significant global issue, leading to more than 700,000 lives each year. However, it can be effectively prevented through evidence-based interventions targeting at-risk individuals. Therefore, the accurate identification of suicidal tendencies is pivotal for the development of proactive and efficient suicide prevention strategies. 
<!--
This project focuses on fine-tuning the state-of-the-art deep learning model BERT for the purpose of detecting individuals with suicide tendencies on X (formally known as Twitter). Subsequently, the negative binomial regression model is applied to delve into the intricate relationships among suicidal tendencies, emotional dimensions, and engagement levels.
-->
{{< /lead >}}

## <em>Abstract</em>
Suicide has become a pressing public health concern. However, obtaining related information through traditional methods, such as self-reported questionnaire surveys, is challenging due to the sensitive and complex nature of suicidal thoughts and behaviour. With the development of the Internet, social media platforms are considered powerful tools for identifying and understanding the nuanced emotional and mental details of individuals. 

In this project, 120,000 suicide-related tweets are collected along with engagement data and user details. Following this, SuiBERT, a specialized BERT model for detecting suicidal content, is developed for the specific suicide detection task. Notably, after three fine-tuning batches, SuiBERT demonstrates an impressive accuracy of 90.92% and a recall of 90.40%.

<!--
Afterward, dimensional emotional scores (valence and arousal) are calculated using a lexicon-based approach. Last, the regression analysis is conducted to explore the relationships between emotional dimensions, engagement, and suicidality.Moreover, the outcome of regression analysis unveils a positive correlation between emotional scores and engagement size, while a negative correlation with engagement conversion rate.
-->

## 1. Collecting Suicide-related Disclose on X {#1.}

### 1.1 Data Collection {#1.1.}
To start, public posts on X related to suicide are collected using Python. The keywords utilised for data collection are ***suicide*** and ***suicidal***. The timeframe covers the period from December 2022 to May 2023, with a monthly target of 20,000 tweets. Consequently, a comprehensive dataset comprising 120,000 samples is collected, spanning 1 to 3 days for each month. 

There are 2 main purposes for the selection of this specific timeframe for data collection. Firstly, it effectively captures the metric ***view***, a crucial component of the dependent variable in this study, that was not introduced until December 22, 2022. Additionally, by spanning the entire day and encompassing multiple months, the dataset can capture fluctuating suicide-related tendencies across different time segments. This approach also mitigates potential biases related to seasons or festivals, ensuring the extraction of the most authentic and nuanced information from users.

Apart from the post content, the complete dataset also includes the following attributes: ***UserID, Tweet, Timestamp, Retweets, Likes, Replies, Views, Bookmarks, Location, Followers, Followings, TotalTweets, Hashtags, HasURL***.

### 1.2. Data Cleaning {#1.2.}

The collected raw data from X is further cleaned to ensure the authenticity of the data for future analysis. This involves filtering out low-activity account samples with a total tweet count of less than 30 and a follower count of less than 10, and then missing values are removed. This leads to the size of the dataset being reduced from 120,000 to 105,315. The data cleansing process ensures data accuracy, consistency, and relevance in subsequent analyses.

<!--
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
-->

## 2. Developing Fine-tined SuiBERT for Identifying Suicidal Posts {#2.}

In this section, SuiBERT, a BERT-based fine-tuned model, is developed. Its purpose is to classify the collected suicide-related data into two categories: ***suicidal*** and ***non-suicidal***, providing corresponding probability scores for each classification.

To achieve this objective, the ***bert-base-uncased*** [^1] model, featuring 768 hidden vectors and 12 self-attention heads, is chosen as the baseline model. 

{{< github repo="google-research/bert" >}}

As BERT was pre-trained on expansive yet general datasets, a subsequent fine-tuning stage is essential for the application of a specific downstream task [^1]. At the same time, due to the scarcity of suicidal labels, a semi-supervised pseudo method is employed to fine-tune the pre-trained ***bert-base-uncased*** model to bridge its application to suicidality detection. This method includes a stepwise process of iteratively fine-tuning the model on an extensive pool of unlabeled data with augmented labels.

### 2.1. Initialization of Fine-Tuning Process of SuiBERT {#2.1.}

To initiate the fine-tuning process, the PyTorch framework (*version: 2.0.1+cu118*) and transformers library (*version: 4.30.2*) are deployed. AdamW ({{< katex >}}\\(beta_1 = 0.9, beta_2 = 0.999\\)) is selected as the optimizer. Given the scarcity of suicidal instances, the cross-entropy loss function is employed to impose stricter penalties for misclassifying suicidal cases (label suicidal as non-suicidal).

Furthermore, the recommended range of hyperparameters are[^1]:
- Batch size: 16, 32;
- Learning rate (Adam): 5 × 10<sup>-5</sup>, 3 × 10<sup>-5</sup>, 2 × 10<sup>-5</sup>;
- item Number of epochs: 2, 3, 4.

Considering both hardware capabilities and database scale, the foundational learning rate is configured at 2 × 10<sup>-5</sup>. The training process spans 5 epochs, with a training batch size of 32 and an evaluation batch size of 8. All computations and tasks are executed on the GPU A100 for enhanced performance.

```python
args = TrainingArguments(
    output_dir='temp/',
    evaluation_strategy='epoch',
    save_strategy='epoch',
    learning_rate=2e-5,
    per_device_train_batch_size=32,
    per_device_eval_batch_size=8,
    num_train_epochs=5,
    weight_decay=0.01,
    load_best_model_at_end=True,
    metric_for_best_model='accuracy',
)
```

### 2.2. Fine-Tuning Strategies for Label Imbalance with Loss Function and Dataset Splitting

To start with, a randomly selected subset (*N = 2,500*) undergoes human labeling, where each tweet is annotated with a status label from the tag set {Suicidal, Non-suicidal}. As depicted in [Figure 1.](#fig1), the dataset displays a significantly imbalanced distribution, with only 2.88\% (*N = 72*) of tweets categorized as suicidal but 97.12\% (*N = 2,428*) as non-suicidal.

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

Notably, when confronting extreme label imbalances, models often prioritize predicting the majority class to achieve better performance metrics. Given that the primary goal of this project is to identify more suicidal cases, the significance of minimizing instances of missed suicidal samples (false negatives) outweighs the consequences of incorrectly categorizing non-suicidal samples (false positives).

In response to the challenge posed by class imbalance, two methods are conducted during the fine-tuning process. The first method involves adjusting class weights within the loss function. By assigning a higher weight to the suicidal class, this adjustment results in misclassifying a suicidal sample holding more significance than misclassifying a non-suicidal sample.
```python
class CustomTrainer(Trainer):
    def compute_loss(self, model, inputs, return_outputs=False):
        labels = inputs.get("labels")
        # forward pass
        outputs = model(**inputs)
        logits = outputs.get('logits')
        # compute custom loss
        loss_fct = nn.CrossEntropyLoss(weight=torch.tensor([6.7, 32.9]))
        loss = loss_fct(logits.view(-1, self.model.config.num_labels), labels.view(-1))
        return (loss, outputs) if return_outputs else loss
```
First, 15% of the entire dataset is allocated to the test set, and the remaining 85% is assigned to the training set.  Following this, the training set is subsequently partitioned into training and validation subsets, with an 85% to 15% ratio, stratified by labels.
```python
df_train, df_test, = train_test_split(df, stratify=df['label'], test_size=0.15, random_state=42)
df_train, df_val = train_test_split(df_train, stratify=df_train['label'],test_size=0.15, random_state=42)
print(df_train.shape, df_test.shape, df_val.shape)
```

### 2.3. Hybrid Pseudolabeling Method for Augmenting Scarce Label Samples {#2.3.}

<!--
To ensure an adequate representation of suicidal instances during the process of model training, the dataset undergoes a series of splits, with stratification based on the label attribute, to create different subsets for training, validation, and testing. 

Initially, the dataset is divided into a training set and a test set, with a split ratio of 85\% for training and 15\% for testing. Subsequently, the training set is further divided into an 85\% training subset and a 15\% validation subset. To this end, the outcome is a 72.25\% training subset (\textit{N} = 1,806), a 12.75\% validation subset (\textit{N} = 319), and a 15\% test subset (\textit{N} = 375). Specifically, the training dataset will serve as the foundation for fine-tuning the model's parameters. The validation set will be used to track the model's performance. Lastly, the test dataset will provide an unbiased estimation of the model's capability to generalize to unseen data.
[Figure 3.2.1.](#ref) demonstrates the overall splitting process.
A conceptual algorithmic overview of this process is illustrated in Figure \ref{fig:suibert-pseudolabeling}.
-->
The algorithm for the self-training-centred pseudo-labelling technique employed to categorize tweets as either suicidal or non-suicidal is explained in detail below. First, the fine-tuned model is applied to a new dataset *N = 10,000* with the objective of identifying potentially suicidal tweets. Subsequently, human interpretation is employed to evaluate the model's predictions of suicidal tweets. Among the model's predictions, tweets that are consistently identified as suicidal by both the model and the human interpreter are added to the suicidal label in the training set, thereby expanding the training data. Conversely, instances where the model is classified as suicidal but is considered non-suicidal by the human interpreter, are marked as misclassified samples. These misclassified instances are subsequently added to the non-suicidal label within the training set. This process is undergone iteratively, whereby the model progressively refines its predictive abilities by incorporating augmented positive instances and learning from its misclassifications. The cycle of iteration continues until batch 3.0, by which point the count of instances labelled as suicidal becomes sufficient. At this stage, the ultimate fine-tuned BERT model (SuiBERT) is generated and subsequently employed to analyze the entire dataset to identify tweets that could potentially indicate signs of suicidal tendencies. Among the total of 105,315 samples, 96.55\% *N = 101,685* are classified as non-suicidal, while a mere 3.45\% *N = 3,630* are labelled as suicidal.

## 3. Evaluating the Performance of SuiBERT Model

By incorporating the power of a pre-trained BERT baseline model with targeted fine-tuning for suicide-related context, the SuiBERT model significantly enhances its performance in the specialized task of detecting suicidal content. This section provides a thorough evaluation of the outcomes, accompanied by a detailed analysis.

In the realm of classification tasks, terms like true positives, true negatives, false positives, and false negatives are employed to compare the results of the classifier under test with trusted external judgments. To evaluate performance, widely used metrics include accuracy, recall, precision, and F1 score.

Given the primary objective of this study is to detect more suicidal tweets, the evaluation process focuses both on accuracy and the recall metric. While accuracy provides a comprehensive assessment of the model's performance, a higher importance is given to assessing the model's effectiveness in identifying suicidal tweets among all instances of suicidality.

After interactive fine-tuning processes, the recall values have exhibited a noticeable improvement, rising from 0.750 to 0.904. This validates the model's competence and robustness in classification performance when applied to the entire dataset.


<div style="display:flex; gap:6px">

{{< badge >}} machine learning {{< /badge >}}

{{< badge >}} social media analysis {{< /badge >}}

{{< badge >}} mental health {{< /badge >}}

{{< badge >}} user engagement {{< /badge >}}

{{< badge >}} project {{< /badge >}}
</div>

[^1]: Devlin, J., Chang, M. W., Lee, K., & Toutanova, K. (2018). Bert: Pre-training of deep bidirectional transformers for language understanding. arXiv preprint arXiv:1810.04805.
