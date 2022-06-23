---
title: "Spot-it - Version 1"
date: 2022-06-20
summary: Spot new songs on Reddit based off your current Spotify playlists.
showtoc: true
draft: false
---
*Note: This project has been documented on what has been completed. More information to come as I continue working!*

This article describes the steps taken to create the Spot-It web application found [here](https://spot-it-songs.herokuapp.com). 

## Overview
Do you ever get tired of being recommended the same songs over and over again while on Spotify? If yes, Reddit is a great place to find new and upcoming artists as well as popular songs. This project seeks to provide a tool for users to explore certain music subreddits with hope to spot songs they may like. This tool will also provide the option for users to find which songs may be similar to playlist they currently have on Spotify. 

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
 

In the ```reddit_collect.py``` file, the functions ```get_indieheads_data()``` and ```extract_song_data()``` are used to extract song posts from the 'r/indieheads' subreddit and append them to the database for further use. 

## Spotify Functions
In the final web application, it will be important to collect some of the user's Spotify data. The ```spotify_collect.py``` file contains three functions that can aid in this data collection process.

### Determine User Playlist
The first function ```list_playlist()``` will return a list of all the users public playlists. This list can be display in a drop down on the dashboard for user selection. This funciton can be found below.

```
def list_playlist(user_id):

    play_name = []
    play_uri = []
    
    playlist = sp.user_playlists(user_id)
    for i in range(0, len(playlist['items'])):
        play_name.append(playlist['items'][i]['name'])
        play_uri.append(playlist['items'][i]['uri'])
    
    #Use playlist name as key and uri as value
    playlists = dict(zip(play_name, play_uri))
    
    return playlists
```

### Determine Songs from Playlist
Once a user selects a playlist, we must gather all songs from that selected playlist. ```Spotipy``` makes this process easy by providing songs based on playlist uri. The ```get_song()``` function can be found below.

```
def get_song(playlist_uri):
    
    song_info = []
    
    playlist_songs = sp.playlist_items(playlist_uri)
    for i in range(0, len(playlist_songs['items'])):
        each_song = {}
        #Track identifier
        each_song['track_id'] = playlist_songs['items'][i]['track']['uri']
        #Track name
        each_song['track_name'] = playlist_songs['items'][i]['track']['name']
        
        #1st artists on track's identifier
        each_song['artist_id'] = playlist_songs['items'][i]['track']['artists'][0]['uri']
        #1st artists on track's name
        each_song['artist_name'] = playlist_songs['items'][i]['track']['artists'][0]['name']

        song_info.append(each_song)
        
    return song_info
```
### Get Data on Songs
Once the songs have been gathered, the audio feature must be extracted for each song. This will serve as our 'prediction' dataset. The ```extract_song_data()``` function can be found below.

```
def extract_song_data(track_id):
   
    #use Spotipy to get track information
    track_feature = sp.audio_features(track_id)

    #Parse the track information
    song_info = {}
    song_info['danceability'] = track_feature[0]['danceability']
    song_info['energy'] = track_feature[0]['energy']
    song_info['key'] = track_feature[0]['key']
    song_info['loudness'] = track_feature[0]['loudness']
    song_info['mode'] = track_feature[0]['mode']
    song_info['speechiness'] = track_feature[0]['speechiness']
    song_info['acousticness'] = track_feature[0]['acousticness']
    song_info['instrumentalness'] = track_feature[0]['instrumentalness']
    song_info['liveness'] = track_feature[0]['liveness']
    song_info['valence'] = track_feature[0]['valence']
    song_info['tempo'] = track_feature[0]['tempo']
    song_info['uri'] = track_feature[0]['uri']
    song_info['duration_ms'] = track_feature[0]['duration_ms']
    
    return song_info
```

## Recommender Model
The main goal of Spot-it is to recommend songs similar to that of the users playlist. The approach to this problem is to use the reddit data as the dataset the algorithm will be built upon and the user data as a 'new' case we want to predict/classify. 

A clustering approach could be used by creating clusters for each subreddit. The users playlist data could then be assigned to certain clusters and a similarity metric could be generated this way. This causes one issue when the amount of subreddits are increased. There needs to be model created for every subreddit and this model will need to be updated every time the data is changed. For version 1 of Spot-it a simplier, more generalized approach will be used. 

### Cosine Similarity
Cosine similarity will allow us to take a vector (song data) from the user and find similar vecotrs (songs) within the subreddit of choice. To create a vector for the user, we will average all track features for their selected features. We will call this metric a user's playlist vibe. Cosine similarity will be used to generate subreddit songs based on the users vibe metric (single vector). 

Obviously this approach is very generalized and it will be considered as to whether the vibe is an accurate measure based on user feedback. 

## Web Application Creation - Streamlit
Once all functions have been created for the tasks we want to carry out, a prototype web application can be created using Streamlit. 

#### User Feedback
A simple form can be added to gain user feedback on the application. This data will saved in a database for later access.

### App Deployment
The Streamlit web application was deployed via Heroku.

## API Creation
To further increase the functionality of Spot-It, a REST API can be created. Endpoints can be made to retrieve data as well as serve model recommendations. 

### Data Retrieval 
An example to get the list of subreddits supported by Spot-It is found below.
```
@app.route("/getCollections")
def get_collections():
    db = client['RedditCollect']
    items = db.list_collection_names()
    return {"data": items}
```

A more user specific endpoint to utilize the ```list_playlist()``` function. 

```
#list user playlist
@app.route("/userPlaylist", methods=["GET","POST"])
def get_user_playlist():
    if request.method == 'POST':
        spotify_username = request.form['spotifyUsername']
    
    playlists = sc.list_playlist(spotify_username)

    return {"data": playlists}
```

### Purpose
This is just an introduction to the functionality of this API. The main purpose of developing this API is to begin to move away from the Streamlit application to a more robust front end. 

## Future Versions and Updates
### Automate Data Collection
Currently the ```reddit_collect.py``` file must be manually run to update the database with new posts. Automation tools can be used to schedule this process on a certain time frame (once a day, every hour, etc.)
### Improve Speed
When working with large playlists, the application becomes quite clunky. By reworking some functions and code structure, the application will run much smoother. 
