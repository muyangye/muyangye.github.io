---
title:  "Netease Playlist to Spotify Migrater"
excerpt: "Creator and Programmer<br/><img src='/images/netease_spotify.png'>"
collection: portfolio
---

Netease Cloud Music is 1 of the 2 most popular music apps in China and I have used it intensively before. But many songs don't have copyrights here in USA (this is definitely not to say Netease is pirating songs, it's just that its subscription model only includes copyrights for songs to be played in China) and I don't want, ironically, to connect to a VPN that fakes my IP in China every time I listen to music. Therefore, I have decided to use Spotify as a subsitutive music app. Now the question becomes how to migrate my old playlists on Netease to Spotify: It is infeasible to do this manually since a playlist may have hundreds, even thousands of songs. This is why I built this project.

Since Netease only opens APIs for partners, I have to rely on existing libraries and the one I used is <a href="https://github.com/mos9527/pyncm" target="_blank">PyNCM</a>. By contrast, Spotify actually opens its official APIs so ofc I tried to use it in the beginning (which implies I didn't use it eventually, explained later). The biggest challenge I encountered is **OAuth 2.0**, a rather complex authentication with minimal documentation/explanation, which is why I am writing a blog here to help others who may face the same difficulty in the future using Spotify as an example.

So, **access_token** is the single most important thing we must pass in to all Spotify (or other apps that implement OAuth 2.0) APIs' request header, and we must first get it through another POST request to the ```/api/token``` endpoint, passing in **client_id** and **client_secret** of Spotify, which is very easy to implement. This is the workflow:

![Picture 2](/images/auth-client-credentials.png)

Passing in the obtained **access_token** to request headers, you are already able to call various APIs including Albums, Artists, Search, Playlists, etc. You can do plenty of things now such as getting an artist's related artists. However, if you try to add a track to a playlist through the correspoinding Playlists API, you will receive an error. I was very confused why, I probably just stared at the screen for half an hour until I noticed the only difference: On top of that Playlists API's documentation, there's something called **Authorization scopes**. This **Authorization scopes** was NEVER mentioned in the <a href="https://developer.spotify.com/documentation/web-api/tutorials/getting-started" target="_blank">Getting started</a> section of Spotify's documentation. What's worse, when I opened Spotify's <a href="https://developer.spotify.com/documentation/web-api/concepts/scopes" target="_blank">Scopes documentation</a>, it only showed you should use abc scopes to do xyz things, without telling you how to use it.

This is where **OAuth 2.0** comes in. Remember the POST request we used to get **access_token**? Before doing that, we must POST to another endpoint ```/authorize``` passing in **scope** in addition to **client_id**. We must also set **response_type** to **"code"** in the same request. The return is **NO LONGER access_token** but a url for you to open and login to your own, real Spotify account. After logging in, you will be redirected to another uri that contains a parameter **code**, which is something we need to pass in when POSTing the ```/api/token``` endpoint along with **grant_type** set to **"authorization_code"** to get **access_token**, the apple of our eyes. This is the more complex workflow:

![Picture 3](/images/auth-code-flow.png)

With that **access token** and correctly set scopes, you can call APIs that require login. Using Python's requests and webbrowser modules, this is the base class that implements OAuth 2.0 with a function that gets the **access token**:

```python
import webbrowser
import requests
import base64

# Spotify's authorization and access_token endpoints, replace with other apps' to extend
AUTHORIZATION_ENDPOINT = "https://accounts.spotify.com/authorize"
ACCESS_TOKEN_ENDPOINT = "https://accounts.spotify.com/api/token"

class OAuth2Client:
    def __init__(self, client_id, client_secret, redirect_uri):
        self.client_id = client_id
        self.client_secret = client_secret
        # NOTE: This self.redirect_uri is different from redirect_uri below after logging in. This is the one when you created your Spotify app
        self.redirect_uri = redirect_uri
        self.scope = "playlist-modify-public playlist-modify-private"
        self.access_token = self.get_access_token()

    # TODO: Add access token cache
    def get_access_token(self):
        login_url = requests.post(AUTHORIZATION_ENDPOINT,
            params = {
                "client_id": self.client_id,
                "response_type": "code",
                "redirect_uri": self.redirect_uri,
                "scope": self.scope
            }
        ).url
        # Open up browser and sign in to Spotify
        webbrowser.open(login_url)
        # This is the redirect_uri that contains code
        redirect_uri = input("Please copy the entire redirected url after you logged in here:\n")
        code = redirect_uri[redirect_uri.index("?code=") + 6 : ]
        access_token = requests.post(ACCESS_TOKEN_ENDPOINT, 
            headers = {
                "Authorization": "Basic " + base64.b64encode(bytes(self.client_id + ":" + self.client_secret, "utf-8")).decode(),
                "Content-Type": "application/x-www-form-urlencoded"
            },
            params = {
                "grant_type": "authorization_code",
                # This is the code in the parameter of redirect_uri
                "code": code,
                "redirect_uri": self.redirect_uri,
            }
        ).json()["access_token"]
        return access_token
```

With this OAuth2Client base class, you will be able to get the access token of any apps that implement OAuth 2.0! And surely this suffices the entire functionality of migrating playlists from Netease Cloud Music to Spotify, given that the correct scopes are passed in. However, I mentioned I eventually did not use Spotify's API. The reason is really for development process. Sure with this code I can easily get the access token, but every time I run this code, I will be prompted to login again which is very annoying. Obviously I can manually store this access_token and not call this function the next time I run, but that seems very inelegant and unprogrammatically to me. See the **TODO** in the above code block? That's the "correct" way to record access_token. I've come up with many ideas on caching access token, including but not limited to database, redis, tempfile, and OS. I feel that all of them are overkills and will make this project unnecessarily complex. I wondered why there's no library for this since this is just a systematic process so I did a Google search and BOOM! <a href="https://github.com/spotipy-dev/spotipy" target="_blank">Spotipy</a> came up as the library I decided to use.

Here is the very simple source code with detailed instructions on how to run and migrate playlists from Netease Cloud Music to Spotify: <a href="https://github.com/muyangye/Netease_To_Spotify" target="_blank">https://github.com/muyangye/Netease_To_Spotify</a>. If you like the tool or this blog, feel free to leave a star!

Most important lesson learned? **ALWAYS LOOK FOR EXISTING LIBRARIES BEFORE RE-BUILDING WHEELS!**