# SeerrSnatcherTL
This is a program written in Python to handle webhooks from Jellyseer, and automatically download their torrents to a watched directory if it is found in the rss cache.
It was born out of a necessity for a custom automation solution similar to radarr/sonarr but more lightweight. It spiraled into a whole web app.

SeerrSnatcherTL v2.0 release - new WebUI!

## License
This project is released under **The Unlicense**, placing the code in the public domain.
You may use, modify, redistribute, or relicense the project without restriction. I made this with copilot (it sucks at coding, did the first few hundred lines only) and mostly chatGPT.


roadmap - next few weeks or months: 

fix rss backup
that's it for now

v 2.1.0 new features
add apps and handling for ANDROID TV and Roku (unfinished, but basic implementation for roku TV app included in this version)

extra requests page to add things that are in tmdb but can't be added in jellyseerr for one reason or another (request more unavailable, etc)
  it searches TMDB by title, matches TMDB id, provides poster and if you click details will let you request seasons. I think if you just click request it adds all seasons. I forget. clicking poster will take you to tmdb result page.

general features introduced in v2.0:

dashboard for consolidated views

searchable logs via logs page

searchable rss feed via RSS page

tracked items view with posters via tracked page (can make searchable if you like)

tracks downloads so no more duplicates (searchable)

will remove request from tracked items once media_available webhook is received from Jellyseerr

scan now / refresh RSS buttons at top, restart button in settings


settings:
many configurable things, added user management to manage more than one TV/Movies dir. 

SMTP configuration as well as notification toggles for what you want to be notified for.

SMTP test send button / function

preferred quality with optional strict enforcement (if you don't want to grab higher quality than is selected when available)

enable background tasks toggle (keep checked unless you don't want to stay on top of things)

log level setting

---

## v2.0 â€“ Major Update

### Project Structure
- Expanded into a modular multi-file application.
- Added directories for templates, static assets, configuration, and service files.
- Introduced a maintainable layout suitable for future growth.

### Web Interface
- Added Flask-based web UI.
- New pages: Dashboard, Downloads, Logs, RSS, Tracked Items, Settings.
- Implemented Jinja2 templates and a unified styling system.
- Added client-side JS for enhanced interactions.

### Configuration
- Introduced `settings.json` for editable runtime settings.
- Removed hardcoded values and centralized configuration.

### Background Processing
- Improved periodic worker for RSS updates and tracked-item processing.
- Ensured required directories and caches are initialized on startup.

### RSS and Metadata Handling
- Improved RSS caching with automatic backups.
- Enhanced TMDB metadata caching with airdate storage.
- More robust handling of incomplete or malformed API responses.

### Torrent Handling
- Improved torrent file saving, naming, and duplicate prevention.
- Added quality scoring and safer content-type checks.
- Improved download history tracking.

### Webhook Handling
- Improved extraction of media type, title, and seasons from webhook payloads.
- Ensured consistent updates when new items or seasons are added.

### System Integration
- Added `SeerrSnatcherTL.service` for systemd support.
- Included a `requirements.txt` for consistent environment setup.

# PLEASE FEEL FREE TO STAR OR ADD PULL REQUESTS AND OPEN ISSUES

# It requires requests and feedparser
## pip install requests feedparser 
or if you're on Ubuntu: 
## sudo apt install python3-requests python3-feedparser
I'm not sure how to add the packages to windows environment.
# It also requires you fill in some fields: rss feeds, TMDB API key, and your watched directories at the top of the script. You can also configure POLLING_INTERVAL to check less often than every 2 minutes, or change the webhook listener port.

This is my current stack:

TorrentLeech account (they don't allow custom RSS anymore so this uses their 2 available feeds and caches results to file to eventually build a local database)

deluge-gtk (for watching folders for new torrents, you can use any client you like with the capability)

jellyseerr (for managing requests, it is set to auto approve with a webhook configured to localhost:5006/webhook to send requests and media_available hooks)

jellyfin (hooked to jellyseerr for library info, using custom tabs to display jellyseerr dash also, will send webhook when requested content available)

TMDB querying (needs API key, it is free to create an account there and get one)

systemd service (unit file included in release v1.0.2)

What this script/web app does in detail:

- reads RSS feeds from your torrent provider. Can't say this will work well with providers other than TorrentLeech, you may need to tweak it to fit your fields. It's not written to handle magnets. If you add a feature request for other providers and send me valid creds/rss feeds I will try to incorporate more providers.

- caches the RSS feeds to a local file, which is backed up every hour*. Will restore from backup if rss cache is corrupt or missing.
* backup logic needs to be fixed in current releases
  
- accepts webhooks on port localhost:5006/webhook from jellyseerr/probably overseerr too (UNTESTED), and adds the entry to tracked_items dictionary.

- queries TMDB to get the air dates and seasons available for the shows you're tracking, caches the data and updates all the time (work in progress cutting down on api calls). Updates cache when new request is received.

- loosely matches TMDB result for tracked item to your RSS entries, tightened matching heuristics a lot to not pull garbage on accident or get bad matches.

- if it finds a good enough match with high confidence, check for existing history in download history, if there is none then download the torrent to the watched directory based on if it is tagged TV or movie. Record to downloaded items so it does not duplicate downloads.

- Checks every 2 minutes (configurable in settings) for new matches. Every 10 minutes for new RSS.

- logs everything you could want to know about the process, change logging to DEBUG from INFO if you want to see EVERYTHING.

Files it creates:

posters/ folder in base dir to cache posters

rss-feeds.json/.bak (.bak not working as of 2.1.0)

downloaded_history.json

tracked_items.json

tmdb_metadata_cache.json

tmdb_newshow_dump.json

SeerrSnatcherTL.log

settings.json 

What it won't do:
- rocket science
- things I didn't write it to (issues, pull requests, and feature requests welcome!)
- find torrents outside of the rss_cache (that's my biggest issue right now, and a fundamental flaw of how it works)
  
You can also use jellyseerr with Plex or Emby and (probably) achieve the same results. You don't need to use deluge as the client, any of them that support folder monitoring with a custom folder for downloads should work fine. This is mostly crafted to deal with TorrentLeech and their restrictive RSS feeds.
