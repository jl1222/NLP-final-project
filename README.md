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

## Collect tweets
#OAuth process, using the keys and tokens
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)
#Creation of the actual interface, using authentication
api = tweepy.API(auth)
text=[]
tweets=tweepy.Cursor(api.user_timeline,id='@realDonaldTrump',
                     tweet_mode='extended').items(2000)
for tweet in tweets:
  if 'RT @' not in tweet.full_text:
    text.append(tweet.full_text) 
