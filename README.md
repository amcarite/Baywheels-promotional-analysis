# Lyft Bay Wheels Promotional Offer Analysis 

As a data scientist with the Lyft Bay Wheels team, I have been tasked with looking into some SQL data sets and doing some analysis to determine potential offers that Lyft can run in order to increase ridership. 

Some frequent offers we run are as follows: 
  * Single Ride 
  * Monthly Membership
  * Annual Membership
  * Bike Share for All
  * Access Pass
  * Corporate Membership
  * etc.



### The ultimate goal of my analysis is to answer the following two questions: 

  * What are the 5 most popular trips that you would call "commuter trips"? 
  
  * What are your recommendations for offers (justify based on your findings)?

### There are three static data sets in the Google BigQuery data store that I am able to use for my analysis:

  * bikeshare_stations

  * bikeshare_status

  * bikeshare_trips

## To begin, I will complete some preliminary exploratory analysis from the Google Big Query Console
#### Some initial queries

- What's the size of this dataset? (i.e., how many trips)

```sql
#standardSQL
SELECT count(distinct(trip_id)) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
983648 trips!

- What is the earliest start date and time and latest end date and time for a trip?

```sql 
SELECT MIN(start_date) as min_start 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

2013-08-29 09:08:00 UTC is the earliest start date

```sql
SELECT MAX(end_date) as max_end 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
2016-08-31 23:48:00 UTC is the latest end date

- How many bikes are there?

```sql
SELECT count(distinct(bike_number))
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

There are 700 bikes 

### Three additional questions that I would like to run queries for

- Question 1: How many trips are taken during commuting hours (7am-9am) and (4pm-6pm)
  * Answer: 445,940 trips 
  * SQL query:

```sql
select count(*) trip_cnt FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
where
(EXTRACT(HOUR from start_date) > 7 and EXTRACT(HOUR from start_date) < 9 or
EXTRACT(HOUR from end_date) > 7  and EXTRACT(HOUR from end_date) < 9 or
EXTRACT(HOUR from start_date) > 16 and EXTRACT(HOUR from start_date) < 21 or
EXTRACT(HOUR from end_date) > 16  and EXTRACT(HOUR from end_date) < 21)
and DATE_DIFF(end_date,start_date,HOUR) < 4 
```
The above query removes all rides longer than 4 hours as these rides are likely not commutes

- Question 2: How does the number of subscriber trips differ during commuting hours (7am-9am) and (4pm-6pm) and non-commuting hours 
  * Answer: 	
272,944 subscriber rides during commute times 
vs 
286,101 subscriber rides during non-commute times

  * SQL query:
query for subscriber rides in commute hours
```sql
select count(*) trip_cnt FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
where
(EXTRACT(HOUR from start_date) > 7 and EXTRACT(HOUR from start_date) < 9 or
EXTRACT(HOUR from end_date) > 7  and EXTRACT(HOUR from end_date) < 9 or
EXTRACT(HOUR from start_date) > 16 and EXTRACT(HOUR from start_date) < 21 or
EXTRACT(HOUR from end_date) > 16  and EXTRACT(HOUR from end_date) < 21)
and DATE_DIFF(end_date,start_date,HOUR) < 4 
and subscriber_type = 'Subscriber'
```
query for subscriber during non-commute hours
```sql
select count(*) trip_cnt FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
where
(EXTRACT(HOUR from start_date) NOT BETWEEN 7 and 9) and 
(EXTRACT(HOUR from start_date) NOT BETWEEN 16 and 18) and
(EXTRACT(HOUR from end_date) NOT BETWEEN 7 and 9) and 
(EXTRACT(HOUR from end_date) NOT BETWEEN 16 and 18) 
and DATE_DIFF(end_date,start_date,HOUR) < 4 
and subscriber_type = 'Subscriber'
```



