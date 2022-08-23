---
title: "Spot-it (In Progress)"
date: 2022-08-23
summary: Spot new songs on Reddit based off your current Spotify playlists.
showtoc: true
draft: false
---
*Note: This project has been documented on what has been completed. More information to come as I continue working!*

The first version the Spot-It web application demo found [here](https://youtu.be/dDmbO5_ccEc).

The link to the updated code can be found [here]()

## Overview
Do you ever get tired of being recommended the same songs over and over again while on Spotify? If yes, Reddit is a great place to find new and upcoming artists as well as popular songs. This project seeks to provide a tool for users to explore certain music subreddits with hope to spot songs they may like. This tool will also provide the option for users to find which songs may be similar to playlist they currently have on Spotify. 

## ETL Process
The ETL process will be broken down into two stages the Reddit stage and then the Spotify stage.

### Reddit
#### Reddit Post Extraction
The first step in my ETL pipeline is to gather posts from specific subreddits. A python script, ```ingest.py```, can be ran using a bash script to extract the raw data into a Mongo collection. Example bash command for the r/indieheads subreddit:
```
python ingest.py \
    --db="TempPosts" \
    --coll="CollectRaw" \
    --sub="indieheads"
```
This script can be ran for any subreddit (in this case, music subreddits). 

#### Reddit Post Transformation
Because each subreddit will have different criteria for what is a song and how to extract the track and artist information, an abstract class will be created and methods will be defined further in the child classes. This class can be seen below:
```
class ExtractSong(RedditFeatures):
    
    def __init__(self, data_field):
        super().__init__(data_field)
    
    def is_track(self):
        return '-' in self.title
    
    def extract_track():
        pass

    def extract_artist():
        pass
```
This class will inherit all the important features we want to keep from the raw JSON file. 

By using this class template, we are easily able to add support for many different subreddits by defining the three methods above. This improves the scalability issue that arised in the verison 1 data collection process.

#### MongoDB Class
To aid in the ETL process, a custom class for basic methods relating to MongoDB can be created. These include establishing a connection, filtering the raw database, uploading data to a collection, and deleting data from a collection. 

#### Reddit ETL Script
A Python script can be used to incorporate all of our defined methods. We can create a dictionary containing the subreddit name as the key and that subreddits ExtractSong class. 
Example:
```
{ "indieheads": ExtractIndieHeads, 
  "AlternativeRock": ExtractAlternativeRock}
```
This will allow all subreddit data to be extracted from the raw database, filter only the song posts, generate "track" and "artist" fields, and finally append this data into a new staging collection where Spotify data can be added. 

### Spotify 
This stage is currently in progress.


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
Now that the ETL process is more scalable and modular, an orchestration tool such as Airflow will be used to schedule the ETL process on a daily or 12 hr period.
### Improve Speed
When working with large playlists, the application becomes quite clunky. By reworking some functions and code structure, the application will run much smoother. 
