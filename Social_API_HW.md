

```
# Dependencies
import tweepy
import json
import pandas as pd

# Twitter API Keys
consumer_key = "2cyrdUbW1HvQlhWc4k4m0Zf95"
consumer_secret = "G6qlHuggipjMlhFRcLOKM2ecZsHaJJaW0M7fzzR5c7mxJOoAo8"
access_token = "37848462-1chMwv1dYyI3a9peTFRC3K2tte9IkLTCHVoqlhyPb"
access_token_secret = "duYVHKnx1rRQbtzpqbml9eKjOSIADZnJqMCbD1e3pzqLI"

# Import and Initialize Sentiment Analyzer
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

# Setup Tweepy API Authentication
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())
```


```
import numpy as np
# Target Search Term
target_terms = ("@BBCWORLD","@nytimes","@CNN","@FOXNEWS","@CBSNEWS")

# Array to hold sentiment
sentiment_array = []

# Variable for holding the oldest tweet
oldest_tweet = ""

sentiment_tot = []
tot_data = []

# Loop through all target users
for target in target_terms:

    # Variables for holding sentiments
    compound_list = []
    positive_list = []
    negative_list = []
    neutral_list = []
    
    # Counter
    counter = 1

    # Loop through 1 times (total of 100 tweets)
    for x in range(1):

        # Run search around each tweet
        public_tweets = api.search(target, count=100, result_type="recent")

        # Loop through all tweets
        for tweet in public_tweets["statuses"]:


                # Run Vader Analysis on each tweet
                compound = analyzer.polarity_scores(tweet["text"])["compound"]
                pos = analyzer.polarity_scores(tweet["text"])["pos"]
                neu = analyzer.polarity_scores(tweet["text"])["neu"]
                neg = analyzer.polarity_scores(tweet["text"])["neg"]

                # Add each value to the appropriate array
                compound_list.append(compound)
                positive_list.append(pos)
                negative_list.append(neg)
                neutral_list.append(neu)
                tweets_ago = counter
                
                total_tweet = {"User": target,
                     "Compound": compound,
                     "Positive": pos,
                     "Neutral": neg,
                     "Negative": neu,
                     "tweets_ago": counter,
                     "Date": tweet["created_at"],
                     "tweet": tweet['text']} 
                
                tot_data.append(total_tweet)
                
                
                counter = counter + 1
                
    # Store the Average Sentiments
    sentiment = {"User": target,
                 "Compound": np.mean(compound_list),
                 "Positive": np.mean(positive_list),
                 "Neutral": np.mean(negative_list),
                 "Negative": np.mean(neutral_list),
                 "Tweet Count": len(compound_list)}
    # Store the Average Sentiments
    sentiment_tot.append(sentiment)
    
    # Print the Sentiments
    print(sentiment)
    print("")
```

    {'User': '@BBCWORLD', 'Compound': -0.18163699999999999, 'Positive': 0.047119999999999995, 'Neutral': 0.13673000000000002, 'Negative': 0.81619000000000019, 'Tweet Count': 100}
    
    {'User': '@nytimes', 'Compound': 0.039362999999999995, 'Positive': 0.069059999999999996, 'Neutral': 0.048739999999999999, 'Negative': 0.88218999999999992, 'Tweet Count': 100}
    
    {'User': '@CNN', 'Compound': -0.00016799999999999649, 'Positive': 0.083190000000000014, 'Neutral': 0.080340000000000009, 'Negative': 0.83645000000000014, 'Tweet Count': 100}
    
    {'User': '@FOXNEWS', 'Compound': 0.047153999999999988, 'Positive': 0.097049999999999984, 'Neutral': 0.067510000000000001, 'Negative': 0.83540999999999987, 'Tweet Count': 100}
    
    {'User': '@CBSNEWS', 'Compound': 0.023787000000000006, 'Positive': 0.094320000000000001, 'Neutral': 0.088650000000000007, 'Negative': 0.81706000000000001, 'Tweet Count': 100}
    



