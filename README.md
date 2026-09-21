# Nine Inch Nails Data

This repository is a collection of data regarding releases and performances by Nine Inch Nails. We have designed a schema that we intend to be generally useful for music and music performance in general. 

The dataset covers over 35 years of Nine Inch Nails — every release, every show, every credit:

* **Discography** — 100+ releases across albums, EPs, singles, soundtracks, and film scores, with catalog numbers, release dates, and label history
* **Tracks** — every song with duration, BPM, lyrics, alternate versions, and full personnel credits (composers, producers, engineers, mixers, remixers, musicians, and instruments)
* **Cover songs** — every cover NIN has recorded or performed live, with original artist attribution
* **Live history** — 1,000+ concerts spanning 1988 to present, with full setlists, venues, cities, countries, and festival appearances
* **Contributors** — emain collaborators across the entire discography, queryable by role

The schema diagram for the database can be [seen within this repository](https://github.com/aahmad/nin-data/blob/main/images/database.png). The database was designed using [PostgreSQL](https://www.postgresql.org).

## Download

The dataset is available in two formats from the [latest release](https://github.com/aahmad/nin-data/releases/latest):

| Format | Description |
|--------|-------------|
| `nin-YYYY-MM-DD.db` | SQLite — zero setup, works everywhere |
| `nin-YYYY-MM-DD.sql` | PostgreSQL dump — full fidelity restore |

## Getting Started

### SQLite

Download the `.db` file and query it immediately:

```bash
sqlite3 nin-YYYY-MM-DD.db

sqlite> SELECT count(*) FROM concerts;
sqlite> SELECT title, bpm FROM tracks ORDER BY bpm DESC LIMIT 5;
```

Works with any SQLite client: [DB Browser for SQLite](https://sqlitebrowser.org), Python/pandas, DuckDB, R, Excel, and more.

### PostgreSQL

```bash
createdb nin
psql nin < nin-YYYY-MM-DD.sql
psql nin
```

## Database Queries

Here are some queries for some interesting results.

### Releases By Age

```sql
SELECT 
  r.title, 
  r.catalog, 
  rt.release_type, 
  age(r.release_date), 
  r.duration, 
  (
    SELECT 
      count(*) 
    FROM 
      release_tracks rt2 
    WHERE 
      rt2.release_id = r.release_id
  ) s 
FROM 
  releases r 
  JOIN release_types rt USING (release_type_id) 
WHERE 
  rt.release_type IN ('album', 'ep', 'single') 
  AND parent_id IS NULL 
ORDER BY 
  release_date ASC;
  

             title             |  catalog  | release_type |           age            | duration  | s
-------------------------------+-----------+--------------+--------------------------+-----------+----
 Down In It                    | Halo 1    | single       | 34 years 4 mons 7 days   |           |  3
 Pretty Hate Machine           | Halo 2    | album        | 34 years 3 mons 2 days   | 48:32:00  | 10
 Head Like A Hole              | Halo 3    | single       | 33 years 10 mons         |           | 11
 Sin                           | Halo 4    | single       | 33 years 3 mons 12 days  |           |  4
 Broken                        | Halo 5    | ep           | 31 years 4 mons          | 31:35:00  |  8
 Fixed                         | Halo 6    | ep           | 31 years 1 mon 15 days   | 40:23:00  |  6
 March Of The Pigs             | Halo 7    | single       | 29 years 10 mons 25 days |           |  4
 The Downward Spiral           | Halo 8    | album        | 29 years 10 mons 14 days | 65:02:00  | 16
 Closer To God                 | Halo 9    | single       | 29 years 7 mons 23 days  |           |  9
 Further Down The Spiral       | Halo 10   | album        | 28 years 7 mons 21 days  | 63:56:00  | 11
 The Perfect Drug              | Halo 11   | single       | 26 years 8 mons 9 days   |           |  5
 The Day The World Went Away   | Halo 13   | single       | 24 years 6 mons 2 days   |           |  0
 The Fragile                   | Halo 14   | album        | 24 years 4 mons 1 day    | 103:39:00 | 23
 We're In This Together        | Halo 15   | single       | 24 years 3 mons 25 days  |           |  0
 ...
```
 
### Release Tracks

All the tracks on a particular release (_Broken_):

```sql
SELECT 
  r.title, 
  r.release_date, 
  rt.track_number, 
  t.display_title, 
  t.duration, 
  t.bpm 
FROM 
  tracks t 
  JOIN release_tracks rt USING (track_id) 
  JOIN releases r USING (release_id) 
WHERE 
  release_id = 20
  ORDER BY rt.track_number;                                                                                                                                                                                                                 

 title  | release_date | track_number |    display_title     | duration | bpm
--------+--------------+--------------+----------------------+----------+-----
 Broken | 1992-09-22   |            1 | Pinion               | 1:02     | 113
 Broken | 1992-09-22   |            2 | Wish                 | 3:46     | 135
 Broken | 1992-09-22   |            3 | Last                 | 4:44     |  90
 Broken | 1992-09-22   |            4 | Help Me I Am In Hell | 1:56     |  74
 Broken | 1992-09-22   |            5 | Happiness In Slavery | 5:21     | 120
 Broken | 1992-09-22   |            6 | Gave Up              | 4:08     | 144
 Broken | 1992-09-22   |           98 | Physical             | 5:29     |  80
 Broken | 1992-09-22   |           99 | Suck                 | 5:07     |  91
(8 rows)
```

### Track Versions

```sql
SELECT 
  DISTINCT r.title AS release, 
  t.display_title AS track_title 
FROM 
  tracks t 
  JOIN release_tracks tr USING (track_id) 
  JOIN releases r USING (release_id) 
WHERE 
  t.parent_id = 42;

                      release                       |              track_title
----------------------------------------------------+---------------------------------------
 Further Down The Spiral                            | Self Destruction, Final
 Further Down The Spiral                            | Self Destruction, Part Two
 Further Down The Spiral                            | The Art of Self Destruction, Part One
 Further Down The Spiral                            | The Beauty Of Being Numb
 Further Down The Spiral International (Halo 10 V2) | Self Destruction, Final
 Further Down The Spiral International (Halo 10 V2) | Self Destruction, Part Three
 Further Down The Spiral International (Halo 10 V2) | The Art of Self Destruction, Part One
(7 rows)
```

### Cover Versions

```sql
SELECT 
  DISTINCT t.display_title, 
  ct.cover_artist, 
  r.release_date 
FROM 
  tracks t 
  JOIN cover_artists ct USING (cover_artist_id) 
  JOIN release_tracks rt USING (track_id) 
  JOIN releases r USING (release_id) 
WHERE 
  r.parent_id IS NULL 
ORDER BY 
  r.release_date;


        display_title        |       cover_artist       | release_date
-----------------------------+--------------------------+--------------
 Get Down, Make Love         | queen                    | 1990-10-10
 Physical                    | adam ant                 | 1992-09-22
 Dead Souls                  | joy division             | 1994-03-08
 Memorabilia                 | soft cell                | 1994-05-30
 Metal                       | gary numan               | 2000-11-21
 Immigrant Song              | jimmy page, robert plant | 2011-12-02
 Is Your Love Strong Enough? | bryan ferry              | 2011-12-02
(7 rows)
```

### Highest BPM

```sql
SELECT t.track_id, t.title, bpm FROM tracks t ORDER BY bpm DESC NULLS LAST;

 track_id |         title         | bpm
----------+-----------------------+-----
       45 | march of the pigs     | 269
      514 | owl hunts rat         | 185
      540 | nothing ever ends     | 180
      413 | them and us           | 178
      543 | stupid panties        | 176
      531 | clockmaker            | 176
      346 | the splinter          | 176
      476 | finding a place       | 174
      527 | the dark knut returns | 174
      361 | millennia             | 170
...
```

### Longest Track Titles

```sql
SELECT title FROM tracks WHERE parent_id IS NULL ORDER BY length(title) DESC;
                      title
-------------------------------------------------
 objects in mirror (are closer than they appear)
 i can't give everything away (farewell mix)
 i'm looking forward to joining you, finally
 parallel timeline with alternate outcome
 the sleep of reason produces monsters
 complications with optimistic outcome
 other ways to get to the same place
 a man walks into an intrinsic field
 a traveller from an antique land
 down in the park (piano version)
 what have we done to each other?
 when it happens (don't mind me)
 i never thought it would be you
 in the hall of the moutain king
 pay no attention to the cactus
 burning bright (field on fire)
 looking forwards and backwards
 dreaming forwards & backwards
 absent friends and old ghosts
 every day is exactly the same
 backpack & brothers to m.i.t.
 looking forwards & backwards
 and all that could have been
 ...
```
 
### Festivals performed live for a certain year


 All festivals performed in the year 2022:

```sql
SELECT 
  v.name, 
  ct.city 
FROM 
  tours t 
  INNER JOIN concerts c USING (tour_id) 
  INNER JOIN venues v USING (venue_id) 
  JOIN locations l USING (location_id) 
  JOIN cities ct USING (city_id)
WHERE 
  v.festival IS TRUE 
  AND date_part('year', performed_on) = 2022 
ORDER BY 
  c.performed_on ASC;
  
         name          |     city
-----------------------+---------------
 shaky knees           | atlanta
 welcome to rockville  | daytona beach
 boston calling        | boston
 boston calling        | boston
 hellfest              | clisson
 primavera sound       | los angeles
 riot fest             | chicago
 louder than life      | louisville
(8 rows)

```

### Countries Most Performed At

```sql
SELECT 
  ct.country, 
  count(*) as num_times_performed 
FROM 
  concerts c 
  JOIN venues v USING (venue_id) 
  JOIN locations l USING (location_id) 
  JOIN countries ct USING (country_id) 
GROUP BY 
  ct.country 
ORDER BY 
  num_times_performed DESC;
  
        country        | num_times_performed
-----------------------+---------------------
 usa                   |                 676
 uk                    |                  59
 canada                |                  47
 germany               |                  36
 australia             |                  34
 japan                 |                  21
 france                |                  17
 netherlands           |                  12
 belgium               |                  11
 spain                 |                  10
 austria               |                   9
 italy                 |                   8
 ...
```

### Cities Most Performed At

```sql
SELECT 
  ct.city, 
  count(*) as num_times_performed 
FROM 
  concerts c 
  JOIN venues v USING (venue_id) 
  JOIN locations l USING (location_id) 
  JOIN cities ct USING (city_id) 
GROUP BY 
  ct.city 
ORDER BY 
  times_played DESC;
  
       city       | num_times_performed
------------------+--------------------
 los angeles      |           29
 chicago          |           27
 london           |           24
 new york         |           24
 atlanta          |           22
 cleveland        |           21
 las vegas        |           19
 detroit          |           16
 boston           |           15
 toronto          |           15
 new orleans      |           14
 houston          |           14
 melbourne        |           13
 dallas           |           13
 ...
```

### Instrumental Tracks

Tracks with no lyrics, ordered by release date:

```sql
SELECT
  t.title,
  r.title AS release,
  r.release_date
FROM tracks t
JOIN release_tracks rt ON rt.track_id = t.track_id
JOIN releases r ON r.release_id = rt.release_id
LEFT JOIN lyrics l ON l.track_id = t.track_id
WHERE t.parent_id IS NULL
  AND rt.main IS TRUE
  AND (l.track_id IS NULL OR l.lyrics IS NULL OR l.lyrics = '')
ORDER BY r.release_date ASC
LIMIT 10;

         title          |         release         | release_date
------------------------+-------------------------+--------------
 Help Me I Am In Hell   | Broken                  | 1992-09-22
 Pinion                 | Broken                  | 1992-09-22
 A Warm Place           | The Downward Spiral     | 1994-03-08
 March Of The Fuckheads | Closer To God           | 1994-05-30
 At The Heart Of It All | Further Down The Spiral | 1995-06-01
 Driver Down            | Lost Highway            | 1997-02-18
 Videodrones; Questions | Lost Highway            | 1997-02-18
 The Frail              | The Fragile             | 1999-09-21
 Just Like You Imagined | The Fragile             | 1999-09-21
 Pilgrimage             | The Fragile             | 1999-09-21
(10 rows)
```

## Database Schema

The schema diagram for the database can be [seen within this repository](https://github.com/aahmad/nin-data/images/database.png).