- Question 3: how does the duration of the trip differ during commuting hours (subscribers and non-subscribers) 
  * Answer: 15.0059 min/trip during commute hours 
  18.46 min/trip during non-commute hours 
  * SQL query:

```sql
select AVG(duration_sec)/60 duration_min FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
where
(EXTRACT(HOUR from start_date) NOT BETWEEN 7 and 9) and 
(EXTRACT(HOUR from start_date) NOT BETWEEN 16 and 18) and
(EXTRACT(HOUR from end_date) NOT BETWEEN 7 and 9) and 
(EXTRACT(HOUR from end_date) NOT BETWEEN 16 and 18) 
```


## Some mroe querying from the BigQuery CLI 


### Re-running queries for reproducibility 

  - What's the size of this dataset? (i.e., how many trips)
    * There are 983,648 trips
    
    * Query:
    ```
    (base) jupyter@python-20210824-223530:~$ bq query --use_legacy_sql=false '
         SELECT count(distinct(trip_id))
         FROM
            `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```


  - What is the earliest start time and latest end time for a trip?
    * The earliest start time was 2013-08-29 09:08:00
    * The latest end time was 2016-08-31 23:48:00

    Min start time query
    ```
    bq query --use_legacy_sql=false '
        SELECT MIN(start_date) as min_start
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```
    Max end time query
    ```
    bq query --use_legacy_sql=false '
        SELECT MAX(end_date) as max_end
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```
    
  - How many bikes are there?
    * There are 700 total bikes in this dataset
    
    * Query:
    ```
    bq query --use_legacy_sql=false '
        SELECT count(distinct(bike_number))
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

### New queries being run on the BigQuery CLI

#### Some questions that may be useful in my analysis

- Question 1: What percentage of rides come to/from BART transit stations?

- Question 2: Do the Bart Transit stations see an increase in rides during commute hours?

- Question 3: What stations have the most rides in the morning commuting hours?

- Question 4: What stations have the most rides in evening commuting hours

- Question 5: What cities have the most bike share rides per station?

- Question 6:  Do people using Caltrain to commute into SF or down to Silicon Valley take bike share to or from the station

### Some answers to the above questions 

