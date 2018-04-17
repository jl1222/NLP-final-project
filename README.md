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
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Unigram%20cloud.png)

```{r}
########### Bigram TF word cloud ###########

tokenizer2 <- function(x){NGramTokenizer(x,Weka_control(min=2,max=2))}
second <- TermDocumentMatrix(cleaned_corpus,control = list(tokenize=tokenizer2))
second <- as.matrix(second)
second[1:2,1:5]
second_frequency <- rowSums(second)
second_frequency <- sort(second_frequency,dec=TRUE)
second_word_freqs <- data.frame(term = names(second_frequency), num = second_frequency)
wordcloud(second_word_freqs$term, second_word_freqs$num,min.freq=5,max.words=500,colors=brewer.pal(8, "Paired"))
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Bigram%20cloud.png)

```{r}
########### Trigram TF word cloud ###########

tokenizer3 <- function(x){NGramTokenizer(x,Weka_control(min=3,max=3))}
third <- TermDocumentMatrix(cleaned_corpus,control = list(tokenize=tokenizer3))
third <- as.matrix(third)
third[1:2,1:5]
third_frequency <- rowSums(third)
third_frequency <- sort(third_frequency,dec=TRUE)
third_word_freqs <- data.frame(term = names(third_frequency), num = third_frequency)
wordcloud(third_word_freqs$term, third_word_freqs$num,min.freq=5,max.words=500,colors=brewer.pal(8, "Paired"))
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Trigram%20cloud.png)

## TF-IDF word cloud 
```{r}
tfidf_tdm <- TermDocumentMatrix(cleaned_corpus,control=list(weighting=weightTfIdf))
tfidf_tdm_m <- as.matrix(tfidf_tdm)

# Term Frequency
term_frequency <- rowSums(tfidf_tdm_m)
# Sort term_frequency in descending order
term_frequency <- sort(term_frequency,dec=TRUE)
# Create word_freqs
word_freqs <- data.frame(term = names(term_frequency), num = term_frequency)
# Create a wordcloud for the values in word_freqs
wordcloud(word_freqs$term, word_freqs$num,min.freq=5,max.words=1000,colors=brewer.pal(8, "Paired"))
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/TF-IDF%20word%20cloud.png)

## Sentiment analysis (bing lexicon) 
```{r}
#Tidy TDM
TDM_text <- TermDocumentMatrix(cleaned_corpus)
text_tidy <- tidy(TDM_text)
# bing
bing_lex <- get_sentiments("bing")
text_bing_lex <- inner_join(text_tidy, bing_lex, by = c("term" = "word"))
text_bing_lex$sentiment_n <- ifelse(text_bing_lex$sentiment=="negative", -1, 1)
text_bing_lex$sentiment_value <- text_bing_lex$sentiment_n * text_bing_lex$count
text_bing_lex
bing_aggdata <- aggregate(text_bing_lex$sentiment_value, list(index = text_bing_lex$document), sum)
bing_aggdata
sapply(bing_aggdata,typeof)
bing_aggdata$index <- as.numeric(bing_aggdata$index)
ggplot(bing_aggdata, aes(index, x)) + geom_point()
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Bing_points.png)
```{r}
ggplot(bing_aggdata, aes(index, x)) + geom_smooth() 
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Bing_smooth.png)

## Comparison word cloud on sentiment
```{r}
library(reshape2)
text_tidy %>%
  inner_join(get_sentiments("bing"), by = c("term" = "word")) %>%
  count(term, sentiment, sort=TRUE) %>%
  acast(term ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors=brewer.pal(8, "Dark2"),
                   max.words=200)
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Comparison%20word%20cloud%20on%20sentiment.png)

## Emotional analysis
```{r}
library(radarchart)
nrc_lex <- get_sentiments("nrc")
nrc <- inner_join(text_tidy, nrc_lex, by = c("term" = "word"))
nrc_noposneg <- nrc[!(nrc$sentiment %in% c("positive","negative")),]
aggdata <- aggregate(nrc_noposneg$count, list(index = nrc_noposneg$sentiment), sum)
chartJSRadar(aggdata)
```
![image](https://github.com/jl1222/NLP-final-project/blob/master/images/Emotional%20analysis.png)
