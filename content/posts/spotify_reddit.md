---
title: "Spot-it (In Progress)"
date: 2022-05-15
summary: Spot new songs on Reddit based off your current Spotify playlists.
showtoc: true
draft: true
---
*Note: This project has been documented on what has been completed. More information to come as I continue working!*

## Overview
Do you ever get tired of being recommended the same songs over and over again while on Spotify? If yes, Reddit is a great place to find new and upcoming artists as well as popular songs. This project seeks to provide a tool for users to explore certain music subreddits with hope to spot songs they may like. This tool also will also provide the option for user to find which songs may be similar to playlist they currently have on Spotify. 

## Data Collection
In order to collect data from Reddit, PRAW (API wrapper) can be used. All API credentials will be saved in the ```config.py``` file. A read-only Reddit instance must be made before gathering posts. Once this is completed, we can scrape post data for the last 100 posts. Because each music subreddit has different title formats it is difficult to create one function to extract songs from all music subreddits. While exploring a solution to this problem, an "extract song" function can be made for each subreddit. The steps for the data collection for the 'indieheads' subreddit are outlined as follows:
1. Setup the read-only reddit instance
2. Specify the subreddit of interest
3. Determine a filtering metric for posts
    - '[FRESH]' means this post is referring to a 'fresh find' song
        - This is what we want so we can filter all new posts by this
    - A reddit link most likely not be referring to a song that is on Spotify
4. Posts have been filtered, so the song title must be extracted. 
    - The general form for 'indieheads' is 'Artist - Song'
    - Use regular expressions to subset:
        - Everything before the '-' character -> Artist
        - Everything after the '-' character -> Song
5. Store each post in a .json file format 

This process may not gather 100% of the songs but in our case, this should be fine because 'most' songs are gathered. 

Next, since we have the artist and song name, through the Spotify API we can gather song data on these songs. The data points are:
- Danceability 
- Valence
- Energy
- Tempo
- Loudness
- Speechiness
- Instrumentalness
- Liveness
- Acousticness

These data points will be later used to compare songs from Reddit to the user's playlist. 
To gather these data points for each song, these songs can be searched in the Spotify API using ```spotipy```. Some posts from the subreddit maybe from youtube or other sites. These songs might not exist on Spotify, but if they do we can extract their data. The final output will contain a JSON file with the songs from Reddit that exist on Spotify and the song's data points. 

The following code is for the 'indieheads' subreddit. 
```
def get_indieheads_data(num_post = 50):
    '''
    This function will scrape the 'indieheads' subreddit for music information
    subreddit (str): name of subreddit
    num_post (int): number of posts to scrape, default = 50, max = 1000
    returns json file format
    '''
    #Create read-only Reddit instance
    reddit = praw.Reddit(client_id=config.reddit_client_id, 
                     client_secret = config.reddit_secret_key, 
                     user_agent=config.reddit_user_agent)

    reddit_data = []
    subreddit = 'indieheads'
    subreddit_scrape = reddit.subreddit(subreddit)
    for post in subreddit_scrape.new(limit = num_post):
        title = post.title
        url = post.url
        if ('[FRESH]' in title) and ('reddit' not in url):
            
            #Subreddit Data Extraction
            temp_dict = {}
            temp_dict['_id'] = post.id
            #temp_dict['title'] = title
            temp_dict['created'] = post.created
            temp_dict['upvotes'] = post.score
            temp_dict['url'] = url
            temp_dict['subreddit'] = subreddit
            
            artist_all = re.sub(r'\-(.*)', '', title)
            artist_all = artist_all.replace('[FRESH]', '')
            temp_dict['artist_all'] = artist_all.lstrip()
            
            track = re.sub(r'(.*)\-', '', title)
            temp_dict['track'] = track.lstrip()
            
            
            #Spotify Data Extraction
            #Load in API information
            client_credentials_manager = SpotifyClientCredentials(
                client_id=config.spotify_client_id, 
                client_secret=config.spotify_secret_key)
            sp = spotipy.Spotify(client_credentials_manager = client_credentials_manager)
            
            #Extract song information from Spotify
            song_res = sp.search('{0} {1}'.format(artist_all, track))
            if song_res['tracks']['items'] == []:
                continue
            else:
                song_data = extract_song_data(song_res['tracks']['items'][0]['uri'])
            
            final_data = {**temp_dict, **song_data}
            
            #Add all combined data to list
            reddit_data.append(final_data)
        
        else:
            continue
        
    return reddit_data
```
From the output of ```get_indieheads_data()``` function, this data can be added to MongoDB to be saved for the future. 

The following function is used to upload only new posts not already in the database. 

```
def export_data(subreddit, database, num_post = 100):
    '''
    This function will populate MongoDB with respective subreddit posts
    subreddit (str): subreddit to scrape and add to DB 
    '''
    #Connect to MongoDB
    client = pymongo.MongoClient(config.mongo_cred)
    #Define database
    db = client[database]
    #Define collection within 'RedditCollect'
    collection = db[subreddit]

    subreddit_all = {
        'indieheads': get_indieheads_data,
        'Alternativerock': get_Alternativerock_data
        }
    
    subreddit_data = subreddit_all[subreddit](num_post = num_post)

    for post in subreddit_data:
        try:
            collection.insert_one(post)
        except pymongo.errors.DuplicateKeyError:
            pass
```

## Next Steps
- Refine the following code to improve and speed up the data collection process
    - Automated collection can be explored in the future
- Create a model to find similar songs between the user's playlists and the subreddit
- Create a dashboard to collect user playlist data 