- What percentage of rides come from BART stations?
  * Answer: 130,514 trips come from (starting or ending trips) Bart Stations vs 983,648 total trips. This shows that around 13% of all trips start or stop at BART station locations. 
  * SQL query:
  ```sql
  SELECT (SELECT count(*) bart_cnt
        FROM   `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE start_station_name = 'Powell Street BART' or
        start_station_name = 'Civic Center BART (7th at Market)' or
        start_station_name = 'Beale at Market' or
        start_station_name = 'Post at Kearney' ) AS bart_trips,
       (SELECT count(*) trip_cnt
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`) AS total_trips
  ```

- Do the Bart Transit stations see an increase in rides during commute hours?
  * Answer: There are 34,569 trips during commute hours and 53,729 during non-commute hours. There does not seem to be an increase.
  * SQL query:
  ```sql
  SELECT (SELECT count(*) bart_cnt_commute
        FROM   `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE (start_station_name = 'Powell Street BART' or
        start_station_name = 'Civic Center BART (7th at Market)' or
        start_station_name = 'Beale at Market' or
        start_station_name = 'Post at Kearney'or
        end_station_name = 'Powell Street BART' or
        end_station_name = 'Civic Center BART (7th at Market)' or
        end_station_name = 'Beale at Market' or
        end_station_name = 'Post at Kearney') and
        (EXTRACT(HOUR from start_date) > 7 and EXTRACT(HOUR from start_date) < 9 or
        EXTRACT(HOUR from end_date) > 7  and EXTRACT(HOUR from end_date) < 9 or
        EXTRACT(HOUR from start_date) > 16 and EXTRACT(HOUR from start_date) < 18 or
        EXTRACT(HOUR from end_date) > 16  and EXTRACT(HOUR from end_date) < 18)
        and DATE_DIFF(end_date,start_date,HOUR) < 4  ) AS bart_com_trips,
       (SELECT count(*) bart_cnt_non_commute
        FROM   `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE (start_station_name = 'Powell Street BART' or
        start_station_name = 'Civic Center BART (7th at Market)' or
        start_station_name = 'Beale at Market' or
        start_station_name = 'Post at Kearney'or
        end_station_name = 'Powell Street BART' or
        end_station_name = 'Civic Center BART (7th at Market)' or
        end_station_name = 'Beale at Market' or
        end_station_name = 'Post at Kearney') and
        ((EXTRACT(HOUR from start_date) NOT BETWEEN 7 and 9) and 
        (EXTRACT(HOUR from start_date) NOT BETWEEN 16 and 18) and
        (EXTRACT(HOUR from end_date) NOT BETWEEN 7 and 9) and 
        (EXTRACT(HOUR from end_date) NOT BETWEEN 16 and 18)) 
        and DATE_DIFF(end_date,start_date,HOUR) < 4 ) AS bart_non_com_trips

  ```
  
- What are the top trips (by trip_count) during morning commute hours?

| Row 	|               Start Station              	|              End Station             	| Number of Trips 	|
|:---:	|:----------------------------------------:	|:------------------------------------:	|:---------------:	|
| 1   	| Harry Bridges Plaza (Ferry Building)     	| 2nd at Townsend                      	|            3118 	|
| 2   	| San Francisco Caltrain (Townsend at 4th) 	| Harry Bridges Plaza (Ferry Building) 	|            1959 	|
| 3   	| Harry Bridges Plaza (Ferry Building)     	| Embarcadero at Sansome               	|            1745 	|
| 4   	| San Francisco Caltrain (Townsend at 4th) 	| Steuart at Market                    	|            1703 	|
| 5   	| Steuart at Market                        	| Embarcadero at Sansome               	|            1676 	|
 
 
 * SQL query:
  ```sql
  SELECT start_station_name,end_station_name, (avg(duration_sec)) avg_duration,count(*) trip_cnt
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
  WHERE
  (EXTRACT(HOUR from start_date) > 7 and EXTRACT(HOUR from start_date) < 9 or
  EXTRACT(HOUR from end_date) > 7  and EXTRACT(HOUR from end_date) < 9)
  and (DATE_DIFF(end_date,start_date,HOUR) < 4) 
  group by start_station_name,end_station_name
  order by 4 desc,start_station_name,end_station_name
  ```

- What are the top trips (by trip_count) during morning commute hours?

| Row 	|      Start Station     	|                End Station               	| Number of Trips 	|
|:---:	|:----------------------:	|:----------------------------------------:	|:---------------:	|
| 1   	| Embarcadero at Sansome 	| Steuart at Market                        	|            2981 	|
| 2   	| Embarcadero at Folsom  	| San Francisco Caltrain (Townsend at 4th) 	|            2951 	|
| 3   	| 2nd at Townsend        	| Harry Bridges Plaza (Ferry Building)     	|            2839 	|
| 4   	| 2nd at South Park      	| Market at Sansome                        	|            2679 	|
| 5   	| Market at 10th         	| San Francisco Caltrain 2 (330 Townsend)  	|            2646 	|
  
  * SQL query:
  ```sql
  SELECT start_station_name,end_station_name, count(*) trip_cnt
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`  
  WHERE
  (EXTRACT(HOUR from start_date) > 16 and EXTRACT(HOUR from start_date) < 19 or
  EXTRACT(HOUR from end_date) > 16  and EXTRACT(HOUR from end_date) < 19)
  and (DATE_DIFF(end_date,start_date,HOUR) < 4) 
  group by start_station_name,end_station_name
  order by 3 desc,start_station_name,end_station_name
  ```

---

## My full analysis will be done in the following jupyter notebook:

[Jupyter Notebook](https://github.com/mids-w205-chakraverty/project-1-amcarite/blob/assignment/Project_1.ipynb)




