# blasterfeed

#### Table of Contents

1. [Description](#description)
2. [Installation](#Installation)
3. [The config file](#the-config-file)  
  3.1. [The structure](#the-structure)  
  3.2. [Example](#example)  
4. [Usage](#usage)  
5. [Docker](#docker)  
  5.1. [Build the Docker Image](#build-the-docker-image)  
  5.2. [Run the Docker Container](#run-the-docker-container)  
6. [Technical decisions](#technical-decisions)  
  6.1. [Python 3](#python-3)  
  6.2. [SQLite3](#sqlite3)  
7. [SQLite3](#sqlite3)  
  7.1. [Description](#description)  
  7.2. [Check the content](#check-the-content)  
8. [dateutil and timezone](#dateutil-and-timezone)  
9. [Known issues](#known-issues)  

## Description

Nowadays RSS feeds are providing just a brief summary instead of the full article.  
This python script creates a new RSS feed fetching the full article using the links present in the feed.

## Installation

blasterfeed is relying on some other python modules.  
To install them

```bash
pip install -r requirements.txt
```

requests>=2.6.0
is to avoid the error

```
  File "/usr/local/lib/python2.7/dist-packages/requests/packages/urllib3/contrib/pyopenssl.py", line 70, in <module>
    ssl.PROTOCOL_SSLv3: OpenSSL.SSL.SSLv3_METHOD,
AttributeError: 'module' object has no attribute 'PROTOCOL_SSLv3'
```

Reference: [patch-pyopenssl-for-sslv3-issue](http://stackoverflow.com/questions/28987891/patch-pyopenssl-for-sslv3-issue)

## The config file

The config file must be called config.yml and must be placed in the _config/_ directory.  
For each feed you need to create a different section.  
All the fields are mandatory.

### The structure

```
feed_name
  feed: <url_of_the_feed>
  output_file: <full_path_of_the_output_file>
```

### Example

```
AwesomeWebsite:
  feed: https://an-awesome-webiste.com/feed.xml
  output_file: /var/www/rss/OriginalFeed.xml

GreatWebsite:
  feed: https://great-webiste.com/feed.xml
  output_file: /var/www/rss/GreatWebsite.xml
```

## Usage

After setting the config file, run

`python3 blasterfeed3k.py`

Available parameters

```
--debug <To enable debug mode>
--disable-cache <To not create a SQLite DB used for caching>
```

## Docker

### Build the Docker Image

```
docker build -t blasterfeed .
```

### Run the Docker Container

Example:

```
docker run --rm -v $(pwd)/config:/home/blasterfeed/config blasterfeed
```


## Technical decisions

### Python 3

This is due to the compatibility offered by newspaper module.
As stated on their [README.md](https://github.com/codelucas/newspaper)

> Newspaper is a Python3 library! Or, view our deprecated and buggy Python2 branch

### SQLite3

Lightweight and sufficient for what this script has to do.

## SQLite3

### Description

To improve speed and reduce HTTP requests, for each item inside a feed, blasterfeed stores the HTML content of the 
full article inside a SQLite database.  
The DB is called blasterfeed-cache.db and it is stored in the _config/_ directory.  
At the end of each XML generation, a cleanup will run, deleting all the articles that are not present anymore in the 
feed.
The fields stored in the DB are:
- Link to the RSS feed
- Link to the item of the feed
- Date, for debug testing only
- Content

### Check the content

To verify what's inside the SQLite database, go inside the config directory and you can run the following commands

To connect to the database
```
sqlite3 blasterfeed-cache.sqlite3
```

To check the schema

```
.schema
```

To check the tables

```
.tables
```

To check what is stored inside the _data_ table

```
SELECT feed_link, item_link, date FROM data;
```

I'm excluding _content_ as it contains the HTML of the article.  
To exit

```
.exit
```

## dateutil and timezone

feedparser uses dateutil to parse the feed pubDate if it is a string.
Unfortunately dateutil doesn't support timezones and it fails with ValueError if the feed pubDate doesn't have one.
To solve this issue, I have generated a dictionary with as much timezone as I could to use with dateutil, so I can 
provide to feedparser a datetime object with timestamp.
To generate this dictionary I have used the script *generate_timezone.py* which is not called anywhere.
The dictionary of the time zones is in *my_timezones.py* and it is imported into *blasterfeed3k.py*

## Known issues

If you get the following error fetching a full article

```
Traceback (most recent call last):
  File "/usr/local/lib/python2.7/dist-packages/newspaper/images.py", line 119, in fetch_url
    p.feed(new_data)
  File "/usr/local/lib/python2.7/dist-packages/PIL/ImageFile.py", line 392, in feed
    im.mode, d, a, im.decoderconfig
  File "/usr/local/lib/python2.7/dist-packages/PIL/Image.py", line 413, in _getdecoder
    raise IOError("decoder %s not available" % decoder_name)
IOError: decoder jpeg not available
```

follow the steps provided here: [PIL/Pillow IOError: decoder jpeg not available](https://coderwall.com/p/faqccw/pil-pillow-ioerror-decoder-jpeg-not-available)
