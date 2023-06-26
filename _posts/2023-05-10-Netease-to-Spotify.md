---
layout: post
title:  "Netease Playlist to Spotify Migrater"
summary: "Creator and Programmer"
date:   2023-05-10 00:00:00
preview: /assets/Netease%20Playlist%20to%20Spotify/cover.png
---

Netease Cloud Music is 1 of the 2 most popular music apps in China and I have used it intensively before. But many songs don't have copyrights here in USA (this is definitely not to say Netease is pirating songs, it's just that its subscription model only includes copyrights for songs to be played in China) and I don't want, ironically, to connect to a VPN that fakes my IP in China every time I listen to music. Therefore, I have decided to use Spotify as a subsitutive music app. Now the question becomes how to migrate my old playlists on Netease to Spotify: It is infeasible to do this manually since a playlist may have hundreds, even thousands of songs. This is why I built this project.

Since Netease only opens APIs for partners, I have to rely on existing libraries and the one I used is <a href="https://github.com/mos9527/pyncm">PyNCM</a>. By contrast, Spotify actually opens its official APIs so ofc I tried to use it in the beginning (which implies I didn't use it eventually, explained later). The biggest challenge I encountered is **OAuth 2.0**, a rather complex authentication with minimal documentation/explanation, which is why I am writing a blog here to help others who may face the same difficulty in the future using Spotify as an example.

So, **access_token** is the single most important thing we must pass in to all Spotify (or other apps that implement **OAuth 2.0**) APIs' request header, and we must first get it through another POST request to the ```/api/token``` endpoint, passing in **client_id** and **client_secret** of Spotify, which is very easy to implement. This is the workflow:

![Picture 2](/assets/Netease%20Playlist%20to%20Spotify/auth-client-credentials.png)

Passing in the obtained **access_token** to request headers, you are already able to call various APIs including Albums, Artists, Search, Playlists, etc. You can do plenty of things now such as getting an artist's related artists. However, if you try to add a track to a playlist through the correspoinding Playlists API, you will receive an error. I was very confused why, I probably just stared at the screen for half an hour until I noticed the only difference: On top of that Playlists API's documentation, there's something called **Authorization scopes**. This **Authorization scopes** was NEVER mentioned in the <a href="https://developer.spotify.com/documentation/web-api/tutorials/getting-started">Getting started</a> section of Spotify's documentation. What's worse, when I opened Spotify's <a href="https://developer.spotify.com/documentation/web-api/concepts/scopes">Scopes documentation</a>, it only showed you should use abc scopes to do xyz things, without telling you how to use it.

This is where **OAuth 2.0** comes in. Remember the POST request we used to get **access_token**? We must pass in **scopes** now in addition to **client_id** and **client_secret**. And we should POST to another endpoint ```/authorize```. The return is **NO LONGER access_token** but a uri for you to open and login to your own, real Spotify account. After logging in, you will be redirected to another uri that contains a parameter **code**, which is something we need to pass in when POSTing the ```/api/token``` endpoint to get **access_token**, the apple of our eyes. This is the more complex workflow:

![Picture 3](/assets/Netease%20Playlist%20to%20Spotify/auth-code-flow.png)

With that **access token** and correctly set scopes, you can call APIs that require login. Using Python's request and webbrowser modules, this is the base function that implements OAuth 2.0 and returns the **access token**:
