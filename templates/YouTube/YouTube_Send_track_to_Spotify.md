<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YouTube/YouTube_Send_track_to_Spotify.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YouTube+-+Send+track+to+Spotify:+Error+short+description">🚨 Bug report</a>

**Tags:** #youtube #spotify #snippet

**Author:** [Josef (Joe) Trchalik](https://www.linkedin.com/in/joseftrchalik/)

## Input

### Import libraries


```python
try:
    import spotipy
except:
    !pip install spotipy
    import spotipy

import spotipy.util as util
from spotipy.oauth2 import SpotifyClientCredentials
from spotipy import SpotifyOAuth

try:
    import youtube_dl
except:
    !pip install youtube_dl
    import youtube_dl

try:
    import google_auth_oauthlib.flow
except:
    !pip install google_auth_oauthlib
    import google_auth_oauthlib.flow

try:
    import googleapiclient.discovery
    import googleapiclient.errors
except:
    !pip install googleapiclient
    import googleapiclient.discovery
    import googleapiclient.errors

import requests, json, os, re, time
from urllib.parse import parse_qs, urlparse
from subprocess import Popen, PIPE
from signal import SIGTERM, SIGKILL
```

### Setup Spotify

Here is a link to a PDF showing you how to get your Spotify credentials: [Link](https://drive.google.com/file/d/1dheG1I97sNJibPuZLrrvjYUsofCZX4Mc/view?usp=sharing)


```python
# Spotify credentials
SPOTIFY_CLIENT_ID = 'SPOTIFY_CLIENT_ID'
SPOTIFY_CLIENT_SECRET = 'SPOTIFY_CLIENT_SECRET'
SPOTIFY_PLAY_LIST_ID = 'SPOTIFY_PLAYLIST_ID'
SPOTIFY_USERNAME = 'SPOTIFY_USERNAME'

scope = 'user-follow-modify playlist-modify-private'
redir_uri = "http://localhost:8000"

# Max number of tracks to find in Spotify against each query
spotify_limit = 5
```

### Setup YouTube

Here is a link to a PDF showing you how to get your YouTube credentials: [Link](https://drive.google.com/file/d/1dheG1I97sNJibPuZLrrvjYUsofCZX4Mc/view?usp=sharing)


```python
# Google developer API key - required if using Youtube playlist URL as an input
GOOGLE_API_KEY = "GOOGLE_API_KEY"

# Google API OAUTH credentials - required for obtaining liked videos from user account(s)
GOOGLE_CLIENT_ID = "GOOGLE_CLIENT_ID"
GOOGLE_CLIENT_SECRET = "GOOGLE_CLEINT_SECRET"
GOOGLE_PROJECT_ID = "GOOGLE_PROJECT_ID"

# Avoiding blacklisting by YouTube
youtubeapi_lim = 99 # Maximum number of videos to fetch by GOOGLEAPICLIENT from a playlist - set to >=99 to avoid ERROR 429 - too many requests
youtubedl_lim = 99 # Maximum number of urls to process by YOUTUBE-DL- set to >=99 to avoid ERROR 429 - too many requests
sleep = 0.5 # Time to pause between fetching individual YT video data

# Get data on the Youtube content
ydl_opts = {'retries': 1, 
            'download': False}
ydl = youtube_dl.YoutubeDL(ydl_opts)
```


```python
# ************ YOUTUBE VIDEO / PLAYLIST / USER INFO

'''INPUT: Youtube video URL, playlist URL or Google API authentication

    OPTIONS:

    youtube_video_url = 'https://www.youtube.com/watch?v=qgaRVvAKoqQ' # URL of a single video

    youtube_video_url = ['https://www.youtube.com/watch?v=qgaRVvAKoqQ', ...] # A list of video URLs

    youtube_video_url='https://www.youtube.com/playlist?list=PLbZIPy20-1pN7mqjckepWF78ndb6ci_qi' # Playlist URL

    youtube_video_url=['https://www.youtube.com/playlist?list=PLbZIPy20-1pN7mqjckepWF78ndb6ci_qi',...] # A list of Playlist URLs

    youtube_video_url=['https://www.youtube.com/playlist?list=PLbZIPy20-1pN7mqjckepWF78ndb6ci_qi',
                        'https://www.youtube.com/watch?v=qgaRVvAKoqQ'] # A mixed list of video & playlist URLs

    youtube_video_url="client_secrets.json" # A path to a JSON file with YouTube API credentials

    # A dictionary file with YouTube credentials - requires "Desktop" OAuth type
    youtube_video_url={"installed":{
        "client_id":GOOGLE_CLIENT_ID,
        "project_id":GOOGLE_PROJECT_ID,
        "auth_uri":"https://accounts.google.com/o/oauth2/auth",
        "token_uri":"https://oauth2.googleapis.com/token",
        "auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs",
        "client_secret":GOOGLE_CLIENT_SECRET,
        "redirect_uris":["urn:ietf:wg:oauth:2.0:oob","http://localhost:"]}}
    
*********************************************************************
'''

youtube_video_url = 'INPUT_AS_PER_ABOVE'
```

## Model

### Process Youtube Data - Functions


```python
# FUNCTIONS TO OBTAIN VIDEO METADATA IF YOUTUBE API LOGIN PROVIDED INSTEAD OF URLS

def get_youtube_client(creds,api_service_name = "youtube",api_version = "v3"):
        """ Log Into Youtube, Copied from Youtube Data API """
        
        # Using a JSON file with user credentials
        if '.json' in creds:
            # Get credentials and create an API client
            scopes = ["https://www.googleapis.com/auth/youtube.readonly"]
            flow = google_auth_oauthlib.flow.InstalledAppFlow.from_client_secrets_file(
                creds, scopes)

            creds = flow.run_console()

            # from the Youtube DATA API - for user data, e.g. liked items
            youtube_client = googleapiclient.discovery.build(api_service_name, api_version, credentials = creds)
        else:
            # Using API KEY - for public data
            youtube_client = googleapiclient.discovery.build(api_service_name, api_version, developerKey = creds)

        return(youtube_client)

def get_liked_videos(creds,lim):

    """Grab Our Liked Videos & Create A Dictionary Of Important Song Information
    https://developers.google.com/youtube/v3/docs/videos/list
    
    CREDS: either json file with credentials OR developer API key
    """
    
    request = get_youtube_client(creds).videos().list(
        part="snippet,contentDetails,statistics",
        myRating="like",
        maxResults=lim
    )
    response = request.execute()

    youtube_urls=[]

    # collect each video and get important information
    for item in response["items"]:
        video_title = item["snippet"]["title"]
        youtube_url = "https://www.youtube.com/watch?v={}".format(item["id"])
        youtube_urls.append(youtube_url)

    return(youtube_urls)
```


```python
# FUNCTIONS FOR GETTING INFO ON YOUTUBE VIDEOS FROM PLAYLIST OR VIDEO URLS

# Split Youtube playlist into individual video URLs
def getPlaylistLinks(url,api_key,api_service_name = "youtube",api_version = "v3"):
    
    try:
        query = parse_qs(urlparse(url).query, keep_blank_values=True)
        playlist_id = query["list"][0]

        print(f'get all playlist items links from {playlist_id}')
        youtube = googleapiclient.discovery.build(api_service_name,api_version, developerKey = api_key)

        request = youtube.playlistItems().list(
            part = "snippet",
            playlistId = playlist_id,
            maxResults = 50
        )
        response = request.execute()

        playlist_items = []
        
        while request is not None:
            response = request.execute()
            playlist_items += response["items"]
            request = youtube.playlistItems().list_next(request, response)

        #print(f"total: {len(playlist_items)}")
        playlist=[f'https://www.youtube.com/watch?v={t["snippet"]["resourceId"]["videoId"]}' 
                  for t in playlist_items]
    except Exception as e:
        playlist=[]
        print(e)
        
    return(playlist)

def llower(x):
    
    # Converts list or tuple elements or strings to lower-case
    
    if isinstance(x,(list,tuple)):
        x=[str(xx).lower() for xx in x]
    elif isinstance(x,str):
        x=str(x).lower()
    else:
        print('LLOWER():Invalid input format - only use list, tuple or scalars as input')
        
    return(x)

# Function for downloading YT video data
def getVideoInfo(url,out_url,down=False,sleep=1):
    
    # https://ostechnix.com/youtube-dl-tutorial-with-examples-for-beginners/
    
    time.sleep(sleep)
    
    try:
        with ydl:
            info = ydl.extract_info(
            url,
            download=down
        )

        kz=list(info.keys())
   
        print(info['title'])
         
        # If we have both artist and track
        if ('artist' in llower(kz) )& ('track' in llower(kz)):
            
            outi={}
            outi['artist']=info['artist'].lower()
            outi['track']=info['track'].lower()
            out_url.append(outi)
        
        # If category = music or VEVO in decription or tag
        elif ('music' in llower(info['categories']))|('vevo' in llower(info['description']))|('vevo' in llower(info['tags'])):
            
            # Items to remove
            rem=['official video','remastered','explicit',')','(','|','"',"'"]
            
            title=info['title'].lower() # title in lower-case
            
            # Remove REM strings (above)
            for r in range(len(rem)):
                title = title.replace(rem[r],'')
            
            # Split artist and track info
            ordr=[0,1] # Order: artist, title
            if '-' in title:
                title = title.split('-')
            elif ':' in title:
                title = title.split(':')
            elif 'by' in title:
                title = title.split('by')
                ordr=[1,0] # Order: title, artist
            
            # If we have both artist and track
            if isinstance(title,list):
                title=[t.strip() for t in title]

                outi={}
                outi['artist']=title[ordr[0]]
                outi['track']=title[ordr[1]]
                out_url.append(outi)
            
    except Exception as e:
        print(e)

    return(out_url,info)
```


```python
# GET VIDEO METADATA
def processURLs(youtube_video_url,lim,api_key,youtubeapi_lim):
    # If a single URL, convert to list

    if isinstance(youtube_video_url,(list,tuple))==False:
        youtube_video_url=[youtube_video_url]
    
    # MAIN BODY
    
    # Iterate through Youtube URLs

    out=[]
    count=1
    all_info=[]
    
    for url in youtube_video_url:

        # If number of fetched videos is less than the limit
        if count<=lim:
            
            print('>>>>>>>>>>>>>>>>>>>>>')
            print('Processing URL: ',url)
            
            # YOUTUBE CREDENTIALS IN A DICT SUPPLIED INSTEAD OF PLAYLIST OR VIDEO URL(s)
            # > convert to a list of URLs of liked videos
            if isinstance(url,dict):
                # save as JSON
                with open('yt_secrets.json', 'w') as outfile:
                    json.dump(url, outfile)
                url='yt_secrets.json'

            # YOUTUBE CREDENTIALS IN A JSON FILE SUPPLIED INSTEAD OF PLAYLIST OR VIDEO URL(s)
            if '.json' in url.lower():
                # Get liked videos from the account
                url=get_liked_videos(url,youtubeapi_lim)
            
            out_url=[]
            
            # It is a list - split into videos and download data of individual videos
            if playlist_str in url:

                print('This is a Youtube playlist - extracting individual URLs!')

                # Can be a playlist or a list of videos

                urls=getPlaylistLinks(url,api_key)

                l=len(urls)

                print('The Youtube playlist contains ',l,' videos')

                for u in urls:
                    if count<=lim:
                        out_url,info=getVideoInfo(u,out_url,sleep=sleep)
                        count+=1
                        all_info.append(info)
                    else:
                        break

                if len(out_url)>0:
                    print('The Youtube playlist contains videos with ',len(out_url),' music tracks with known artists: ')
                    for lu in range(len(out_url)):
                        print(out_url[lu]['artist'],' - ',out_url[lu]['track'])
                else:
                    print('Sorry, the Youtube playlist does not contain any videos with music track names and / or artist info')

            # It is a  single video
            else:

                print('This is a single Youtube video!')
                
                url_split=url.split('&list=')
                
                if isinstance(url_split,list):
                    url=url_split[0]
                
                out_url,info=getVideoInfo(url,out_url,sleep=sleep)
                all_info.append(info)
                count+=1

                if len(out_url)>0:
                    print('This Youtube video contains valid artists & track data: ')
                    for lu in range(len(out_url)):
                        print(out_url[lu]['artist'],' - ',out_url[lu]['track'])
                else:
                    print('Sorry, this Youtube video does not contain music track name and / or known artist info')

            out=out+out_url

        # Number of videos is larger than the limit
        else:
            break
            
    return(out,all_info)
```


```python
# Helper function for address already in use error
def freePort(prt):
    try:
        command = "netstat -ano | findstr "+str(prt)
        c = Popen(command, shell=True, stdout=PIPE, stderr = PIPE)
        stdout, stderr = c.communicate()
        if len(stdout)>0:
            pid = int(stdout.decode().strip().split(' ')[-1])
            os.kill(pid,SIGTERM)
            #os.kill(pid,SIGKILL)
    except Exception as e:
        print(e)
        # Change port no
        prt+=1
        
    return(prt)

# Function for findin the track on Spotify - public scope
def findSpotifyTrack(out,client_id,client_secret,spotify_limit):
    
    # Log into Spotify API - for accessing public data
    client_credentials_manager = SpotifyClientCredentials(client_id=client_id, client_secret=client_secret)
    sp = spotipy.Spotify(client_credentials_manager=client_credentials_manager)

    # FIND TRACKS
    items=[]
    spotify_ids=[]

    # Search for tracks
    for t in out:

        query='artist:' + t['artist'] + ' track:' + t['track']
        track_id = sp.search(q=query,type='track',limit=spotify_limit)

        l_out=len(track_id['tracks']['items'])

        if l_out>0:
            for ls in range(l_out):
                artist=track_id['tracks']['items'][ls]['artists'][0]['name']
                track=track_id['tracks']['items'][ls]['name']
                spotify_id=track_id['tracks']['items'][ls]['id']

                song=artist+' - '+track
                items.append(song)

                spotify_id="spotify:track:"+spotify_id
                spotify_ids.append(spotify_id)

    return(spotify_ids,items)
        
# Function for adding a track to user playlist - user / private scope
def addSpotifyTrack(spotify_ids,client_id, client_secret, redirect_uri_port,username,playlist_id,retry=3):
    
    spotify_idsF=[]
    retry_count=1
    
    while retry_count<=retry:
        
        redirect_uri="http://localhost:"+str(redirect_uri_port)
        
        # Spotify user authorization - for accessing user data
        creds = SpotifyOAuth(scope=scope, client_id=client_id, client_secret=client_secret, 
                             redirect_uri=redirect_uri)
        sp2 = spotipy.Spotify(auth_manager=creds)
        
        # ADD TRACKS TO THE PLAYLIST
        try:
            sp2.user_playlist_add_tracks(username, playlist_id=playlist_id, tracks=spotify_ids)
            spotify_idsF=spotify_ids
            
            retry_count=retry+1
        except Exception as e:
            print(e)
            err=str(e)
            if 'address already in use' in err.lower():
                # If port not available, free it or change port no
                redirect_uri_port=freePort(redirect_uri_port)
                retry_count+=1
                
            else:
                # Other type of error - terminate
                retry_count=retry+1
        
    return(spotify_idsF)
```

### Process Youtube Data - Execute


```python
# **** OBTAIN YOUTUBE VIDEO DETAILS FROM VIDEO OR PLAYLIST URLS
out,all_info=processURLs(youtube_video_url,youtubedl_lim,GOOGLE_API_KEY,youtubeapi_lim)

# Saving all track info for reference
with open('all_track_info.json', 'w') as outfile:
    json.dump(all_info,outfile)

print('***********************')
print('Youtube data processing completed - total number of tracks with interpreter info found: ',len(out))

for i in range(len(out)):
    print(out[i]['artist']+' - '+out[i]['track'])
```

### Find tracks on Spotify & add to the playlist


```python
# Find tracks in spotify
spotify_ids,items = findSpotifyTrack(out,
                                     SPOTIFY_CLIENT_ID,
                                     SPOTIFY_CLIENT_SECRET,
                                     spotify_limit)

# Add tracks to Spotify playlist - user / private scope
if len(spotify_ids) > 0:
    spotify_idsF = addSpotifyTrack(spotify_ids,
                                   SPOTIFY_CLIENT_ID,
                                   SPOTIFY_CLIENT_SECRET,
                                   redir_uri_port,
                                   SPOTIFY_USERNAME,
                                   SPOTIFY_PLAY_LIST_ID)
else:
    spotify_idsF = []
```

## Output

### Return message


```python
if len(spotify_idsF)==0:
    print('Completed: No tracks found & added to Spotify playlist, sorry!')
else:
    print('Completed: ',len(spotify_idsF),' tracks added to Spotify playlist: ')
    print('------------------------')
    for i in items:
        print(i+'\n')
```