```
#Create DF from total sentiment and export to CSV
summary_df = pd.DataFrame.from_dict(sentiment_tot)
summary_df
summary_df.to_csv("Total_Sentiment_media.csv", encoding="utf-8", index=False)
summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>Positive</th>
      <th>Tweet Count</th>
      <th>User</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.181637</td>
      <td>0.81619</td>
      <td>0.13673</td>
      <td>0.04712</td>
      <td>100</td>
      <td>@BBCWORLD</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.039363</td>
      <td>0.88219</td>
      <td>0.04874</td>
      <td>0.06906</td>
      <td>100</td>
      <td>@nytimes</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.000168</td>
      <td>0.83645</td>
      <td>0.08034</td>
      <td>0.08319</td>
      <td>100</td>
      <td>@CNN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.047154</td>
      <td>0.83541</td>
      <td>0.06751</td>
      <td>0.09705</td>
      <td>100</td>
      <td>@FOXNEWS</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.023787</td>
      <td>0.81706</td>
      <td>0.08865</td>
      <td>0.09432</td>
      <td>100</td>
      <td>@CBSNEWS</td>
    </tr>
  </tbody>
</table>
</div>




```
#Create DF from sentiment per tweet and export to CSV
Full_tweet_df = pd.DataFrame.from_dict(tot_data)
Full_tweet_df.to_csv("Sentiment_per_tweet_media.csv", encoding="utf-8", index=False)
Full_tweet_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>Positive</th>
      <th>User</th>
      <th>tweet</th>
      <th>tweets_ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.6166</td>
      <td>Wed Mar 07 03:10:27 +0000 2018</td>
      <td>0.705</td>
      <td>0.000</td>
      <td>0.295</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Russia MP: 'I don't feel people ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:10:06 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Making boots from hippopotamus a...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:09:46 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Australia and East Timor sign hi...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:09:34 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Trump tariffs: President says EU...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.4767</td>
      <td>Wed Mar 07 03:09:21 +0000 2018</td>
      <td>0.780</td>
      <td>0.220</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: 'Haunted by nightmares' of worki...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:09:11 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>@captainNaCl89 @BBCWorld Since 2011 plehboi 😂</td>
      <td>6</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-0.8860</td>
      <td>Wed Mar 07 03:08:51 +0000 2018</td>
      <td>0.514</td>
      <td>0.486</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Pepe the Frog 'is killed off to ...</td>
      <td>7</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:08:49 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:08:23 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>@Howboutnahb @BBCWorld Obama is probably an at...</td>
      <td>9</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-0.2960</td>
      <td>Wed Mar 07 03:08:19 +0000 2018</td>
      <td>0.720</td>
      <td>0.184</td>
      <td>0.096</td>
      <td>@BBCWORLD</td>
      <td>@BBCWorld England giving a Russian spy a home ...</td>
      <td>10</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:08:15 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Rex Tillerson slams China's rela...</td>
      <td>11</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:08:13 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: A party-goer stole Frances McDor...</td>
      <td>12</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:08:11 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>@ak_bais @SecNielsen @CNN @FoxNews @WhiteHouse...</td>
      <td>13</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.3382</td>
      <td>Wed Mar 07 03:08:07 +0000 2018</td>
      <td>0.745</td>
      <td>0.103</td>
      <td>0.152</td>
      <td>@BBCWORLD</td>
      <td>@BBCBreaking @BBCWorld All this winning is mak...</td>
      <td>14</td>
    </tr>
    <tr>
      <th>14</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:08:04 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>15</td>
    </tr>
    <tr>
      <th>15</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:08:03 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>16</td>
    </tr>
    <tr>
      <th>16</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:07:58 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Oscars 2018: The Shape of Water ...</td>
      <td>17</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:07:52 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>18</td>
    </tr>
    <tr>
      <th>18</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:07:17 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>19</td>
    </tr>
    <tr>
      <th>19</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:06:57 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>#LokpalAppointment.Delay will defame Union gov...</td>
      <td>20</td>
    </tr>
    <tr>
      <th>20</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:06:28 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>21</td>
    </tr>
    <tr>
      <th>21</th>
      <td>0.7177</td>
      <td>Wed Mar 07 03:06:20 +0000 2018</td>
      <td>0.667</td>
      <td>0.000</td>
      <td>0.333</td>
      <td>@BBCWORLD</td>
      <td>@BBCWorld Thanks for sharing!  The world needs...</td>
      <td>22</td>
    </tr>
    <tr>
      <th>22</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:06:19 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Australia and East Timor sign hi...</td>
      <td>23</td>
    </tr>
    <tr>
      <th>23</th>
      <td>-0.5385</td>
      <td>Wed Mar 07 03:06:03 +0000 2018</td>
      <td>0.600</td>
      <td>0.274</td>
      <td>0.126</td>
      <td>@BBCWORLD</td>
      <td>@BBCWorld That's happening as well here in Bor...</td>
      <td>24</td>
    </tr>
    <tr>
      <th>24</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:05:45 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>25</td>
    </tr>
    <tr>
      <th>25</th>
      <td>-0.8860</td>
      <td>Wed Mar 07 03:05:32 +0000 2018</td>
      <td>0.514</td>
      <td>0.486</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Pepe the Frog 'is killed off to ...</td>
      <td>26</td>
    </tr>
    <tr>
      <th>26</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:05:30 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>@BBCWorld 😮WOW thats faster than using the int...</td>
      <td>27</td>
    </tr>
    <tr>
      <th>27</th>
      <td>-0.4215</td>
      <td>Wed Mar 07 03:05:28 +0000 2018</td>
      <td>0.811</td>
      <td>0.189</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: Diver swims through thick soup o...</td>
      <td>28</td>
    </tr>
    <tr>
      <th>28</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:05:25 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: The girl who sabotaged her own w...</td>
      <td>29</td>
    </tr>
    <tr>
      <th>29</th>
      <td>-0.6908</td>
      <td>Wed Mar 07 03:05:24 +0000 2018</td>
      <td>0.749</td>
      <td>0.251</td>
      <td>0.000</td>
      <td>@BBCWORLD</td>
      <td>RT @BBCWorld: North Korea used VX nerve agent ...</td>
      <td>30</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>470</th>
      <td>-0.5859</td>
      <td>Wed Mar 07 03:02:21 +0000 2018</td>
      <td>0.798</td>
      <td>0.202</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews People are disgusting on this TL. Boo...</td>
      <td>71</td>
    </tr>
    <tr>
      <th>471</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:02:06 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: CBS News investigates the mining ...</td>
      <td>72</td>
    </tr>
    <tr>
      <th>472</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:02:00 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>https://t.co/k45D7MNUyO  @SenSherrodBrown @Rep...</td>
      <td>73</td>
    </tr>
    <tr>
      <th>473</th>
      <td>0.3182</td>
      <td>Wed Mar 07 03:01:54 +0000 2018</td>
      <td>0.779</td>
      <td>0.082</td>
      <td>0.139</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Adult film performer Stormy Danie...</td>
      <td>74</td>
    </tr>
    <tr>
      <th>474</th>
      <td>-0.4767</td>
      <td>Wed Mar 07 03:01:46 +0000 2018</td>
      <td>0.492</td>
      <td>0.508</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews I'm crying. 😂#StBernardPuppy</td>
      <td>75</td>
    </tr>
    <tr>
      <th>475</th>
      <td>-0.4019</td>
      <td>Wed Mar 07 03:01:43 +0000 2018</td>
      <td>0.701</td>
      <td>0.199</td>
      <td>0.100</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Surveillance video shows a woman ...</td>
      <td>76</td>
    </tr>
    <tr>
      <th>476</th>
      <td>-0.4019</td>
      <td>Wed Mar 07 03:01:40 +0000 2018</td>
      <td>0.701</td>
      <td>0.199</td>
      <td>0.100</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Surveillance video shows a woman ...</td>
      <td>77</td>
    </tr>
    <tr>
      <th>477</th>
      <td>0.2732</td>
      <td>Wed Mar 07 03:01:37 +0000 2018</td>
      <td>0.909</td>
      <td>0.000</td>
      <td>0.091</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: @CBSNews In October 2016, she acc...</td>
      <td>78</td>
    </tr>
    <tr>
      <th>478</th>
      <td>-0.1027</td>
      <td>Wed Mar 07 03:01:26 +0000 2018</td>
      <td>0.943</td>
      <td>0.057</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: "I find it hard to believe." Two ...</td>
      <td>79</td>
    </tr>
    <tr>
      <th>479</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:01:24 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: @CBSNews Michael Avenatti, the at...</td>
      <td>80</td>
    </tr>
    <tr>
      <th>480</th>
      <td>0.9402</td>
      <td>Wed Mar 07 03:01:09 +0000 2018</td>
      <td>0.377</td>
      <td>0.100</td>
      <td>0.523</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews Yea! Another fury friend received hel...</td>
      <td>81</td>
    </tr>
    <tr>
      <th>481</th>
      <td>0.2732</td>
      <td>Wed Mar 07 03:01:05 +0000 2018</td>
      <td>0.909</td>
      <td>0.000</td>
      <td>0.091</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: @CBSNews In October 2016, she acc...</td>
      <td>82</td>
    </tr>
    <tr>
      <th>482</th>
      <td>0.1027</td>
      <td>Wed Mar 07 03:00:57 +0000 2018</td>
      <td>0.847</td>
      <td>0.068</td>
      <td>0.085</td>
      <td>@CBSNEWS</td>
      <td>RT @UnchainedAtLast: Did you miss the chance t...</td>
      <td>83</td>
    </tr>
    <tr>
      <th>483</th>
      <td>0.1027</td>
      <td>Wed Mar 07 03:00:43 +0000 2018</td>
      <td>0.847</td>
      <td>0.068</td>
      <td>0.085</td>
      <td>@CBSNEWS</td>
      <td>RT @UnchainedAtLast: Did you miss the chance t...</td>
      <td>84</td>
    </tr>
    <tr>
      <th>484</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:00:41 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews CAN'T PICTURE THESE TWO IN THE SAME R...</td>
      <td>85</td>
    </tr>
    <tr>
      <th>485</th>
      <td>-0.4019</td>
      <td>Wed Mar 07 03:00:39 +0000 2018</td>
      <td>0.701</td>
      <td>0.199</td>
      <td>0.100</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Surveillance video shows a woman ...</td>
      <td>86</td>
    </tr>
    <tr>
      <th>486</th>
      <td>0.3818</td>
      <td>Wed Mar 07 03:00:27 +0000 2018</td>
      <td>0.894</td>
      <td>0.000</td>
      <td>0.106</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: "Flippy," a machine that costs $6...</td>
      <td>87</td>
    </tr>
    <tr>
      <th>487</th>
      <td>0.5003</td>
      <td>Wed Mar 07 03:00:24 +0000 2018</td>
      <td>0.356</td>
      <td>0.207</td>
      <td>0.437</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews Weapons aren't toys they kill innocen...</td>
      <td>88</td>
    </tr>
    <tr>
      <th>488</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:00:20 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: The little girl who was photograp...</td>
      <td>89</td>
    </tr>
    <tr>
      <th>489</th>
      <td>0.3182</td>
      <td>Wed Mar 07 03:00:19 +0000 2018</td>
      <td>0.779</td>
      <td>0.082</td>
      <td>0.139</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Adult film performer Stormy Danie...</td>
      <td>90</td>
    </tr>
    <tr>
      <th>490</th>
      <td>0.0000</td>
      <td>Wed Mar 07 03:00:07 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>NOW: @CBSEveningNews with @JeffGlor airs in it...</td>
      <td>91</td>
    </tr>
    <tr>
      <th>491</th>
      <td>0.0000</td>
      <td>Wed Mar 07 02:59:43 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews @CBSThisMorning Are the clintons spon...</td>
      <td>92</td>
    </tr>
    <tr>
      <th>492</th>
      <td>0.3182</td>
      <td>Wed Mar 07 02:59:15 +0000 2018</td>
      <td>0.779</td>
      <td>0.082</td>
      <td>0.139</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Adult film performer Stormy Danie...</td>
      <td>93</td>
    </tr>
    <tr>
      <th>493</th>
      <td>0.0000</td>
      <td>Wed Mar 07 02:59:13 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>@CNN @MSNBC @CBSNews @CBSThisMorning @TheTalkC...</td>
      <td>94</td>
    </tr>
    <tr>
      <th>494</th>
      <td>-0.4019</td>
      <td>Wed Mar 07 02:59:07 +0000 2018</td>
      <td>0.701</td>
      <td>0.199</td>
      <td>0.100</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Surveillance video shows a woman ...</td>
      <td>95</td>
    </tr>
    <tr>
      <th>495</th>
      <td>0.2878</td>
      <td>Wed Mar 07 02:59:04 +0000 2018</td>
      <td>0.910</td>
      <td>0.000</td>
      <td>0.090</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: CBS News analyzed more than a dec...</td>
      <td>96</td>
    </tr>
    <tr>
      <th>496</th>
      <td>0.0000</td>
      <td>Wed Mar 07 02:59:00 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: CBS News investigates the mining ...</td>
      <td>97</td>
    </tr>
    <tr>
      <th>497</th>
      <td>0.0000</td>
      <td>Wed Mar 07 02:58:54 +0000 2018</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>@CBSNEWS</td>
      <td>@CBSNews Makes sense assuming Tyler was  fairl...</td>
      <td>98</td>
    </tr>
    <tr>
      <th>498</th>
      <td>-0.4019</td>
      <td>Wed Mar 07 02:58:53 +0000 2018</td>
      <td>0.701</td>
      <td>0.199</td>
      <td>0.100</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Surveillance video shows a woman ...</td>
      <td>99</td>
    </tr>
    <tr>
      <th>499</th>
      <td>0.6124</td>
      <td>Wed Mar 07 02:58:45 +0000 2018</td>
      <td>0.808</td>
      <td>0.000</td>
      <td>0.192</td>
      <td>@CBSNEWS</td>
      <td>RT @CBSNews: Texas Rep. Beto O'Rourke faces st...</td>
      <td>100</td>
    </tr>
  </tbody>
