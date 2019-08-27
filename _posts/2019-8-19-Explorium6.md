---
published: true
title: Predicting the next hit song (Part 2)
collection: st
layout: single
author_profile: false
read_time: true
categories: [machinelearning]
excerpt : "Applied Data Science"
header :
    overlay_image: "https://maelfabien.github.io/assets/images/wolf.jpg"
    teaser: "https://maelfabien.github.io/assets/images/wolf.jpg"
comments : true
toc: true
search: false
toc_sticky: true
sidebar:
    nav: sidebar-sample
---

<script type="text/javascript" async
    src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

In the previous article, we explored basic models and data enrichments for our hit song classifier. In this article, we will try to push our model a little more, and improve the model performance through better data enrichment and feature engineering. First of all, let's recall the context. 

# The Context

When you decide to produce an artist or invest in a marketing campaign for a song, there are many factors to take into account. But what if data science could help with this task? What if it could help predict whether a song is going to be a hit or not? 

Several articles and papers try to explain why a song became a hit, and the features these songs share. We will try to go a bit further, and build a hit song classifier. To build such a classifier, we typically will need a lot of data enrichment, since there is no single source of data that can help with such a vast task. We will use the following sources to help us build the dataset :
- Google Trends
- Spotify 
- Billboard
- Genius.com

We will consider the following :
- A song is a hit if it reaches the top 10 of the most trending songs of the year
- Otherwise, it's not a hit

Up to now, we have been using data from the Billboard 100 most popular songs between 2010 and 2018 included. We then enriched the data using the Spotify's API. Our model achieved an accuracy of 93% on the test set.

# Data Enrichment through Genius.com

