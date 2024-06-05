+++
title = 'Using BERTopic for Topic Extraction in Tweets'
date = 2023-11-25T16:00:00+01:00
tags = ['WIP', 'NLP', 'BERT', 'topic extraction', 'machine learning', 'project']
draft = true
+++

{{< lead >}}
Suicide stands as a pressing global concern, underscoring the imperative for proactive detection and robust protective
measures. In pursuit of ground-based countermeasures, this project delves deeply into suicide-related disclosure on
platform X. It employs comprehensive pattern analyses and topic extraction to uncover distinctive patterns within both
suicidal and non-suicidal groups.
{{< /lead >}}

## BERTopic
BERTopic is an open-source library for topic extraction based on the BERT NLP model.
To Learn More about BERTopic, please check out the Github repo:
{{< github repo="MaartenGr/BERTopic" >}}

## The Code

### Imports
{{< highlight Python >}}
import pandas as pd
from bertopic import BERTopic
from sentence_transformers import SentenceTransformer, util
from umap import UMAP
from hdbscan import HDBSCAN
from sklearn.feature_extraction.text import CountVectorizer
from bertopic.vectorizers import ClassTfidfTransformer
from bertopic.representation import KeyBERTInspired
from bertopic.representation import MaximalMarginalRelevance
from sentence_transformers import SentenceTransformer
import numpy as np
import re
{{< /highlight >}}

### Extracting Topics
{{< highlight Python >}}
df = pd.read_csv("cleaned-tweets.csv",low_memory=True)

suicidal = df.loc[df['Prediction_label'] == 'LABEL_1']
non_suicidal = df.loc[df['Prediction_label'] == 'LABEL_0'].sample(
    n=len(suicidal),
    random_state=42)

docus = pd.concat([suicidal, non_suicidal], axis=0)

sentence_model = SentenceTransformer("all-MiniLM-L6-v2")
representation_model = MaximalMarginalRelevance(diversity=0.4)
umap_model = UMAP(
    n_neighbors=15,
    n_components=5,
    min_dist=0.0,
    metric='cosine')
hdbscan_model = HDBSCAN(
    min_cluster_size=15,
    metric='euclidean',
    cluster_selection_method='eom',
    prediction_data=True)
vectorizer_model = CountVectorizer(stop_words="english")
ctfidf_model = ClassTfidfTransformer(reduce_frequent_words=True)

topic_model = BERTopic(
    representation_model=representation_model,
    embedding_model=sentence_model,   
    umap_model=umap_model,              
    hdbscan_model=hdbscan_model,        
    vectorizer_model=vectorizer_model,  
    ctfidf_model=ctfidf_model,          
    calculate_probabilities=True,
    verbose=True)

topics, probabilities = topic_model.fit_transform(docus['Tweet'])
{{< /highlight >}}


### Extracted Topics
{{< highlight Python >}}
topic_model.get_topic_info()
{{< /highlight >}}


| Topic | Count | Name                                    | Representation                                    | Representative_Docs                               |
|-------|-------|-----------------------------------------|---------------------------------------------------|---------------------------------------------------|
| -1    | 3510  | -1_did_committing_life_child            | [did, committing, life, child, die, talk, idea... | [idk but being a suicidal on the last day of t... |
| 0     | 262   | 0_smock_sawyermerritt_pauly0x_hange_zzz | [smock, sawyermerritt, pauly0x, hange_zzz, sui... | [@EldritchCatCult @Ceododudismo @RulerOfHumani... |
| 1     | 216   | 1_trans_transgender_affirming_lgbtq     | [trans, transgender, affirming, lgbtq, youth, ... | [Transgender people have higher rates of depre... |
| 2     | 180   | 2_amusement_bitch_toughts_ik            | [amusement, bitch, toughts, ik, tesla, tryna, ... | [Its like if i dont feel suicidal within a wee... |
| 3     | 126   | 3_depressed_stress_future_feels         | [depressed, stress, future, feels, el, dental,... | [major dental issues make me sorta suicidal-an... |
| ...   | ...   | ...                                     | ...                                               |
| 61    | 17    | 61_voice_france_mohammad_demand         | [voice, france, mohammad, demand, iranrevoluti... | [@aftaabgardun The great man Mohammad Moradi c... |
| 62    | 17    | 62_princeton_campus_professors_hazing   | [princeton, campus, professors, hazing, prosec... | [The Princeton University student who was init... |
| 63    | 16    | 63_coal_electricity_co2_minnesota       | [coal, electricity, co2, minnesota, bans, econ... | [@Aetrion @KevinCuddeback @Bpaulik777 @tedcruz... |
| 64    | 15    | 64_wins_addicted_dyin_dxlrjxv3p4        | [wins, addicted, dyin, dxlrjxv3p4, ketsele3000... | [@ketsele3000 Please hurt me. I need that way ... |
| 65    | 15    | 65_pray_emilovestwit_ldgwyv03r6_sneaks  | [pray, emilovestwit, ldgwyv03r6, sneaks, reads... | [If my suicide attempts had worked, nobody wou... |

### Appending Topic Info to Tweet Input Data

{{< highlight Python >}}
topic_info = topic_model.get_document_info(docus['Tweet'])
topic_appended = pd.concat([topic_info,docus],axis=1)
topic_appended.drop(['Document'], axis=1, inplace=True)
topic_appended.to_csv("topic-output.csv")
{{< /highlight >}}

## Analysis of the Extracted Topics
{{< typeit
tag=h4
speed=50
lifelike=true
breakLines=false
loop=true
>}}
LGBTQ
Entertainment
Violence
Politics
News
Psychological_support
Woman_issues
Religion
Emotion_expression
Unemployment
Suicide_prevention
Social_media
Education
Seasons
Environment
Diet
Alcohol
Financial
Insomnia
Family
{{< /typeit >}}

{{< article link="/anhuigui/projects/suibert/" >}}

{{< chart >}}
type: 'bar',
data: {
labels: ['LGBTQ', 'Entertainment', 'Violence', 'Politics', 'News', 'Psychological_support', 'Woman_issues', 'Religion', 'Emotion_expression', 'Unemployment', 'Suicide_prevention', 'Social_media', 'Education', 'Seasons', 'Environment', 'Diet', 'Alcohol', 'Financial', 'Insomnia', 'Family'],
datasets: [
{
label: 'Non-suicidal',
backgroundColor: 'rgba(75, 192, 192, 0.2)',
borderColor: 'rgba(75, 192, 192, 1)',
borderWidth: 1,
data: [204, 203, 188, 180, 137, 79, 55, 47, 45, 41, 34, 33, 31, 23, 19, 9, 5, 4, 1, 1],
},
{
label: 'Suicidal',
backgroundColor: 'rgba(255, 99, 132, 0.2)',
borderColor: 'rgba(255, 99, 132, 1)',
borderWidth: 1,
data: [3, 216, 5, 3, 17, 90, 47, 12, 299, 72, 11, 69, 0, 131, 0, 89, 33, 17, 84, 34],
},
],
}
{{< /chart >}}
