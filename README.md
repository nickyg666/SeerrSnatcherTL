# SeerrSnatcherTL
This is a program written in Python to handle webhooks from Jellyseer, and automatically download them to a watched directory
It was born out of a necessity for a custom automation solution similar to radarr/sonarr but more lightweight. 
I don't expect it to do movies too well, I barely tested that part. 
#PLEASE FEEL FREE TO STAR OR ADD PULL REQUESTS AND OPEN ISSUES

#It requires requests and feedparser
#pip install requests feedparser 
or if you're on Ubuntu: 
sudo apt install python3-requests python3-feedparser
I'm not sure how to add the packages to windows environment.

This is my current stack:
TorrentLeech account (they don't allow custom RSS anymore so this uses their 2 available feeds and caches results to file)
deluge-gtk (for watching folders for new torrents, you can use any client you like)
jellyseerr (for managing requests, it is set to auto approve with a webhook configured to localhost:5005 to send requests)
jellyfin (hooked to jellyseerr for library info, using custom tabs to display jellyseerr dash also)
TMDB querying (needs API key, it is free to create an account there and get one)

What this script does in detail:
- reads RSS feeds from your torrent provider. Can't say this will work well with providers other than TorrentLeech, you may need to tweak it to fit your fields. It's not written to handle magnets.
- caches the RSS feeds to a local file, which is backed up every hour. Will restore from backup if rss cache is corrupt or missing.
- accepts webhooks on port 5005 from jellyseerr/probably overseerr too (UNTESTED), and adds the entry to tracked_shows dictionary.
- queries TMDB to get the air dates and seasons available for the shows you're tracking, caches the data and updates daily
- loosely matches TMDB result for tracked show to your RSS entries
- if it finds a good enough match with high confidence, check for existing history in download history, if there is none then download the torrent to the watched directory based on if it is tagged TV or movie.
- Checks every 2 minutes (configurable in POLLING_INTERVAL) for RSS updates / new matches.
- logs everything you could want to know about the process.

What it won't do:
- rocket science
- filter quality out, it will download the first match it finds (usually in the highest quality but it's random)
- things I didn't write it to (issues and pull requests welcome!)

You can also use jellyseerr with Plex or Emby and achieve the same results. You don't need to use deluge as the client, any of them that support folder monitoring with a custom folder for downloads should work fine.