Genius.com is a great resource if you are looking for song lyrics. It offers a great API, all of which is packaged in a great library called `lyricsgenius`. Start by installing the package (instructions can be found on [GitHub](https://github.com/johnwmillr/LyricsGenius)).

You will have to get a token from [Genius.com developer's website](https://docs.genius.com/).

Start by importing the package :

```python
import lyricsgenius as genius
api = genius.Genius('YOUR_TOKEN_GOES_HERE')
```

As before, the API has a powerful search functionality :

```python
def lookup_lyrics(song):
    try :
        return api.search_song(song).lyrics
    except :
        return None
```

And create a column "lyrics" that contains the lyrics of each song. This one might take some time.

```python
df['lyrics'] = df['lookup'].apply(lambda x: lookup_lyrics(x))
```

Notice how some of the text is not clean and contains `\n` to denote a new line, or text between brackets to split sections :

```python
def clean_txt(song):
    song = ' '.join(song.split("\n"))
    song = re.sub("[\[].*?[\]]", "", song)
    return song

df['lyrics'] = df['lyrics'].apply(lambda x: clean_txt(x))
df = df.dropna() #Drop song if we don't have lyrics
```

Some features we could add are :
- the length of the lyrics
- the number of unique words used
- the length of the lyrics without stopwords
- the number of unique words used without stopwords

We will use NLTK stop words list in english, but we should also consider that some of the songs of the Billboard Top 100 Year-End are not english songs.

```python
from nltk.corpus import stopwords 
from nltk.tokenize import word_tokenize 
stop_words = set(stopwords.words('english'))

def len_lyrics(song):
    return len(song.split())

def len_unique_lyrics(song):
    return len(list(set(song.split())))

def rmv_stop_words(song):
    song = [w for w in song.split() if not w in stop_words] 
    return len(song)

def rmv_set_stop_words(song):
    song = [w for w in song.split() if not w in stop_words] 
    return len(list(set(song)))
```

Then, apply this to the dataset :

```python
df['len_lyrics'] = df['lyrics'].apply(lambda x: len_lyrics(x))
df['len_unique_lyrics'] = df['lyrics'].apply(lambda x: len_unique_lyrics(x))
df['without_stop_words'] = df['lyrics'].apply(lambda x: rmv_stop_words(x))
df['unique_without_stop_words'] = df['lyrics'].apply(lambda x: rmv_set_stop_words(x))
```

## Data exploration

Just like in the first article, some data exploration might bring us additional insights.

How many words are used in the lyrics?

```python
plt.figure(figsize=(12,8))
plt.hist(df[df['len_lyrics']<2000]['len_lyrics'], bins=70) #Not plot outliers
plt.title("Number of words")
plt.show()
```

![image](https://maelfabien.github.io/assets/images/expl5_15.png)

The histogram above does not represent outliers, but a few songs count over 2000 words. On average, there are 467 words in a song and 166 unique words. This can be verified by :

```python
np.mean(df['len_lyrics'])
np.mean(df['len_unique_lyrics'])
```

The ratio of unique words over total words is 35%. We can also plot the distribution of this ratio :

```python
plt.figure(figsize=(12,8))
plt.hist(df['len_unique_lyrics']/df['len_lyrics'], bins=50)
plt.title("Ratio Unique Words over total words")
plt.show()
```

![image](https://maelfabien.github.io/assets/images/expl5_20.png)

To illustrate the diversity of the vocabulary used in the songs, we can compute the ratio of words that are not stop words over all words :

![image](https://maelfabien.github.io/assets/images/expl5_21.png)

This is it for the count of words. Now, what are the most common words that singers use in their texts?

```python
from wordcloud import WordCloud, STOPWORDS
word_cloud = df['lyrics'].values

str1 = ' '.join(word_cloud)
stopwords = set(STOPWORDS)

wordcloud = WordCloud(stopwords=stopwords, background_color="white").generate(str(str1))

plt.figure(figsize=(15,8))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```

![image](https://maelfabien.github.io/assets/images/expl5_16.png)

We won't spend too much time commenting that, but Yeah, Oh and Baby should definitely be on your hit-song to-do list.

## Lyrics Sentiment

Should a song be positive? Negative? Neutral? To assess the positiveness of a song and its intensity, we will use Valence Aware Dictionary and sEntiment Reasoner (VADER), a lexicon and rule-based sentiment analysis tool, available on [Github](https://github.com/cjhutto/vaderSentiment). This method relies on lexicons, and has over 7500 words annotated by linguists. This kind of algorithm was used before the rise of Natural Language Processing, but can still be useful in cases like this one where we do not have labeled data or trained models for song sentiment classification.

```python
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

df['sentimentVaderPos'] = df['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['pos'])
df['sentimentVaderNeg'] = df['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['neg'])
df['sentimentVaderComp'] = df['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['compound'])
df['sentimentVaderNeu'] = df['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['neu'])
```

We can also create a feature that is the difference between the positive and the negative score :

```python
df['Vader'] = df['sentimentVaderPos'] - df['sentimentVaderNeg']
```

What are the sentiments expressed in the songs?

```python
plt.figure(figsize=(12,8))
plt.hist(df['Vader'], bins=50)
plt.axvline(0, c='r')
plt.title("Average sentiment")
plt.show()
```

![image](https://maelfabien.github.io/assets/images/expl5_17.png)

On average, the sentiment is slightly positive.

## New model

Let us now train a new model and see whether the performance was improved. First, we create the train and test sets and apply oversampling :

```python
X = df.drop(["Artist_Feat", "Artist", "Artist_Feat_Num", "Title", "Hit", "lookup", "release_date", "genres", "lyrics"], axis=1)
y = df["Hit"]

sm = SMOTE(random_state=42)
X_res, y_res = sm.fit_resample(X, y)
X_train, X_test, y_train, y_test = train_test_split(X_res,y_res, test_size=0.2, random_state=42) 
```

Then, we ddefine the random forest classifier and train the model :

```python
rf=RandomForestClassifier(n_estimators=100)
rf.fit(X_train, y_train)

y_pred = rf.predict(X_test)
accuracy_score(y_pred, y_test)
```

The accuracy score improved by close to 5% and reaches 98.3%.

What are the most important features in this new model?

```python
importances = rf.feature_importances_
indices = np.argsort(importances)

plt.figure(figsize=(12,8))
plt.title('Feature Importances')
plt.barh(range(len(indices)), importances[indices], align='center')
plt.yticks(range(len(indices)), [X.columns[i] for i in indices])
plt.xlabel('Relative Importance')
plt.show()
```

![image](https://maelfabien.github.io/assets/images/expl5_18.png)

The order of the important features remains the same, but the compound sentiment feature is now one of the most important features.

# Making predictions 

## Prediction function

We can build a predictor that takes as an input the name of the song and the singer, creates the features, and output the probability of being a hit. Since the algorithm has never been trained on 2019 songs, we can feed it with recent songs and observe the outcome. 

We can recall the whole pipeline first :

![image](https://maelfabien.github.io/assets/images/expl5_22.png)

Let's build this pipeline and try it with "Lover" by Taylor Swift, a song that was recently published when we wrote the article:


```python
def model_prediction(artist, title):
    
    df_pred = pd.DataFrame.from_dict({
        "Artist":[artist], 
        "Title":[title]})
        
    df_pred["Featuring"] = df_pred.apply(lambda row: featuring(row['Artist']), axis=1)
    df_pred["Artist_Feat"] = df_pred.apply(lambda row: featuring_substring(row['Artist']), axis=1)
    df_pred['Title_Length'] = df_pred['Title'].apply(lambda x: num_words(x))
    df_pred['lookup'] = df_pred['Title'] + " " + df_pred["Artist_Feat"]
    df_pred['available_markets'], df_pred['release_date'], df_pred['total_followers'],
    df_pred['genres'], df_pred['popularity'], df_pred['acousticness'], df_pred['danceability'],
    df_pred['duration_ms'], df_pred['energy'], df_pred['instrumentalness'], df_pred['key'],
    df_pred['liveness'], df_pred['loudness'], df_pred['speechiness'], df_pred['tempo'],
    df_pred['time_signature'], df_pred['valence'] = zip(*df_pred['lookup'].map(artist_info))
    df_pred['release_date'] = pd.to_datetime(df_pred['release_date'])
    df_pred['month_release'] = df_pred['release_date'].apply(lambda x: x.month)
    df_pred['day_release'] = df_pred['release_date'].apply(lambda x: x.day)
    df_pred['weekday_release'] = df_pred['release_date'].apply(lambda x: x.weekday())
    df_pred['lookup'] = df_pred['Title'] + " " + df_pred["Artist"]
    df_pred['lyrics'] = df_pred['lookup'].apply(lambda x: lookup_lyrics(x))
    df_pred['lyrics'] = df_pred['lyrics'].apply(lambda x: clean_txt(x))
    df_pred['len_lyrics'] = df_pred['lyrics'].apply(lambda x: len_lyrics(x))
    df_pred['len_unique_lyrics'] = df_pred['lyrics'].apply(lambda x: len_unique_lyrics(x))
    df_pred['without_stop_words'] = df_pred['lyrics'].apply(lambda x: rmv_stop_words(x))
    df_pred['unique_without_stop_words'] = df_pred['lyrics'].apply(lambda x: rmv_set_stop_words(x))
    df_pred['sentimentVaderPos'] = df_pred['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['pos'])
    df_pred['sentimentVaderNeg'] = df_pred['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['neg'])
    df_pred['sentimentVaderComp'] = df_pred['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['compound'])
    df_pred['sentimentVaderNeu'] = df_pred['lyrics'].apply(lambda x: analyzer.polarity_scores(x)['neu'])
    df_pred['Vader'] = df_pred['sentimentVaderPos'] - df_pred['sentimentVaderNeg']

    X = df_pred.drop(["Artist_Feat", "Artist", "Title", "lookup", "release_date", "genres", "lyrics"], axis=1).astype(float)
    y_pred = rf.predict_proba(X)
    
    print("It's a NOT hit with probability : " + str(y_pred[0][0]))
    print("It's a hit with probability : " + str(y_pred[0][1]))
    
    return y_pred
```

We can create an interactive form in the Notebook to ask the user for the name of the artist and title of the song, and output the prediction.

```python
from ipywidgets import widgets, interact

artist = widgets.Text()
title = widgets.Text()

ui = widgets.HBox([artist, title])

def f(artist, title):
    return model_prediction(artist, title)
```

And in the next cell, type :

```python
interact(f, artist='Taylor Swift', title='Lover')
```

![image](https://maelfabien.github.io/assets/images/expl5_19.png)

According to our algorithm, there are only 22% chances that the song "Lover" by TaylorSwift will make it to the top 10 of the most popular songs of 2019. 

# Conclusion

Through this article, we illustrated the importance of external data sources for most data science problems. A good enrichment can boost the performance of your model, and relevant feature engineering can help gain additional performance. 

Here is a performance summary of the different steps of our model :

| Description | Model | Performance |
| -- | -- | -- |
| Data from billboard | Decision Tree | F1-Score : 6.6% |
| Enrich with Spotify and oversample | Random Forest | Accuracy : 93% |
| Enrich with Genius | Random Forest | Accuracy : 98% |

Sources and resources:
- [Lyricsgenius](https://github.com/johnwmillr/LyricsGenius)
