# [Sparkify] Postgres Database and ETL pipeline For Song Play Analysis

## Problem

A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analytics team is particularly interested in understanding what songs users are listening to. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

In this project, I define fact and dimension tables for a star schema for the particular analytics, and write an ETL pipeline that transfers data from files in two local directories into these tables in Postgres using Python and SQL.


## About this repository

1. `data`

- Song datasets: Each file is in JSON format and contains metadata about a song and the artist of that song. Below is an example of what a single song file, TRAAAAW128F429D538.json looks like.

```
{"num_songs: 1
artist_id: "ARD7TVE1187B99BFB1"
artist_latitude: null
artist_longitude: null
artist_location: "California - LA"
artist_name: "Casual"
song_id: "SOMZWCG12A8C13C480"
title: "I Didn't Mean To"
duration: 218.93179
year: 0
```

- Log datasets: The JSON files contain the simulation of activity logs from a music streaming app based on specified configurations. The JSON files are partitioned by year and month. For example, here are filepaths to two files in this dataset.

```
{"artist":"Slipknot","auth":"LoggedIn","firstName":"Aiden","gender":"M","itemInSession":0,"lastName":"Ramirez","length":192.57424,"level":"paid","location":"New York-Newark-Jersey City, NY-NJ-PA","method":"PUT","page":"NextSong","registration":1540283578796.0,"sessionId":19,"song":"Opium Of The People (Album Version)","status":200,"ts":1541639510796,"userAgent":"\"Mozilla\/5.0 (Windows NT 6.1) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/36.0.1985.143 Safari\/537.36\"","userId":"20"}
```

2. `sql_queries.py`: This file contains all SQL queries. This is imported into other files.
3. `create_tables.py`: Running this file does the following.
    - Drops (if exists) and Creates the sparkify database. 
    - Establishes connection with the sparkify database and gets cursor to it.    
    - Drops all the tables.  
    - Creates all tables needed. 
    - Finally, closes the connection. 
    
4. `test.ipynb`: This notebook contains SQL queries to view the first 5 rows of each table. This can be used to see if the data is correctly loaded to the tables in the database.
5. `etl.ipynb`: This notebook reads and processes a single file from song_data and log_data and loads the data into your tables.
6. `etl.py`: reads and processes files from song_data and log_data and loads them into your tables. You can fill this out based on your work in the ETL notebook.
7. `README.md`: Current file

## Database Schema Design
The schema used for this exercise is the Star Schema: There is one main fact table containing all the measures associated to each event (user song plays), and 4 dimentional tables, each with a primary key that is being referenced from the fact table.

On why to use a relational database for this case:

- The data types are structured (we know before-hand the structure of the jsons we need to analyze, and where and how to extract and transform each field)
- The amount of data we need to analyze is not big enough to require big data related solutions.
- Ability to use SQL that is more than enough for this kind of analysis
- Data needed to answer business questions can be modeled using simple ERD models

- Fact Table
    - songplays - records in log data associated with song plays i.e. records with page NextSong
        - songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent

- Dimension Tables
    - users - users in the app
        - user_id, first_name, last_name, gender, level
    - songs - songs in music database
        - song_id, title, artist_id, year, duration
    - artists - artists in music database
        - artist_id, name, location, latitude, longitude
    - time - timestamps of records in songplays broken down into specific units
        - start_time, hour, day, week, month, year, weekday

## ETL Pipeline
The ETL pipeline can be executed by running `etl.py`, which does the following.

1. Connect to Sparkify database

2. Process `song_data`
- Read song file (JSON files in `song_data` directory)
- Insert song record into `songs` table
- Insert artist record into `artist` table

3. Process `log_data`
- Read log file (JSON files in `log_data` directory)
- Insert time data record into `time table`
- Insert user data record into `user table`
- Insert songplay record
    - Query 'song_id', 'artist_id' of the given song, artist name and the length in the log data
    - Insert the query results with the rest of the data into `songplays` table.