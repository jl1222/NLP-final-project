# NLP-final-project

1. Twitter feed text analysis (minimum 2000 tweets)/Other data sources

2. TF word clouds
Unigram
Bigram
Trigram

3. TF-IDF word cloud

4. Sentiment analysis (any one lexicon)

5. Comparison/Contrast word clouds based on sentiment

6. Emotional analysis (any one lexicon)

## Collect tweets (by Python)
```{python}
#OAuth process, using the keys and tokens
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)

#Creation of the actual interface, using authentication
api = tweepy.API(auth)

#Collect tweets
text=[]
tweets=tweepy.Cursor(api.user_timeline,id='@realDonaldTrump',
                     tweet_mode='extended').items(2000)
for tweet in tweets:
  if 'RT @' not in tweet.full_text:
    text.append(tweet.full_text) 
```

## Clean corpus (by R)
```{r}
text <- read.csv('TrumpTwitter.csv')

#making a corpus of a vector source
corpus <- VCorpus(VectorSource(text$X0))

#Cleaning corpus - pre_processing
clean_corpus <- function(cleaned_corpus){
  removeURL <- content_transformer(function(x) gsub("(f|ht)tp(s?)://\\S+", "", x, perl=T))
  cleaned_corpus <- tm_map(cleaned_corpus, removeURL)
  cleaned_corpus <- tm_map(cleaned_corpus, content_transformer(replace_abbreviation))
  cleaned_corpus <- tm_map(cleaned_corpus, content_transformer(tolower))
  cleaned_corpus <- tm_map(cleaned_corpus, removePunctuation)
  cleaned_corpus <- tm_map(cleaned_corpus, removeNumbers)
  cleaned_corpus <- tm_map(cleaned_corpus, removeWords, stopwords("english"))
  cleaned_corpus <- tm_map(cleaned_corpus, stripWhitespace)
  return(cleaned_corpus)
}
cleaned_corpus <- clean_corpus(corpus)
```

## TF word clouds (by R)
```{r}
library(RWeka)
library(wordcloud)

########### Unigram TF word cloud ###########
first <- TermDocumentMatrix(cleaned_corpus)
first <- as.matrix(first)

# Term Frequency
first_frequency <- rowSums(first)
# Sort term_frequency in descending order
first_frequency <- sort(first_frequency,dec=TRUE)
# Create word_freqs
first_word_freqs <- data.frame(term = names(first_frequency), num = first_frequency)
# Create a wordcloud for the values in word_freqs
wordcloud(first_word_freqs$term, first_word_freqs$num,min.freq=5,max.words=500,colors=brewer.pal(8, "Paired"))
```

![image](https://github.com/jl1222/NLP-final-project/blob/master/Unigram%20cloud.png)
