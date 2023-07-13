---
layout: post
title: Analyzing My Spotify Top Songs 2016 - 2022 Part 1
---

In this series of blog posts, I will discuss how my musical tastes have changed over the past six years using the data from my Spotify top songs playlists. One of the things I love about Spotify is their summary playlists of the top 100 songs you listened to during the past year. These playlists are called "Your Top Songs (year)" and are usually created in early December of that year. I've been a Spotify premium subscriber since May/June 2014, but I don't have a record of my top songs from 2014 or 2015. However, I do have a record of my top songs from 2016 to 2022.

### My Musical Tastes Prior to Spotify

Before I met my wife, I pretty much just listened to video game and movie soundtracks and remixes of video game music made by the very talented folks at [OCReMix](https://ocremix.org). I exclusively used the [foobar2000](https://foobar2000.org) software with a classic 'green text on a black background' theme to listen to my music. I turned my nose up at any kind of pop, hip hop, or country music. I was so into OCReMix that I named my college radio show after it - The OCReMix Show with DJ Willdabeast. Shortly after we met, my wife introduced me to Spotify and to music that didn't come from a movie or video game. The OCReMix Show then became the OverClocked Fusion Show with DJ Willdabeast and DJ Supernova where video game music was played side-by-side with country, pop, hiphop, and rock/alternative. 

## Spotify Audio Features
Every song on Spotify can be characterized by the following metrics: acousticness, danceability, duration, energy, instrumentalness, key, liveness, loudness, mode, speechiness, tempo, time signature, and valence. A description of each audio feature is below for your convenience. I lifted some of the language from Spotify's official [documentation](https://developer.spotify.com/documentation/web-api/reference/#/operations/get-audio-features).

* Acousticness is the confidence that a track is acoustic (0.0 - not acoustic, 1.0 - acoustic). Acoustic music primarily uses instruments that don't use electricity to produce sound such as the trumpet, saxophone, clarinet, acoustic guitar, piano, violin, etc.

* Danceability is the confidence that a track is suitable for dancing (0.0 - least danceable, 1.0 - most danceable). This metric combines several musical elements including tempo, rhythm stability, beat strength, and overall regularity.

* Duration is the length of the track.

* Energy describes the intensity and activity of a track (0.0 - very low energy, 1.0 - very high energy). Classical music scores low while metal music scores high. This metric combines several elements including dynamic range, perceived loudness, timbre, onset rate, and general entropy.

* Instrumentalness is the confidence that a track contains no vocals other than "oohs" or "aahs" (values greater than 0.5 likely contain no vocals, but confidence increases as the value approaches 1.0). 

* Key - the key the track is in. From Spotify's docs, "Integers map to pitches using standard Pitch Class (https://en.wikipedia.org/wiki/Pitch_class) notation. E.g. 0 = C, 1 = C♯/D♭, 2 = D, and so on. If no key was detected, the value is -1."

* Liveness is the confidence that a track was performed live (0.0 - recorded in studio, > 0.8 - likely song was recorded live).

* Loudness is the loudness of the track. This value is the average loudness of the track and thus allows comparison between tracks. 

* Mode indicates whether a given track is major or minor key (0 - minor, 1 - major).

* Speechiness is the confidence that a track primarily consists of speech. Values greater than 0.66 are likely all spoken words. Values between 0.33 and 0.66 contain a mix of music and speech. Values below 0.33 contain music with sung vocals.

* Tempo is the overall tempo of a track in beats per minute (BPM).

* Time signature indicates how many beats are in each measure. Values range from 3 to 7 which indicate "3/4" to "7/4" time.

* Valence describes the musical positiveness of a track (0.0 - negative, 1.0 - positive). From Spotify's docs: "Tracks with high valence sound more positive (e.g. happy, cheerful, euphoric), 
while tracks with low valence sound more negative (e.g. sad, depressed, angry)."



## Python Code
Using the power of Python and the [Spotify API](https://developer.spotify.com/documentation/web-api/), I downloaded my top songs from 2016 to 2022 along with their audio features as CSV files. Once I had my top songs playlists downloaded with the associated audio features, I collected them together using pandas and then used Seaborn to generate box-and-whisker plots of each year's data side-by-side.

Below is the data extraction script. I used the requests library to interface with the API and the pandas library to convert the list of dicts to a CSV file.
```python
# -*- coding: utf-8 -*-
"""
@author: Will Pisani, PhD

This script will download my top songs from 2016 to 2022 and save them to CSV files.
"""

import requests, os
import pandas as pd

def GetAllTracksInfo(playlist_id,authorization_header):
    """
    Parameters
    ----------
    playlist_id : String
        Spotify playlist ID.
    authorization_header : dict
        The standard Spotify API authorization header.

    Returns
    -------
    tracks : list 
        List of dicts with each dict containing information about a song.

    """
    # base URL of all Spotify API endpoints
    BASE_URL = 'https://api.spotify.com/v1/'
    track_json = requests.get(BASE_URL + 'playlists/' + playlist_id + '/tracks',headers=authorization_header).json()
    tracks = []
    
    for track in track_json['items']:
        track = track['track']
        track_info = requests.get(BASE_URL + 'audio-features/' + track['id'],headers=authorization_header).json()
        track_info.update({
                            'track_name': track['name'],
                            'release_data': track['album']['release_date'],
                            'album_name': track['album']['name'],
                            'popularity': track['popularity']})
        
        artists = [] # To handle multiple artists
        for artist in track['artists']:
            artists.append(artist['name'])
        artists_str = ', '.join(artists)
        
        track_info.update({
            'artists': artists_str})
        
        tracks.append(track_info)
    
    return tracks

auth_url = 'https://accounts.spotify.com/api/token'

CLIENT_ID = 'your_client_id'
CLIENT_SECRET = 'your_client_secret'

data = {
    'grant_type': 'client_credentials',
    'client_id': CLIENT_ID,
    'client_secret': CLIENT_SECRET,
}

auth_response = requests.post(auth_url, data=data)

access_token = auth_response.json().get('access_token')

headers = {
    'Authorization': f'Bearer {access_token}'
}

# base URL of all Spotify API endpoints
BASE_URL = 'https://api.spotify.com/v1/'

# My Top Songs = MTS
my_top_songs = {2016 : {'id' : '2016_id'},
                2017 : {'id' : '2017_id'},
                2018 : {'id' : '2018_id'},
                2019 : {'id' : '2019_id'},
                2020 : {'id' : '2020_id'},
                2021 : {'id' : '2021_id'},
                2022 : {'id' : '2022_id'}}

os.chdir(r'/directory/to/save/playlist/csvs/to')

for year in range(2016,2022+1):
    # Add tracks to dictionary for debugging and inspection
    my_top_songs[year]['tracks'] = GetAllTracksInfo(my_top_songs[year]['id'],headers)
    df = pd.DataFrame(my_top_songs[year]['tracks'])
    df.to_csv(f"MTS{str(year)}.csv")
```

Below is the data analysis script.

```python
# -*- coding: utf-8 -*-
"""
@author: Will Pisani, PhD

This script will analyze My Top Songs from 2016 to presently available data.
"""
from collections import Counter
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import os

directory = r'/directory/to/csv/files'
os.chdir(directory)
years = ['2016','2017','2018','2019','2020','2021','2022]

# Combine all years into one data frame
dataframes = []
for year in years:
    dataframes.append(pd.read_csv(f'MTS{year}.csv').assign(Year=int(year)))
df = pd.concat(dataframes)

# Get averages and standard deviations of each characteristic by year
mean_df = df.groupby(['Year']).mean()
std_df = df.groupby(['Year']).std()

cols = df.columns

# Add duration in minutes to the dataframe
df['duration_min'] = df['duration_ms']/1000/60
features = ['acousticness','danceability','duration_min','energy','instrumentalness',
            'key','liveness','loudness','mode','speechiness','tempo','time_signature',
            'valence']

# Spotify's official docs on the audio features
# https://developer.spotify.com/documentation/web-api/reference/#/operations/get-audio-features

os.chdir(r'/directory/where/you/want/to/save/the/plots/to')
for feature in features:
    fig, ax = plt.subplots()
    ax = sns.boxplot(x="Year",y=feature,data=df)
    fig.savefig(f'My_Top_Songs_Box_Plot_{feature}.png',dpi=300)
```


## My Top Songs Analysis - Acousticness

Now let's talk about the first audio feature - acousticness!

<figure>
   <img
   src="../images/Spotify/My_Top_Songs_Box_Plot_acousticness.png"
   alt="My Top Songs 2016-2022 Acousticness">
   <figcaption>Figure 1. Acousticness of Will Pisani's top songs as a function of time. The confidence that a song is acoustic increases as the value increases.</figcaption>
</figure>
 As you can see, the acousticness of my top songs has steadily decreased over the years with a large decrease in Q3 from 2019 to 2020. It's clear I listened to significantly more music with electronic elements than orchestral elements. The more I listened to music on Spotify and discovered more music, the less I listened to orchestral/acoustic music.
 <br /><br />
 
 My top songs spanned nearly the entire acousticness range from 2016 to 2019, but the range decreased significantly from 2020 onwards. There are four outliers in 2021 that are close to 1.0 acousticness, but most songs are nowhere near 1.0. So what happened with 2020? Well, the COVID-19 pandemic happened. 2020 was a dark time for me and I started listening to a lot more pop, rock, pop-rock, and pop-punk (enough that I made a pop-rock/pop-punk playlist in Sept 2020). Acoustically, 2021 and 2022 were a continuation of the downward trend, with 2022 exhibiting a significant decrease relative to 2021. I've really leaned into the pop, pop-rock, and country-pop over the last three years. I listen to more pop now than video game music (Golly, I don't think Will from 2014 would have ever thought that was possible!).

## Conclusion
I hope you enjoyed this post and found the Python code to be helpful. Stay tuned for the next part in the series!