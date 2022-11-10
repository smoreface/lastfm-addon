# Last.fm add-on for Splunk

This repo contains the code for the Last.fm add-on for Splunk. If you want to collect your [Last.fm](https://www.last.fm/) scrobbles so that you can analyze them in Splunk, use this add-on!

## About this add-on

This add-on collects a user's recently listened tracks from the Last.fm API and inputs them into Splunk so that you can analyze your music data.

I created this app for my own personal purposes, and am open sourcing it so that others can also explore their Last.fm data in Splunk.

## Prerequisites for using this add-on

To use this add-on, you must have the following:

- A Last.fm account with music scrobbled. See [What is a Scrobble and what is Scrobbling?](https://cbsi.secure.force.com/lastfm/articles/LastFM/What-is-scrobbling) in the Last.fm Frequently Asked Questions.
- A Last.fm API key. See [Create API account](https://www.last.fm/api/account/create) on the Last.fm API website.

## Before you start using this add-on

Before you start, consider how many Last.fm scrobbles you want to import.

This add-on is best for collecting your music scrobbles starting when you install it. If you have a lot of data in Last.fm, I don't recommend using this add-on to retrieve years of scrobble history. Instead, I recommend backfilling your data before you start using this add-on.

To backfill your data, do the following:

1. Run [the lastexport.py script](https://github.com/encukou/lastscrape-gui/blob/master/lastexport.py) and save the output to a file.
2. [Import the file into your Splunk instance](https://docs.splunk.com/Documentation/Splunk/9.0.2/Data/Uploaddata).
3. [Set the sourcetype of the data](https://docs.splunk.com/Documentation/Splunk/9.0.2/Data/Setsourcetype) to a custom name, such as `lastfm_backfill`. Make sure to set the `MAX_DAYS_AGO` setting to a number of days earlier than your Last.fm data archive. I picked something arbitrarily large like 10000.

If you backfill your data, you can note the [unix timestamp](https://www.epochconverter.com/) of your last scrobble from that output as a checkpoint to use with this add-on. See [Configure this add-on](#configure-this-add-on).

### Extra disclaimers

I have only ever run this add-on on a single instance of Splunk on a Linux server, and previously, my personal Mac laptop. It is currently running on an instance with Splunk version 9.0.1.

I am open to issues, pull requests, and suggestions for this add-on, but I cannot offer detailed support.

## What's in this add-on

This add-on contains the following:

- A modular input to collect data from the Last.fm API.
- Field extractions for Last.fm data.
- Field aliases to normalize the Last.fm fields to simpler field names: `album`, `artist`, `artist_mbid`, `track_name`.

### About the data input

The modular input calls the Last.fm API endpoint [user.getRecentTracks](https://www.last.fm/api/show/user.getRecentTracks) endpoint every 90 seconds to retrieve the tracks most recently listened to by the configured username.

The interval is configurable, but I recommend 90 seconds as a frequency for avoiding duplicate events but still catching shorter songs such as [Norgaard](https://www.youtube.com/watch?v=8VI3q4T-1Jc) by the Vaccines.

While the endpoint response includes the "currently playing track with the `nowplaying="true"` attribute", the input trims that portion of the response before indexing the event.

Each Splunk event created by this input is equivalent to one "scrobble", or listen event for a track.

## Install this add-on

To install this add-on, you can do the following:

1. Clone this repository.
2. Stop the Splunk platform.
3. Copy the `TA-lastfm-add-on-for-splunk` directory into the `$SPLUNK_HOME/etc/apps` directory.
4. Start the Splunk platform.

There are a number of ways to install Splunk add-ons. See [About installing Splunk add-ons](https://docs.splunk.com/Documentation/AddOns/released/Overview/Installingadd-ons) for general guidance for installing add-ons in Splunk.

## Configure this add-on

After you install this add-on, configure it to start collecting your Last.fm data.

In Splunk Web, do the following:

1. Open the Lastfm add-on for Splunk.
2. On the **Inputs** page, click **Create New Input**.
3. Enter a name for your input. Use something distinctive and memorable.
4. Specify a time interval, or a frequency at which to run the input. I use **90** seconds.
5. Specify an index. I use a **music** index.
6. Specify your Last.fm username.
7. Specify your Last.fm API key.
8. (Optional) Provide a unix timestamp to use as an initial checkpoint to start retrieving data from the Last.fm API.
9. Leave the default **Checkpoint type** as **Auto**.
10. Click **Add**.

I have only used this to create one input for one username, but you could certainly collect data from multiple usernames and compare their listening activities. 

## Test this add-on

To see whether or not this add-on is working, do the following:

1. Listen to some music that Last.fm is able to scrobble for about 15 minutes.
2. Search the index you specified for your input to identify events.

If you want to use a dashboard that I built to see highlights from my recent Last.fm activity and easily validate that my input is working, Install the [Music App for Splunk](https://github.com/smoreface/music_app_for_splunk) and open the `Splunking Music` dashboard.

## Questions, suggestions, and more?

Please [open an issue](https://github.com/smoreface/lastfm-addon/issues/new/choose) in this GitHub repository.