</table>
<p>500 rows × 8 columns</p>
</div>




```
import matplotlib.pyplot as plt
# Obtain the x and y coordinates for each of the news tweets
BBC = Full_tweet_df[Full_tweet_df["User"] == "@BBCWORLD"]
CBS = Full_tweet_df[Full_tweet_df["User"] == "@CBSNEWS"]
NYTIMES = Full_tweet_df[Full_tweet_df["User"] == "@nytimes"]
FOX = Full_tweet_df[Full_tweet_df["User"] == "@FOXNEWS"]
CNN = Full_tweet_df[Full_tweet_df["User"] == "@CNN"]


```


```
fig, ax = plt.subplots()
# Build the scatter plots for each media type
plt.scatter(BBC['tweets_ago'], 
            BBC['Compound'], 
            c="lightblue", 
            edgecolor="black", linewidths=1, marker="o", 
            alpha=1, label="BBC")

plt.scatter(CBS['tweets_ago'], 
            CBS['Compound'], 
            c="green", 
            edgecolor="black", linewidths=1, marker="o", 
            alpha=1, label="CBS")
plt.scatter(NYTIMES['tweets_ago'], 
            NYTIMES['Compound'], 
            c="yellow", 
            edgecolor="black", linewidths=1, marker="o", 
            alpha=1, label="NYTIMES")
plt.scatter(FOX['tweets_ago'], 
            FOX['Compound'], 
            c="royalblue", 
            edgecolor="black", linewidths=1, marker="o", 
            alpha=1, label="FOX")
plt.scatter(CNN['tweets_ago'], 
            CNN['Compound'], 
            c="red", 
            edgecolor="black", linewidths=1, marker="o", 
            alpha=1, label="CNN")


# Incorporate the other graph properties
plt.title("Sentiment Analysis of Media Tweets (3/6/2018)")
plt.ylabel("Tweet Polarity")
plt.xlabel("Tweets Ago")
plt.xlim((105,-5))
plt.ylim(-1.0,1.0)
plt.grid(True)

# Create a legend
lgnd = plt.legend(fontsize="small", mode="Expanded", 
                  numpoints=1, scatterpoints=1, 
                  loc=(1.01,.60), title="Media Sources", 
                  labelspacing=0.2,edgecolor='none')
lgnd.legendHandles[0]._sizes = [20]
lgnd.legendHandles[1]._sizes = [20]
lgnd.legendHandles[2]._sizes = [20]
lgnd.legendHandles[3]._sizes = [20]
lgnd.legendHandles[4]._sizes = [20]

sns.set(style="darkgrid")
# Save Figure
plt.savefig("Fig1.png", bbox_inches='tight')

# Show plot
plt.show()
```


