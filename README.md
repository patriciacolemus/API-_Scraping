## **Twitter API scraping**

Twitter is a popular social network where users share messages called tweets. Twitter allows us to mine the data of any user using Twitter API.

In order to analyze the discussion on Twitter about vaccines, **6,000 tweets** in Spanish were collected, which contain at least one term from a set of keywords, such as “vaccine”, “vaccination”, “vaccinate”. Only tweets were collected, so retweets were discarded.

Before starting the search, it was necessary to sign up for a developer account [developer account on Twitter] (https://developer.twitter.com/)

### Libraries
The following libraries help with this activity: 
`tweepy`
`json`
`pandas`
`numpy`
`csv`

#### Authorization 
After having a developer account, and saving your App's key and tokens and keeping them secure, it is important to authenticate: 

```
auth = tweepy.OAuthHandler(credentials.API_KEY, credentials.API_SECRET_KEY)
api = tweepy.API(auth)
```

#### Request
Search parameters: 
- q = words, hashtags or users you want to search
- lang = the default language is English, but it can be changed. 
- result_type = it refers to the type of tweets you want to search. The available options are: 
   - - mixed : Include both popular and real time results in the response.
   - - recent : return only the most recent results in the response
   - popular : return only the most popular results in the response.
- max_id = allows every search to have different entries

```
id= None 
tweets = []
for tweet in tweepy.Cursor(api.search_tweets,
                       q='vacuna OR vacunas OR vacunación OR vacunar' +  " -filter:retweets",
                       lang="es",
                       result_type='mixed',   
                       max_id=id).items(6000): 
    tweets.append(tweet)
```

*It's importat to keep in mind there is a limit of 500K Tweets/month search request.*

#### Working with a dataframe ### 
Twitter search query returns data in JSON. Because it can be easier to work with a dataframe, `pandas` was used  for converting simple flattened JSON into a DataFrame.

```
data = [[tweet.user.id_str, tweet.user.screen_name, tweet.text, tweet.id_str, tweet.created_at, tweet.retweet_count,
         tweet.favorite_count, tweet.user.description,tweet.user.verified, tweet.user.followers_count, 
         tweet.user.friends_count, tweet.entities['hashtags'], tweet.entities['user_mentions'], 
         tweet.entities['urls']] for tweet in tweets]

```
##### Variables 
Here are the list of variables we can find while working with Twitter API: 

- *created_at* : The time the status was posted.
- *id_str* : The ID of the status as a string.
- *text* : The text of the status.
- *entities* : The parsed entities of the status such as hashtags, URLs etc.
- *source* : The source of the status.
- *source_url* : The URL of the source of the status.
- *in_reply_to_status_id* : The ID of the status being replied to.
- *in_reply_to_status_id_str* : The ID of the status being replied to in as a string.
- *in_reply_to_user_id* : The ID of the user being replied to.
- *in_reply_to_user_id_str* : The ID of the user being replied to as a string.
- *in_reply_to_screen_name* : The screen name of the user being replied to
- *user* : The User object of the poster of the status.
- *geo* : The geo object of the status.
- *coordinates* : The coordinates of the status.
- *place* : The place of the status.
- *contributors* : The contributors of the status.
- *is_quote_status* : Indicates whether the status is a quoted status or not.
- *retweet_count* : The number of retweets of the status.
- *favorite_count* : The number of likes of the status.
- *favorited* : Indicates whether the status has been favourited by the authenticated user or not.
- *retweeted* : Indicates whether the status has been retweeted by the authenticated user or not.
- *possibly_sensitive* : Indicates whether the status is sensitive or not.
- *lang* : The language of the status.

#### Data cleaning
All empty rows were replace with 0 value. 

```
null_cols[null_cols > 0]
drop_cols = list(null_cols[null_cols > 100].index)
df = df.drop(drop_cols, axis=1)
```

Then I identify the users who talk the most about the vaccine and tagged them 

```
# Unique users
uniqueUsers = df.user_id.nunique() 
duplicateUsers = df['user_id'].duplicated().sum() 
print('Number of duplicate users:', duplicateUsers, 'from:', uniqueUsers)

# Add a new variable to identify unique or duplicate users
conteos = df.user_id.value_counts()
conteos[df.user_id]
df['User_tag'] = np.where(conteos[df.user_id] == 1, 'Unique','Duplicate')
```

After this, I identified the users with the most followers 
```
pd.cut(df.followers, 5)
tag = ['Very popular', 'Popular', 'Few followers', 'Unpopular', 'No followers'] 
pd.cut(df.followers, 5, labels = tag) 
pd.cut(df.followers, 5, labels = tag).value_counts()

# Add a new variable to identify the followers range of the users
df['Popularity'] = pd.cut(df.followers, 5, labels=tag)
```