![png](output_5_0.png)



```
#Create news org field for graph
summary_df['News_org'] = summary_df['User'].str[1:4]
summary_df['News_org'] = summary_df['News_org'].str.upper()
summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>Positive</th>
      <th>Tweet Count</th>
      <th>User</th>
      <th>News_org</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.181637</td>
      <td>0.81619</td>
      <td>0.13673</td>
      <td>0.04712</td>
      <td>100</td>
      <td>@BBCWORLD</td>
      <td>BBC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.039363</td>
      <td>0.88219</td>
      <td>0.04874</td>
      <td>0.06906</td>
      <td>100</td>
      <td>@nytimes</td>
      <td>NYT</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.000168</td>
      <td>0.83645</td>
      <td>0.08034</td>
      <td>0.08319</td>
      <td>100</td>
      <td>@CNN</td>
      <td>CNN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.047154</td>
      <td>0.83541</td>
      <td>0.06751</td>
      <td>0.09705</td>
      <td>100</td>
      <td>@FOXNEWS</td>
      <td>FOX</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.023787</td>
      <td>0.81706</td>
      <td>0.08865</td>
      <td>0.09432</td>
      <td>100</td>
      <td>@CBSNEWS</td>
      <td>CBS</td>
    </tr>
  </tbody>
</table>
</div>




```
import seaborn as sns
#Create Color Scheme
color=(['lightblue',"yellow","darkgreen","red","royalblue"])

```


```
#Build Bar Chart with Labels and color scheme
fig, ax = plt.subplots()
rects1 = ax.bar(summary_df['News_org'], summary_df['Compound'], color=color, alpha=0.5, align="center",width=1,edgecolor='black')


def autolabel(rects):
    """
    Attach a text label above each bar displaying its height
    """
    for rect in rects:
        height = rect.get_height()
        plt.text(rect.get_x()+rect.get_width()/2., 1.05*height, '%.2f'% float(height),
                ha='center', va='bottom')

autolabel(rects1)



plt.title("Overall Media Sentiment Based on Twitter (3/6/2018)")
plt.ylabel("Tweet Polarity")
plt.xlabel("Media")
plt.ylim(-0.20,0.1)

# Save Figure
plt.savefig("Fig2.png")

plt.show()
```


![png](output_8_0.png)

